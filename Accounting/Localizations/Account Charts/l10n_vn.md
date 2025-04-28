# Odoo Module: l10n_vn

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# This module is Copyright (c) 2009-2013 General Solutions (http://gscom.vn) All Rights Reserved.

from odoo import api, SUPERUSER_ID


def _post_init_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_vn.vn_template').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    "name": "Vietnam - Accounting",
    "version": "2.0.1",
    "author": "General Solutions",
    'website': 'http://gscom.vn',
    'category': 'Accounting/Localizations/Account Charts',
    "description": """
This is the module to manage the accounting chart for Vietnam in Odoo.
=========================================================================

This module applies to companies based in Vietnamese Accounting Standard (VAS)
with Chart of account under Circular No. 200/2014/TT-BTC

**Credits:**
    - General Solutions.
    - Trobz
""",
    "depends": [
        "account",
        "base_iban",
        "l10n_multilang"
    ],
    "data": [
         'data/l10n_vn_chart_data.xml',
         'data/account.account.template.csv',
         'data/l10n_vn_chart_post_data.xml',
         'data/account_tax_group_data.xml',
         'data/account_tax_report_data.xml',
         'data/account_tax_data.xml',
         'data/account_chart_template_data.xml',
    ],

    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_post_init_hook',

    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","reconcile"
"chart1121","Vietnamese Dong","1121","asset_cash","vn_template","False"
"chart1122","Foreign currencies","1122","asset_cash","vn_template","False"
"chart1123","Monetary Gold","1123","asset_cash","vn_template","False"
"chart1211","Shares",1211,"asset_current","vn_template","False"
"chart1212","Bonds",1212,"asset_current","vn_template","False"
"chart1218","Other securities and financial instruments",1218,"asset_current","vn_template","False"
"chart1281","Term deposits",1281,"asset_current","vn_template","False"
"chart1282","Bonds",1282,"asset_current","vn_template","False"
"chart1283","Lending loans",1283,"asset_current","vn_template","False"
"chart1288","Other held to maturity investments",1288,"asset_current","vn_template","False"
"chart131","Trade receivables",131,"asset_receivable","vn_template","True"
"chart132","Trade receivables(pos)",132,"asset_receivable","vn_template","True"
"chart1331","VAT on purchase of goods and services",1331,"asset_current","vn_template","False"
"chart1332","VAT on purchase of fixed assets",1332,"asset_current","vn_template","False"
"chart1361","Working capital provided to sub-units",1361,"asset_receivable","vn_template","True"
"chart1362","Intra-company receivables on foreign exchange",1362,"asset_receivable","vn_template","True"
"chart1363","Intra-company receivables on borrowing costs eligible to be capitalized",1363,"asset_receivable","vn_template","True"
"chart1368","Other intra-company receivables",1368,"asset_receivable","vn_template","True"
"chart1381","Shortage of assets awaiting resolution",1381,"asset_receivable","vn_template","True"
"chart1385","Receivables from privatization",1385,"asset_receivable","vn_template","True"
"chart1388","Other receivables",1388,"asset_receivable","vn_template","True"
"chart141","Advances",141,"asset_receivable","vn_template","True"
"chart151","Goods in transit",151,"asset_current","vn_template","False"
"chart152","Raw materials",152,"asset_current","vn_template","False"
"chart1531","Tools and supplies",1531,"asset_current","vn_template","False"
"chart1532","Reusable packaging materials",1532,"asset_current","vn_template","False"
"chart1533","Instruments for renting",1533,"asset_current","vn_template","False"
"chart1534","Equipment and spare parts for replacement",1534,"asset_current","vn_template","False"
"chart1541","Construction contracts",1541,"asset_current","vn_template","False"
"chart1542","Other work-in-progress products",1542,"asset_current","vn_template","False"
"chart1543","Services",1543,"asset_current","vn_template","False"
"chart1544","Warranty costs",1544,"asset_current","vn_template","False"
"chart1551","Finished products - inventory",1551,"asset_current","vn_template","False"
"chart1557","Finished products - real estates",1557,"asset_current","vn_template","False"
"chart1561","Purchase costs",1561,"asset_current","vn_template","False"
"chart1562","Incidental purchase costs",1562,"asset_current","vn_template","False"
"chart1567","Properties held for sale",1567,"asset_current","vn_template","False"
"chart157","Outward goods on consignment",157,"asset_current","vn_template","False"
"chart158","Goods in bonded warehouse",158,"asset_current","vn_template","False"
"chart1611","Expenditure brought forward",1611,"asset_current","vn_template","False"
"chart1612","Expenditure of current year",1612,"asset_current","vn_template","False"
"chart171","Government bonds purchased for resale",171,"asset_current","vn_template","False"
"chart2111","Buildings and structures",2111,"asset_non_current","vn_template","False"
"chart2112","Machinery and equipment",2112,"asset_non_current","vn_template","False"
"chart2113","Means of transportation and transmission",2113,"asset_non_current","vn_template","False"
"chart2114","Office equipment and furniture",2114,"asset_non_current","vn_template","False"
"chart2115","Perennial plants, working animals and farm livestocks",2115,"asset_non_current","vn_template","False"
"chart2118","Other fixed assets",2118,"asset_non_current","vn_template","False"
"chart2121","Finance lease tangible fixed assets",2121,"asset_non_current","vn_template","False"
"chart2122","Finance lease intangible fixed assets",2122,"asset_non_current","vn_template","False"
"chart2131","Land use rights",2131,"asset_non_current","vn_template","False"
"chart2132","Copyrights",2132,"asset_non_current","vn_template","False"
"chart2133","Patents and inventions",2133,"asset_non_current","vn_template","False"
"chart2134","Product labels and trademarks",2134,"asset_non_current","vn_template","False"
"chart2135","Computer software",2135,"asset_non_current","vn_template","False"
"chart2136","Licenses and franchises",2136,"asset_non_current","vn_template","False"
"chart2138","Other intangible fixed assets",2138,"asset_non_current","vn_template","False"
"chart2141","Depreciation of tangible fixed assets",2141,"asset_non_current","vn_template","False"
"chart2142","Depreciation of finance lease assets",2142,"asset_non_current","vn_template","False"
"chart2143","Amortization of intangible assets",2143,"asset_non_current","vn_template","False"
"chart2147","Depreciation of investment properties",2147,"asset_non_current","vn_template","False"
"chart217","Investment properties",217,"asset_non_current","vn_template","False"
"chart221","Investment in subsidiaries",221,"asset_non_current","vn_template","False"
"chart222","Investment in joint ventures and associates",222,"asset_non_current","vn_template","False"
"chart2281","Equity investments in other entities Other investment",2281,"asset_non_current","vn_template","False"
"chart2291","Allowances for decline in value of trading securities",2291,"asset_non_current","vn_template","False"
"chart2292","Allowances for impairment of investments in other entities",2292,"asset_non_current","vn_template","False"
"chart2293","Allowances for doubtful debts",2293,"asset_non_current","vn_template","False"
"chart2294","Allowances for inventories",2294,"asset_non_current","vn_template","False"
"chart2411","Fixed assets prior to commissioning",2411,"asset_non_current","vn_template","False"
"chart2412","Construction works",2412,"asset_non_current","vn_template","False"
"chart2413","Major repairs of fixed assets",2413,"asset_non_current","vn_template","False"
"chart242","Prepaid expenses",242,"asset_non_current","vn_template","False"
"chart243","Deferred tax assets",243,"asset_non_current","vn_template","False"
"chart244","Mortgage, collaterals and deposits",244,"asset_non_current","vn_template","False"
"chart331","Trade payables",331,"liability_payable","vn_template","True"
"chart33311","Output VAT",33311,"liability_current","vn_template","False"
"chart33312","VAT on imported goods",33312,"liability_current","vn_template","False"
"chart3332","Special consumption tax",3332,"liability_current","vn_template","False"
"chart3333","Import and export tax",3333,"liability_current","vn_template","False"
"chart3334","Corporate income tax",3334,"liability_current","vn_template","False"
"chart3335","Personal income tax",3335,"liability_current","vn_template","False"
"chart3336","Tax on use of natural resources ",3336,"liability_current","vn_template","False"
"chart3337","Land and housing tax, and rental charges",3337,"liability_current","vn_template","False"
"chart33381","Environment protection tax",33381,"liability_current","vn_template","False"
"chart33382","Other taxes",33382,"liability_current","vn_template","False"
"chart3339","Fees, charges and other payables",3339,"liability_payable","vn_template","True"
"chart3341","Payables to staff",3341,"liability_payable","vn_template","True"
"chart3348","Payables to others",3348,"liability_payable","vn_template","True"
"chart335","Accrued expenses",335,"liability_payable","vn_template","True"
"chart3361","Intra-company payables for operating capital received",3361,"liability_payable","vn_template","True"
"chart3362","Intra-company payables for foreign exchange differences",3362,"liability_payable","vn_template","True"
"chart3363","Intra-company payables for borrowing costs eligible to be capitalized",3363,"liability_payable","vn_template","True"
"chart3368","Other inter-company payables",3368,"liability_payable","vn_template","True"
"chart337","Progress billings for construction contracts",337,"liability_payable","vn_template","True"
"chart3381","Surplus of assets awaiting resolution",3381,"liability_payable","vn_template","True"
"chart3382","Trade union fees",3382,"liability_payable","vn_template","True"
"chart3383","Social insurance",3383,"liability_payable","vn_template","True"
"chart3384","Health insurance",3384,"liability_payable","vn_template","True"
"chart3385","Payables on equitization",3385,"liability_payable","vn_template","True"
"chart3386","Unemployment insurance",3386,"liability_payable","vn_template","True"
"chart3387","Unearned revenue",3387,"liability_payable","vn_template","True"
"chart3388","Other payables",3388,"liability_payable","vn_template","True"
"chart3411","Borrowing loans liabilities",3411,"liability_current","vn_template","False"
"chart3412","Finance lease liabilities",3412,"liability_current","vn_template","False"
"chart3431","Ordinary bonds",3431,"liability_current","vn_template","False"
"chart34311","Par value of bonds",34311,"liability_current","vn_template","False"
"chart34312","Bond discounts",34312,"liability_current","vn_template","False"
"chart34313","Bond premiums",34313,"liability_current","vn_template","False"
"chart3432","Convertible bonds",3432,"liability_current","vn_template","False"
"chart344","Deposits received",344,"liability_current","vn_template","False"
"chart347","Deferred tax liabilities",347,"liability_current","vn_template","False"
"chart3521","Product warranty provisions",3521,"liability_current","vn_template","False"
"chart3522","Construction warranty provisions",3522,"liability_current","vn_template","False"
"chart3523","Enterprise restructuring provisions",3523,"liability_current","vn_template","False"
"chart3524","Other provisions",3524,"liability_current","vn_template","False"
"chart3531","Bonus fund",3531,"liability_current","vn_template","False"
"chart3532","Welfare fund",3532,"liability_current","vn_template","False"
"chart3533","Welfare fund used for fixed asset acquisitions",3533,"liability_current","vn_template","False"
"chart3534","Management bonus fund",3534,"liability_current","vn_template","False"
"chart3561","Science and technology development fund",3561,"liability_current","vn_template","False"
"chart3562","Science and technology development fund used for fixed asset acquisition",3562,"liability_current","vn_template","False"
"chart357","Price stabilization fund",357,"liability_current","vn_template","False"
"chart41111","Ordinary shares with voting rights",41111,"equity","vn_template","False"
"chart41112","Preference shares",41112,"equity","vn_template","False"
"chart4112","Capital surplus",4112,"equity","vn_template","False"
"chart4113","Conversion options on convertible bonds",4113,"equity","vn_template","False"
"chart4118","Other capital",4118,"equity","vn_template","False"
"chart412","Differences upon asset revaluation",412,"equity","vn_template","False"
"chart4131","Exchange rate differences on revaluation of monetary items denominated in foreign currency ",4131,"equity","vn_template","False"
"chart4132","Exchange rate differences in pre-operating period",4132,"equity","vn_template","False"
"chart414","Investment and development fund",414,"equity","vn_template","False"
"chart417","Enterprise reorganization assistance fund",417,"equity","vn_template","False"
"chart418","Other equity funds",418,"equity","vn_template","False"
"chart419","Treasury shares",419,"equity","vn_template","False"
"chart4211","Undistributed profit after tax brought forward",4211,"equity","vn_template","False"
"chart4212","Undistributed profit(loss) after tax for the current year",4212,"equity","vn_template","False"
"chart441","Capital expenditure funds",441,"equity","vn_template","False"
"chart4611","Non-business funds bought forward",4611,"equity","vn_template","False"
"chart4612","Non-business funds for current year",4612,"equity","vn_template","False"
"chart466","Non-business funds used for fixed asset acquisitions",466,"equity","vn_template","False"
"chart5111","Revenue from sales of merchandises",5111,"income","vn_template","False"
"chart5112","Revenue from sales of finished goods",5112,"income","vn_template","False"
"chart5113","Revenue from services rendered",5113,"income","vn_template","False"
"chart5114","Revenue from government grants",5114,"income","vn_template","False"
"chart5117","Revenue from investment properties",5117,"income","vn_template","False"
"chart5118","Other revenue",5118,"income","vn_template","False"
"chart515","Financial income",515,"income","vn_template","False"
"chart5211","Trade discounts",5211,"income","vn_template","False"
"chart5212","Sales returns",5212,"income","vn_template","False"
"chart5213","Sales rebates",5213,"income","vn_template","False"
"chart6111","Purchases of raw materials",6111,"expense_direct_cost","vn_template","False"
"chart621","Direct raw material costs",621,"expense_direct_cost","vn_template","False"
"chart622","Direct labour costs",622,"expense_direct_cost","vn_template","False"
"chart6231","Labour costs",6231,"expense","vn_template","False"
"chart6232","Material costs",6232,"expense","vn_template","False"
"chart6233","Production tools and instruments",6233,"expense","vn_template","False"
"chart6234","Depreciation expense",6234,"expense","vn_template","False"
"chart6237","Outside services ",6237,"expense","vn_template","False"
"chart6238","Other expenses",6238,"expense","vn_template","False"
"chart6271","Factory staff costs",6271,"expense","vn_template","False"
"chart6272","Material costs",6272,"expense","vn_template","False"
"chart6273","Production tools and instruments",6273,"expense","vn_template","False"
"chart6274","Fixed asset depreciation",6274,"expense","vn_template","False"
"chart6277","Outside services",6277,"expense","vn_template","False"
"chart6278","Other expenses",6278,"expense","vn_template","False"
"chart631","Production costs",631,"expense_direct_cost","vn_template","False"
"chart632","Costs of goods sold",632,"expense_direct_cost","vn_template","False"
"chart635","Financial expenses",635,"expense","vn_template","False"
"chart6411","Staff expenses",6411,"expense","vn_template","False"
"chart6412","Materials and packing materials",6412,"expense","vn_template","False"
"chart6413","Tools and instruments",6413,"expense","vn_template","False"
"chart6414","Fixed asset deprecation",6414,"expense","vn_template","False"
"chart6415","Warranty expenses",6415,"expense","vn_template","False"
"chart6417","Outside services",6417,"expense","vn_template","False"
"chart6418","Other expenses",6418,"expense","vn_template","False"
"chart6421","Staff expenses",6421,"expense","vn_template","False"
"chart6422","Office supply expenses",6422,"expense","vn_template","False"
"chart6423","Office equipment expenses",6423,"expense","vn_template","False"
"chart6424","Fixed asset depreciation",6424,"expense","vn_template","False"
"chart6425","Taxes, fees and charges",6425,"expense","vn_template","False"
"chart6426","Provision expenses",6426,"expense","vn_template","False"
"chart6427","Outside services",6427,"expense","vn_template","False"
"chart6428","Other expenses",6428,"expense","vn_template","False"
"chart711","Other Income",711,"income_other","vn_template","False"
"chart811","Other Expenses",811,"expense","vn_template","False"
"chart8211","Current tax expense",8211,"expense","vn_template","False"
"chart8212","Deferred tax expense",8212,"expense","vn_template","False"
"chart911","Income Summary",911,"equity_unaffected","vn_template","False"
"chart9993","Cash Discount Loss",9993,"expense","vn_template","False"
"chart9994","Cash Discount Income",9994,"income_other","vn_template","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_vn.vn_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
﻿<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Tax Definitions -->
    <!-- for purchase -->
    <record id="tax_purchase_vat10" model="account.tax.template">
        <field name="chart_template_id" ref="vn_template"/>
        <field name="name">Deductible VAT 10%</field>
        <field name="description">Deductible VAT 10%</field>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_03_02_01_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'plus_report_expression_ids': [ref('account_tax_report_line_03_01_01_vn_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_03_02_01_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'minus_report_expression_ids': [ref('account_tax_report_line_03_01_01_vn_tag')],
                }),
            ]"/>
    </record>
    <record id="tax_purchase_vat5" model="account.tax.template">
        <field name="chart_template_id" ref="vn_template"/>
        <field name="name">Deductible VAT 5%</field>
        <field name="description">Deductible VAT 5%</field>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_02_02_01_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'plus_report_expression_ids': [ref('account_tax_report_line_02_01_01_vn_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_02_02_01_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'minus_report_expression_ids': [ref('account_tax_report_line_02_01_01_vn_tag')],
                }),
            ]"/>
    </record>
    <record id="tax_purchase_vat0" model="account.tax.template">
        <field name="chart_template_id" ref="vn_template"/>
        <field name="name">Deductible VAT 0%</field>
        <field name="description">Deductible VAT 0%</field>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_01_02_01_vn_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_01_02_01_vn_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <!-- for sale -->
    <record id="tax_sale_vat10" model="account.tax.template">
        <field name="chart_template_id" ref="vn_template"/>
        <field name="name">Value Added Tax (VAT) 10%</field>
        <field name="description">Value Added Tax (VAT) 10%</field>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_03_02_02_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'plus_report_expression_ids': [ref('account_tax_report_line_03_01_02_vn_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_03_02_02_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'minus_report_expression_ids': [ref('account_tax_report_line_03_01_02_vn_tag')],
                }),
            ]"/>
    </record>
    <record id="tax_sale_vat5" model="account.tax.template">
        <field name="chart_template_id" ref="vn_template"/>
        <field name="name">Value Added Tax (VAT) 5%</field>
        <field name="description">Value Added Tax (VAT) 5%</field>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_02_02_02_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'plus_report_expression_ids': [ref('account_tax_report_line_02_01_02_vn_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_02_02_02_vn_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'minus_report_expression_ids': [ref('account_tax_report_line_02_01_02_vn_tag')],
                }),
            ]"/>
    </record>
    <record id="tax_sale_vat0" model="account.tax.template">
        <field name="chart_template_id" ref="vn_template"/>
        <field name="name">Value Added Tax (VAT) 0%</field>
        <field name="description">Value Added Tax (VAT) 0%</field>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_01_02_02_vn_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_01_02_02_vn_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
            <field name="country_id" ref="base.vn"/>
        </record>
        <record id="tax_group_5" model="account.tax.group">
            <field name="name">VAT 5%</field>
            <field name="country_id" ref="base.vn"/>
        </record>
        <record id="tax_group_10" model="account.tax.group">
            <field name="name">VAT 10%</field>
            <field name="country_id" ref="base.vn"/>
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
        <field name="country_id" ref="base.vn"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_01_vn" model="account.report.line">
                <field name="name">Purchase of Goods and Services</field>
                <field name="aggregation_formula">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES.balance + UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_01_01_vn" model="account.report.line">
                        <field name="name">VAT on purchase of goods and services</field>
                        <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_0.balance + VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_5.balance + VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_10.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 0%</field>
                                <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 5%</field>
                                <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 10%</field>
                                <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 10%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_02_01_vn" model="account.report.line">
                        <field name="name">Untaxed Purchase of Goods and Services</field>
                        <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_0.balance + UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_5.balance + UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_10.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_02_01_vn" model="account.report.line">
                                <field name="name">Untaxed Purchase of Goods and Services taxed 0%</field>
                                <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_02_01_vn" model="account.report.line">
                                <field name="name">Untaxed Purchase of Goods and Services taxed 5%</field>
                                <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_02_01_vn" model="account.report.line">
                                <field name="name">Untaxed Purchase of Goods and Services taxed 10%</field>
                                <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 10%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_02_vn" model="account.report.line">
                <field name="name">Sales of Goods and Services</field>
                <field name="aggregation_formula">VAT_ON_SALES_OF_GOODS_AND_SERVICES.balance + UNTAXED_SALES_OF_GOODS_AND_SERVICES.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_01_02_vn" model="account.report.line">
                        <field name="name">VAT on sales of goods and services</field>
                        <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">VAT_ON_SALES_OF_GOODS_AND_SERVICES_0.balance + VAT_ON_SALES_OF_GOODS_AND_SERVICES_5.balance + VAT_ON_SALES_OF_GOODS_AND_SERVICES_10.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 0%</field>
                                <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 5%</field>
                                <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 10%</field>
                                <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 10%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_02_02_vn" model="account.report.line">
                        <field name="name">Untaxed Sales of Goods and Services</field>
                        <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_0.balance + UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_5.balance + UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_10.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_02_02_vn" model="account.report.line">
                                <field name="name">Untaxed sales of goods and services taxed 0%</field>
                                <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_02_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_02_02_vn" model="account.report.line">
                                <field name="name">Untaxed sales of goods and services taxed 5%</field>
                                <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_02_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_02_02_vn" model="account.report.line">
                                <field name="name">Untaxed sales of goods and services taxed 10%</field>
                                <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_02_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 10%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_vn_chart_data.xml

```xml
﻿<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <!-- Account Chart Templates -->
    <record id="vn_template" model="account.chart.template">
        <field name="name">VN - Chart of Accounts</field>
        <field name="code_digits">0</field>
        <field name="currency_id" ref="base.VND"/>
        <field name="bank_account_code_prefix">112</field>
        <field name="cash_account_code_prefix">111</field>
        <field name="transfer_account_code_prefix">113</field>
        <field name="spoken_languages" eval="'vi_VN'"/>
        <field name="country_id" ref="base.vn"/>
    </record>
</data>
</odoo>

```

## File: data\l10n_vn_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="vn_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="chart131"/>
        <field name="property_account_payable_id" ref="chart331"/>
        <field name="property_account_expense_categ_id" ref="chart1561"/>
        <field name="property_account_income_categ_id" ref="chart5111"/>
        <field name="income_currency_exchange_account_id" ref="chart515"/>
        <field name="expense_currency_exchange_account_id" ref="chart635"/>
        <field name="default_pos_receivable_account_id" ref="chart131"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="chart9993"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="chart9994"/>
    </record>
</odoo>

```

## File: i18n_extra\l10n_vn.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_vn
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 14.0\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2021-09-30 03:34+0000\n"
"PO-Revision-Date: 2021-09-30 03:34+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart335
#: model:account.account.template,name:l10n_vn.chart335
msgid "Accrued expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart141
#: model:account.account.template,name:l10n_vn.chart141
msgid "Advances"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2291
#: model:account.account.template,name:l10n_vn.chart2291
msgid "Allowances for decline in value of trading securities"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2293
#: model:account.account.template,name:l10n_vn.chart2293
msgid "Allowances for doubtful debts"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2292
#: model:account.account.template,name:l10n_vn.chart2292
msgid "Allowances for impairment of investments in other entities"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2294
#: model:account.account.template,name:l10n_vn.chart2294
msgid "Allowances for inventories"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2143
#: model:account.account.template,name:l10n_vn.chart2143
msgid "Amortization of intangible assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart34312
#: model:account.account.template,name:l10n_vn.chart34312
msgid "Bond discounts"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart34313
#: model:account.account.template,name:l10n_vn.chart34313
msgid "Bond premiums"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1212
#: model:account.account,name:l10n_vn.1_chart1282
#: model:account.account.template,name:l10n_vn.chart1212
#: model:account.account.template,name:l10n_vn.chart1282
msgid "Bonds"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3531
#: model:account.account.template,name:l10n_vn.chart3531
msgid "Bonus fund"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3411
#: model:account.account.template,name:l10n_vn.chart3411
msgid "Borrowing loans liabilities"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2111
#: model:account.account.template,name:l10n_vn.chart2111
msgid "Buildings and structures"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart441
#: model:account.account.template,name:l10n_vn.chart441
msgid "Capital expenditure funds"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4112
#: model:account.account.template,name:l10n_vn.chart4112
msgid "Capital surplus"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2135
#: model:account.account.template,name:l10n_vn.chart2135
msgid "Computer software"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1541
#: model:account.account.template,name:l10n_vn.chart1541
msgid "Construction contracts"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3522
#: model:account.account.template,name:l10n_vn.chart3522
msgid "Construction warranty provisions"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2412
#: model:account.account.template,name:l10n_vn.chart2412
msgid "Construction works"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4113
#: model:account.account.template,name:l10n_vn.chart4113
msgid "Conversion options on convertible bonds"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3432
#: model:account.account.template,name:l10n_vn.chart3432
msgid "Convertible bonds"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2132
#: model:account.account.template,name:l10n_vn.chart2132
msgid "Copyrights"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3334
#: model:account.account.template,name:l10n_vn.chart3334
msgid "Corporate income tax"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart632
#: model:account.account.template,name:l10n_vn.chart632
msgid "Costs of goods sold"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart8211
#: model:account.account.template,name:l10n_vn.chart8211
msgid "Current tax expense"
msgstr ""

#. module: l10n_vn
#: model:account.tax,description:l10n_vn.1_tax_purchase_vat0
#: model:account.tax,name:l10n_vn.1_tax_purchase_vat0
#: model:account.tax.template,description:l10n_vn.tax_purchase_vat0
#: model:account.tax.template,name:l10n_vn.tax_purchase_vat0
msgid "Deductible VAT 0%"
msgstr ""

#. module: l10n_vn
#: model:account.tax,description:l10n_vn.1_tax_purchase_vat10
#: model:account.tax,name:l10n_vn.1_tax_purchase_vat10
#: model:account.tax.template,description:l10n_vn.tax_purchase_vat10
#: model:account.tax.template,name:l10n_vn.tax_purchase_vat10
msgid "Deductible VAT 10%"
msgstr ""

#. module: l10n_vn
#: model:account.tax,description:l10n_vn.1_tax_purchase_vat5
#: model:account.tax,name:l10n_vn.1_tax_purchase_vat5
#: model:account.tax.template,description:l10n_vn.tax_purchase_vat5
#: model:account.tax.template,name:l10n_vn.tax_purchase_vat5
msgid "Deductible VAT 5%"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart243
#: model:account.account.template,name:l10n_vn.chart243
msgid "Deferred tax assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart8212
#: model:account.account.template,name:l10n_vn.chart8212
msgid "Deferred tax expense"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart347
#: model:account.account.template,name:l10n_vn.chart347
msgid "Deferred tax liabilities"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart344
#: model:account.account.template,name:l10n_vn.chart344
msgid "Deposits received"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6234
#: model:account.account.template,name:l10n_vn.chart6234
msgid "Depreciation expense"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2142
#: model:account.account.template,name:l10n_vn.chart2142
msgid "Depreciation of finance lease assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2147
#: model:account.account.template,name:l10n_vn.chart2147
msgid "Depreciation of investment properties"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2141
#: model:account.account.template,name:l10n_vn.chart2141
msgid "Depreciation of tangible fixed assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart412
#: model:account.account.template,name:l10n_vn.chart412
msgid "Differences upon asset revaluation"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart622
#: model:account.account.template,name:l10n_vn.chart622
msgid "Direct labour costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart621
#: model:account.account.template,name:l10n_vn.chart621
msgid "Direct raw material costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart417
#: model:account.account.template,name:l10n_vn.chart417
msgid "Enterprise reorganization assistance fund"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3523
#: model:account.account.template,name:l10n_vn.chart3523
msgid "Enterprise restructuring provisions"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart33381
#: model:account.account.template,name:l10n_vn.chart33381
msgid "Environment protection tax"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1534
#: model:account.account.template,name:l10n_vn.chart1534
msgid "Equipment and spare parts for replacement"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2281
#: model:account.account.template,name:l10n_vn.chart2281
msgid "Equity investments in other entities Other investment"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4132
#: model:account.account.template,name:l10n_vn.chart4132
msgid "Exchange rate differences in pre-operating period"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4131
#: model:account.account.template,name:l10n_vn.chart4131
msgid ""
"Exchange rate differences on revaluation of monetary items denominated in "
"foreign currency "
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1611
#: model:account.account.template,name:l10n_vn.chart1611
msgid "Expenditure brought forward"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1612
#: model:account.account.template,name:l10n_vn.chart1612
msgid "Expenditure of current year"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6271
#: model:account.account.template,name:l10n_vn.chart6271
msgid "Factory staff costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3339
#: model:account.account.template,name:l10n_vn.chart3339
msgid "Fees, charges and other payables"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2122
#: model:account.account.template,name:l10n_vn.chart2122
msgid "Finance lease intangible fixed assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3412
#: model:account.account.template,name:l10n_vn.chart3412
msgid "Finance lease liabilities"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2121
#: model:account.account.template,name:l10n_vn.chart2121
msgid "Finance lease tangible fixed assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart635
#: model:account.account.template,name:l10n_vn.chart635
msgid "Financial expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart515
#: model:account.account.template,name:l10n_vn.chart515
msgid "Financial income"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1551
#: model:account.account.template,name:l10n_vn.chart1551
msgid "Finished products - inventory"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1557
#: model:account.account.template,name:l10n_vn.chart1557
msgid "Finished products - real estates"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6414
#: model:account.account.template,name:l10n_vn.chart6414
msgid "Fixed asset deprecation"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6274
#: model:account.account,name:l10n_vn.1_chart6424
#: model:account.account.template,name:l10n_vn.chart6274
#: model:account.account.template,name:l10n_vn.chart6424
msgid "Fixed asset depreciation"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2411
#: model:account.account.template,name:l10n_vn.chart2411
msgid "Fixed assets prior to commissioning"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1122
#: model:account.account.template,name:l10n_vn.chart1122
msgid "Foreign currencies"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart158
#: model:account.account.template,name:l10n_vn.chart158
msgid "Goods in bonded warehouse"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart151
#: model:account.account.template,name:l10n_vn.chart151
msgid "Goods in transit"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart171
#: model:account.account.template,name:l10n_vn.chart171
msgid "Government bonds purchased for resale"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3384
#: model:account.account.template,name:l10n_vn.chart3384
msgid "Health insurance"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3333
#: model:account.account.template,name:l10n_vn.chart3333
msgid "Import and export tax"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1562
#: model:account.account.template,name:l10n_vn.chart1562
msgid "Incidental purchase costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart911
#: model:account.account.template,name:l10n_vn.chart911
msgid "Income Summary"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1533
#: model:account.account.template,name:l10n_vn.chart1533
msgid "Instruments for renting"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3363
#: model:account.account.template,name:l10n_vn.chart3363
msgid "Intra-company payables for borrowing costs eligible to be capitalized"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3362
#: model:account.account.template,name:l10n_vn.chart3362
msgid "Intra-company payables for foreign exchange differences"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3361
#: model:account.account.template,name:l10n_vn.chart3361
msgid "Intra-company payables for operating capital received"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1363
#: model:account.account.template,name:l10n_vn.chart1363
msgid ""
"Intra-company receivables on borrowing costs eligible to be capitalized"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1362
#: model:account.account.template,name:l10n_vn.chart1362
msgid "Intra-company receivables on foreign exchange"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart414
#: model:account.account.template,name:l10n_vn.chart414
msgid "Investment and development fund"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart222
#: model:account.account.template,name:l10n_vn.chart222
msgid "Investment in joint ventures and associates"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart221
#: model:account.account.template,name:l10n_vn.chart221
msgid "Investment in subsidiaries"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart217
#: model:account.account.template,name:l10n_vn.chart217
msgid "Investment properties"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6231
#: model:account.account.template,name:l10n_vn.chart6231
msgid "Labour costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3337
#: model:account.account.template,name:l10n_vn.chart3337
msgid "Land and housing tax, and rental charges"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2131
#: model:account.account.template,name:l10n_vn.chart2131
msgid "Land use rights"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1283
#: model:account.account.template,name:l10n_vn.chart1283
msgid "Lending loans"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2136
#: model:account.account.template,name:l10n_vn.chart2136
msgid "Licenses and franchises"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_vn_template_liquidity_transfer
#: model:account.account.template,name:l10n_vn.vn_template_liquidity_transfer
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2112
#: model:account.account.template,name:l10n_vn.chart2112
msgid "Machinery and equipment"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2413
#: model:account.account.template,name:l10n_vn.chart2413
msgid "Major repairs of fixed assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3534
#: model:account.account.template,name:l10n_vn.chart3534
msgid "Management bonus fund"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6232
#: model:account.account,name:l10n_vn.1_chart6272
#: model:account.account.template,name:l10n_vn.chart6232
#: model:account.account.template,name:l10n_vn.chart6272
msgid "Material costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6412
#: model:account.account.template,name:l10n_vn.chart6412
msgid "Materials and packing materials"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2113
#: model:account.account.template,name:l10n_vn.chart2113
msgid "Means of transportation and transmission"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1123
#: model:account.account.template,name:l10n_vn.chart1123
msgid "Monetary Gold"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart244
#: model:account.account.template,name:l10n_vn.chart244
msgid "Mortgage, collaterals and deposits"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4611
#: model:account.account.template,name:l10n_vn.chart4611
msgid "Non-business funds bought forward"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4612
#: model:account.account.template,name:l10n_vn.chart4612
msgid "Non-business funds for current year"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart466
#: model:account.account.template,name:l10n_vn.chart466
msgid "Non-business funds used for fixed asset acquisitions"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2114
#: model:account.account.template,name:l10n_vn.chart2114
msgid "Office equipment and furniture"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6423
#: model:account.account.template,name:l10n_vn.chart6423
msgid "Office equipment expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6422
#: model:account.account.template,name:l10n_vn.chart6422
msgid "Office supply expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3431
#: model:account.account.template,name:l10n_vn.chart3431
msgid "Ordinary bonds"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart41111
#: model:account.account.template,name:l10n_vn.chart41111
msgid "Ordinary shares with voting rights"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart811
#: model:account.account.template,name:l10n_vn.chart811
msgid "Other Expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart711
#: model:account.account.template,name:l10n_vn.chart711
msgid "Other Income"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4118
#: model:account.account.template,name:l10n_vn.chart4118
msgid "Other capital"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart418
#: model:account.account.template,name:l10n_vn.chart418
msgid "Other equity funds"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6238
#: model:account.account,name:l10n_vn.1_chart6278
#: model:account.account,name:l10n_vn.1_chart6418
#: model:account.account,name:l10n_vn.1_chart6428
#: model:account.account.template,name:l10n_vn.chart6238
#: model:account.account.template,name:l10n_vn.chart6278
#: model:account.account.template,name:l10n_vn.chart6418
#: model:account.account.template,name:l10n_vn.chart6428
msgid "Other expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2118
#: model:account.account.template,name:l10n_vn.chart2118
msgid "Other fixed assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1288
#: model:account.account.template,name:l10n_vn.chart1288
msgid "Other held to maturity investments"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2138
#: model:account.account.template,name:l10n_vn.chart2138
msgid "Other intangible fixed assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3368
#: model:account.account.template,name:l10n_vn.chart3368
msgid "Other inter-company payables"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1368
#: model:account.account.template,name:l10n_vn.chart1368
msgid "Other intra-company receivables"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1542
#: model:account.account.template,name:l10n_vn.chart1542
msgid "Other products"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3524
#: model:account.account.template,name:l10n_vn.chart3524
msgid "Other provisions"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5118
#: model:account.account.template,name:l10n_vn.chart5118
msgid "Other revenue"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1218
#: model:account.account.template,name:l10n_vn.chart1218
msgid "Other securities and financial instruments"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart33382
#: model:account.account.template,name:l10n_vn.chart33382
msgid "Other taxes"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3388
#: model:account.account.template,name:l10n_vn.chart3388
msgid "Others"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1388
#: model:account.account.template,name:l10n_vn.chart1388
msgid "Others receivables"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart33311
#: model:account.account.template,name:l10n_vn.chart33311
msgid "Output VAT"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6277
#: model:account.account,name:l10n_vn.1_chart6417
#: model:account.account,name:l10n_vn.1_chart6427
#: model:account.account.template,name:l10n_vn.chart6277
#: model:account.account.template,name:l10n_vn.chart6417
#: model:account.account.template,name:l10n_vn.chart6427
msgid "Outside services"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6237
#: model:account.account.template,name:l10n_vn.chart6237
msgid "Outside services "
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart157
#: model:account.account.template,name:l10n_vn.chart157
msgid "Outward goods on consignment"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart34311
#: model:account.account.template,name:l10n_vn.chart34311
msgid "Par value of bonds"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2133
#: model:account.account.template,name:l10n_vn.chart2133
msgid "Patents and inventions"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3385
#: model:account.account.template,name:l10n_vn.chart3385
msgid "Payables on equitization"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3348
#: model:account.account.template,name:l10n_vn.chart3348
msgid "Payables to others"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3341
#: model:account.account.template,name:l10n_vn.chart3341
msgid "Payables to staff"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2115
#: model:account.account.template,name:l10n_vn.chart2115
msgid "Perennial plants, working animals and farm livestocks"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3335
#: model:account.account.template,name:l10n_vn.chart3335
msgid "Personal income tax"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart41112
#: model:account.account.template,name:l10n_vn.chart41112
msgid "Preference shares"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart242
#: model:account.account.template,name:l10n_vn.chart242
msgid "Prepaid expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart357
#: model:account.account.template,name:l10n_vn.chart357
msgid "Price stabilization fund"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart2134
#: model:account.account.template,name:l10n_vn.chart2134
msgid "Product labels and trademarks"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3521
#: model:account.account.template,name:l10n_vn.chart3521
msgid "Product warranty provisions"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart631
#: model:account.account.template,name:l10n_vn.chart631
msgid "Production costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart337
#: model:account.account.template,name:l10n_vn.chart337
msgid "Progress billings for construction contracts"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1567
#: model:account.account.template,name:l10n_vn.chart1567
msgid "Properties held for sale"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6426
#: model:account.account.template,name:l10n_vn.chart6426
msgid "Provision expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1561
#: model:account.account.template,name:l10n_vn.chart1561
msgid "Purchase costs"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_vn
msgid "Purchase of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6111
#: model:account.account.template,name:l10n_vn.chart6111
msgid "Purchases of raw materials"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart152
#: model:account.account.template,name:l10n_vn.chart152
msgid "Raw materials"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1385
#: model:account.account.template,name:l10n_vn.chart1385
msgid "Receivables from privatization"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1532
#: model:account.account.template,name:l10n_vn.chart1532
msgid "Reusable packaging materials"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5114
#: model:account.account.template,name:l10n_vn.chart5114
msgid "Revenue from government grants"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5117
#: model:account.account.template,name:l10n_vn.chart5117
msgid "Revenue from investment properties"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5112
#: model:account.account.template,name:l10n_vn.chart5112
msgid "Revenue from sales of finished goods"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5111
#: model:account.account.template,name:l10n_vn.chart5111
msgid "Revenue from sales of merchandises"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5113
#: model:account.account.template,name:l10n_vn.chart5113
msgid "Revenue from services rendered"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_vn
msgid "Sales of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5213
#: model:account.account.template,name:l10n_vn.chart5213
msgid "Sales rebates"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5212
#: model:account.account.template,name:l10n_vn.chart5212
msgid "Sales returns"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3561
#: model:account.account.template,name:l10n_vn.chart3561
msgid "Science and technology development fund"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3562
#: model:account.account.template,name:l10n_vn.chart3562
msgid ""
"Science and technology development fund used for fixed asset acquisition"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1543
#: model:account.account.template,name:l10n_vn.chart1543
msgid "Services"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1211
#: model:account.account.template,name:l10n_vn.chart1211
msgid "Shares"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1381
#: model:account.account.template,name:l10n_vn.chart1381
msgid "Shortage of assets awaiting resolution"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3383
#: model:account.account.template,name:l10n_vn.chart3383
msgid "Social insurance"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3332
#: model:account.account.template,name:l10n_vn.chart3332
msgid "Special consumption tax"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6411
#: model:account.account,name:l10n_vn.1_chart6421
#: model:account.account.template,name:l10n_vn.chart6411
#: model:account.account.template,name:l10n_vn.chart6421
msgid "Staff expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3381
#: model:account.account.template,name:l10n_vn.chart3381
msgid "Surplus of assets awaiting resolution"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3336
#: model:account.account.template,name:l10n_vn.chart3336
msgid "Tax on use of natural resources "
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6425
#: model:account.account.template,name:l10n_vn.chart6425
msgid "Taxes, fees and charges"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1281
#: model:account.account.template,name:l10n_vn.chart1281
msgid "Term deposits"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6233
#: model:account.account,name:l10n_vn.1_chart6273
#: model:account.account,name:l10n_vn.1_chart6413
#: model:account.account.template,name:l10n_vn.chart6233
#: model:account.account.template,name:l10n_vn.chart6273
#: model:account.account.template,name:l10n_vn.chart6413
msgid "Tools and instruments"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1531
#: model:account.account.template,name:l10n_vn.chart1531
msgid "Tools and supplies"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart5211
#: model:account.account.template,name:l10n_vn.chart5211
msgid "Trade discounts"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart331
#: model:account.account.template,name:l10n_vn.chart331
msgid "Trade payables"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart131
#: model:account.account.template,name:l10n_vn.chart131
msgid "Trade receivables"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart132
#: model:account.account.template,name:l10n_vn.chart132
msgid "Trade receivables(pos)"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3382
#: model:account.account.template,name:l10n_vn.chart3382
msgid "Trade union fees"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart419
#: model:account.account.template,name:l10n_vn.chart419
msgid "Treasury shares"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4211
#: model:account.account.template,name:l10n_vn.chart4211
msgid "Undistributed profit after tax brought forward"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart4212
#: model:account.account.template,name:l10n_vn.chart4212
msgid "Undistributed profit(loss) after tax for the current year"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3387
#: model:account.account.template,name:l10n_vn.chart3387
msgid "Unearned revenue"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3386
#: model:account.account.template,name:l10n_vn.chart3386
msgid "Unemployment insurance"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_01_vn
msgid "Untaxed Purchase of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_02_01_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_01_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_02_01_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_03_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_02_01_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_02_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 5%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_02_vn
msgid "Untaxed Sales of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_02_02_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_01_02_02_vn
msgid "Untaxed sales of goods and services taxed 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_02_02_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_03_02_02_vn
msgid "Untaxed sales of goods and services taxed 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_02_02_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_02_02_02_vn
msgid "Untaxed sales of goods and services taxed 5%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.group,name:l10n_vn.tax_group_0
msgid "VAT 0%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.group,name:l10n_vn.tax_group_10
msgid "VAT 10%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.group,name:l10n_vn.tax_group_5
msgid "VAT 5%"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart33312
#: model:account.account.template,name:l10n_vn.chart33312
msgid "VAT on imported goods"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1332
#: model:account.account.template,name:l10n_vn.chart1332
msgid "VAT on purchase of fixed assets"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1331
#: model:account.account.template,name:l10n_vn.chart1331
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_01_vn
msgid "VAT on purchase of goods and services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_01_01_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_01_01_01_vn
msgid "VAT on purchase of goods and services 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_01_01_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_03_01_01_vn
msgid "VAT on purchase of goods and services 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_01_01_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_02_01_01_vn
msgid "VAT on purchase of goods and services 5%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_02_vn
msgid "VAT on sales of goods and services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_01_02_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_01_01_02_vn
msgid "VAT on sales of goods and services 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_01_02_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_03_01_02_vn
msgid "VAT on sales of goods and services 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_01_02_vn
#: model:account.report.line,tag_name:l10n_vn.account_tax_report_line_02_01_02_vn
msgid "VAT on sales of goods and services 5%"
msgstr ""

#. module: l10n_vn
#: model:account.chart.template,name:l10n_vn.vn_template
msgid "VN - Chart of Accounts"
msgstr ""

#. module: l10n_vn
#: model:account.tax,description:l10n_vn.1_tax_sale_vat0
#: model:account.tax,name:l10n_vn.1_tax_sale_vat0
#: model:account.tax.template,description:l10n_vn.tax_sale_vat0
#: model:account.tax.template,name:l10n_vn.tax_sale_vat0
msgid "Value Added Tax (VAT) 0%"
msgstr ""

#. module: l10n_vn
#: model:account.tax,description:l10n_vn.1_tax_sale_vat10
#: model:account.tax,name:l10n_vn.1_tax_sale_vat10
#: model:account.tax.template,description:l10n_vn.tax_sale_vat10
#: model:account.tax.template,name:l10n_vn.tax_sale_vat10
msgid "Value Added Tax (VAT) 10%"
msgstr ""

#. module: l10n_vn
#: model:account.tax,description:l10n_vn.1_tax_sale_vat5
#: model:account.tax,name:l10n_vn.1_tax_sale_vat5
#: model:account.tax.template,description:l10n_vn.tax_sale_vat5
#: model:account.tax.template,name:l10n_vn.tax_sale_vat5
msgid "Value Added Tax (VAT) 5%"
msgstr ""

#. module: l10n_vn
#: model:ir.ui.menu,name:l10n_vn.account_reports_vn_statements_menu
msgid "Vietnam"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1121
#: model:account.account.template,name:l10n_vn.chart1121
msgid "Vietnamese Dong"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1544
#: model:account.account.template,name:l10n_vn.chart1544
msgid "Warranty costs"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart6415
#: model:account.account.template,name:l10n_vn.chart6415
msgid "Warranty expenses"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3532
#: model:account.account.template,name:l10n_vn.chart3532
msgid "Welfare fund"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart3533
#: model:account.account.template,name:l10n_vn.chart3533
msgid "Welfare fund used for fixed asset acquisitions"
msgstr ""

#. module: l10n_vn
#: model:account.account,name:l10n_vn.1_chart1361
#: model:account.account.template,name:l10n_vn.chart1361
msgid "Working capital provided to sub-units"
msgstr ""

```

## File: migrations\14.0.2.0.1\post-migration.py

```python
# -*- coding: utf-8 -*-
from odoo import api, SUPERUSER_ID

FIXED_ACCOUNTS_MAP = {
    '5221': '5211',
    '5222': '5212',
    '5223': '5213'
    }


def _fix_revenue_deduction_accounts_code(env):
    vn_template = env.ref('l10n_vn.vn_template')
    for company in env['res.company'].with_context(active_test=False).search([('chart_template_id', '=', vn_template.id)]):
        for incorrect_code, correct_code in FIXED_ACCOUNTS_MAP.items():
            account = env['account.account'].search([('code', '=', incorrect_code), ('company_id', '=', company.id)])
            if account:
                account.write({'code': correct_code})


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    _fix_revenue_deduction_accounts_code(env)

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
      <image width="1024" height="683" transform="translate(4.8 6.07) scale(0.05 0.05)" xlink:href="data:image/png;base64,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oAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAAAAoAIGAAAAAKiAAQAAAAAqYAAAAACAChgAAADg/2/HDgQAAAAABPlbD3JhBDAgAAAAAGBAAAAAAMCAAAAAAIABAQAAAAADAgAAAAAGBAAAAAAMCAAAAAAYEAAAAAAwIAAAAABgQAAAAADAgAAAAACAAQEAAAAAAwIAAAAABgQAAAAADAgAAAAAGBAAAAAAMCAAAAAAYEAAAAAAwIAAAAAAgAEBAAAAAAMCAAAAAAYEAAAAAAwIAAAAABgQAAAAADAgAAAAAGBAAAAAAMCAAAAAAIABAQAAAAADAgAAAAAGBAAAAAAMCAAAAAAYEAAAAAAwIAAAAABgQAAAAADAgAAAAACAAQEAAAAAAwIAAAAABgQAAAAADAgAAAAAGBAAAAAAMCAAAAAAYEAAAAAAwIAAAAAAgAEBAAAAAAMCAAAAAAYEAAAAAAwIAAAAABgQAAAAADAgAAAAAGBAAAAAAMCAAAAAAIABAQAAAAADAgAAAAAGBAAAAAAMCAAAAAAYEAAAAAAwIAAAAABgQAAAAADAgAAAAACAAQEAAAAAAwIAAAAABgQAAAAADAgAAAAAGBAAAAAAMCAAAAAAYEAAAAAAwIAAAAAAgAEBAAAAAAMCAAAAAAYEAAAAAAwIAAAAABgIpmWZYW+IbkwAAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

