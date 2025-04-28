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
         'data/account_data.xml',
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
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"chart1121","Vietnamese Dong","1121","account.data_account_type_liquidity","vn_template","False"
"chart1122","Foreign currencies","1122","account.data_account_type_liquidity","vn_template","False"
"chart1123","Monetary Gold","1123","account.data_account_type_liquidity","vn_template","False"
"chart1211","Shares",1211,"account.data_account_type_current_assets","vn_template","False"
"chart1212","Bonds",1212,"account.data_account_type_current_assets","vn_template","False"
"chart1218","Other securities and financial instruments",1218,"account.data_account_type_current_assets","vn_template","False"
"chart1281","Term deposits",1281,"account.data_account_type_current_assets","vn_template","False"
"chart1282","Bonds",1282,"account.data_account_type_current_assets","vn_template","False"
"chart1283","Lending loans",1283,"account.data_account_type_current_assets","vn_template","False"
"chart1288","Other held to maturity investments",1288,"account.data_account_type_current_assets","vn_template","False"
"chart131","Trade receivables",131,"account.data_account_type_receivable","vn_template","True"
"chart132","Trade receivables(pos)",132,"account.data_account_type_receivable","vn_template","True"
"chart1331","VAT on purchase of goods and services",1331,"account.data_account_type_current_assets","vn_template","False"
"chart1332","VAT on purchase of fixed assets",1332,"account.data_account_type_current_assets","vn_template","False"
"chart1361","Working capital provided to sub-units",1361,"account.data_account_type_receivable","vn_template","True"
"chart1362","Intra-company receivables on foreign exchange",1362,"account.data_account_type_receivable","vn_template","True"
"chart1363","Intra-company receivables on borrowing costs eligible to be capitalized",1363,"account.data_account_type_receivable","vn_template","True"
"chart1368","Other intra-company receivables",1368,"account.data_account_type_receivable","vn_template","True"
"chart1381","Shortage of assets awaiting resolution",1381,"account.data_account_type_receivable","vn_template","True"
"chart1385","Receivables from privatization",1385,"account.data_account_type_receivable","vn_template","True"
"chart1388","Others receivables",1388,"account.data_account_type_receivable","vn_template","True"
"chart141","Advances",141,"account.data_account_type_receivable","vn_template","True"
"chart151","Goods in transit",151,"account.data_account_type_current_assets","vn_template","False"
"chart152","Raw materials",152,"account.data_account_type_current_assets","vn_template","False"
"chart1531","Tools and supplies",1531,"account.data_account_type_current_assets","vn_template","False"
"chart1532","Reusable packaging materials",1532,"account.data_account_type_current_assets","vn_template","False"
"chart1533","Instruments for renting",1533,"account.data_account_type_current_assets","vn_template","False"
"chart1534","Equipment and spare parts for replacement",1534,"account.data_account_type_current_assets","vn_template","False"
"chart1541","Construction contracts",1541,"account.data_account_type_current_assets","vn_template","False"
"chart1542","Other products",1542,"account.data_account_type_current_assets","vn_template","False"
"chart1543","Services",1543,"account.data_account_type_current_assets","vn_template","False"
"chart1544","Warranty costs",1544,"account.data_account_type_current_assets","vn_template","False"
"chart1551","Finished products - inventory",1551,"account.data_account_type_current_assets","vn_template","False"
"chart1557","Finished products - real estates",1557,"account.data_account_type_current_assets","vn_template","False"
"chart1561","Purchase costs",1561,"account.data_account_type_current_assets","vn_template","False"
"chart1562","Incidental purchase costs",1562,"account.data_account_type_current_assets","vn_template","False"
"chart1567","Properties held for sale",1567,"account.data_account_type_current_assets","vn_template","False"
"chart157","Outward goods on consignment",157,"account.data_account_type_current_assets","vn_template","False"
"chart158","Goods in bonded warehouse",158,"account.data_account_type_current_assets","vn_template","False"
"chart1611","Expenditure brought forward",1611,"account.data_account_type_current_assets","vn_template","False"
"chart1612","Expenditure of current year",1612,"account.data_account_type_current_assets","vn_template","False"
"chart171","Government bonds purchased for resale",171,"account.data_account_type_current_assets","vn_template","False"
"chart2111","Buildings and structures",2111,"account.data_account_type_non_current_assets","vn_template","False"
"chart2112","Machinery and equipment",2112,"account.data_account_type_non_current_assets","vn_template","False"
"chart2113","Means of transportation and transmission",2113,"account.data_account_type_non_current_assets","vn_template","False"
"chart2114","Office equipment and furniture",2114,"account.data_account_type_non_current_assets","vn_template","False"
"chart2115","Perennial plants, working animals and farm livestocks",2115,"account.data_account_type_non_current_assets","vn_template","False"
"chart2118","Other fixed assets",2118,"account.data_account_type_non_current_assets","vn_template","False"
"chart2121","Finance lease tangible fixed assets",2121,"account.data_account_type_non_current_assets","vn_template","False"
"chart2122","Finance lease intangible fixed assets",2122,"account.data_account_type_non_current_assets","vn_template","False"
"chart2131","Land use rights",2131,"account.data_account_type_non_current_assets","vn_template","False"
"chart2132","Copyrights",2132,"account.data_account_type_non_current_assets","vn_template","False"
"chart2133","Patents and inventions",2133,"account.data_account_type_non_current_assets","vn_template","False"
"chart2134","Product labels and trademarks",2134,"account.data_account_type_non_current_assets","vn_template","False"
"chart2135","Computer software",2135,"account.data_account_type_non_current_assets","vn_template","False"
"chart2136","Licenses and franchises",2136,"account.data_account_type_non_current_assets","vn_template","False"
"chart2138","Other intangible fixed assets",2138,"account.data_account_type_non_current_assets","vn_template","False"
"chart2141","Depreciation of tangible fixed assets",2141,"account.data_account_type_non_current_assets","vn_template","False"
"chart2142","Depreciation of finance lease assets",2142,"account.data_account_type_non_current_assets","vn_template","False"
"chart2143","Amortization of intangible assets",2143,"account.data_account_type_non_current_assets","vn_template","False"
"chart2147","Depreciation of investment properties",2147,"account.data_account_type_non_current_assets","vn_template","False"
"chart217","Investment properties",217,"account.data_account_type_non_current_assets","vn_template","False"
"chart221","Investment in subsidiaries",221,"account.data_account_type_non_current_assets","vn_template","False"
"chart222","Investment in joint ventures and associates",222,"account.data_account_type_non_current_assets","vn_template","False"
"chart2281","Equity investments in other entities Other investment",2281,"account.data_account_type_non_current_assets","vn_template","False"
"chart2291","Allowances for decline in value of trading securities",2291,"account.data_account_type_non_current_assets","vn_template","False"
"chart2292","Allowances for impairment of investments in other entities",2292,"account.data_account_type_non_current_assets","vn_template","False"
"chart2293","Allowances for doubtful debts",2293,"account.data_account_type_non_current_assets","vn_template","False"
"chart2294","Allowances for inventories",2294,"account.data_account_type_non_current_assets","vn_template","False"
"chart2411","Fixed assets prior to commissioning",2411,"account.data_account_type_non_current_assets","vn_template","False"
"chart2412","Construction works",2412,"account.data_account_type_non_current_assets","vn_template","False"
"chart2413","Major repairs of fixed assets",2413,"account.data_account_type_non_current_assets","vn_template","False"
"chart242","Prepaid expenses",242,"account.data_account_type_non_current_assets","vn_template","False"
"chart243","Deferred tax assets",243,"account.data_account_type_non_current_assets","vn_template","False"
"chart244","Mortgage, collaterals and deposits",244,"account.data_account_type_non_current_assets","vn_template","False"
"chart331","Trade payables",331,"account.data_account_type_payable","vn_template","True"
"chart33311","Output VAT",33311,"account.data_account_type_current_liabilities","vn_template","False"
"chart33312","VAT on imported goods",33312,"account.data_account_type_current_liabilities","vn_template","False"
"chart3332","Special consumption tax",3332,"account.data_account_type_current_liabilities","vn_template","False"
"chart3333","Import and export tax",3333,"account.data_account_type_current_liabilities","vn_template","False"
"chart3334","Corporate income tax",3334,"account.data_account_type_current_liabilities","vn_template","False"
"chart3335","Personal income tax",3335,"account.data_account_type_current_liabilities","vn_template","False"
"chart3336","Tax on use of natural resources ",3336,"account.data_account_type_current_liabilities","vn_template","False"
"chart3337","Land and housing tax, and rental charges",3337,"account.data_account_type_current_liabilities","vn_template","False"
"chart33381","Environment protection tax",33381,"account.data_account_type_current_liabilities","vn_template","False"
"chart33382","Other taxes",33382,"account.data_account_type_current_liabilities","vn_template","False"
"chart3339","Fees, charges and other payables",3339,"account.data_account_type_payable","vn_template","True"
"chart3341","Payables to staff",3341,"account.data_account_type_payable","vn_template","True"
"chart3348","Payables to others",3348,"account.data_account_type_payable","vn_template","True"
"chart335","Accrued expenses",335,"account.data_account_type_payable","vn_template","True"
"chart3361","Intra-company payables for operating capital received",3361,"account.data_account_type_payable","vn_template","True"
"chart3362","Intra-company payables for foreign exchange differences",3362,"account.data_account_type_payable","vn_template","True"
"chart3363","Intra-company payables for borrowing costs eligible to be capitalized",3363,"account.data_account_type_payable","vn_template","True"
"chart3368","Other inter-company payables",3368,"account.data_account_type_payable","vn_template","True"
"chart337","Progress billings for construction contracts",337,"account.data_account_type_payable","vn_template","True"
"chart3381","Surplus of assets awaiting resolution",3381,"account.data_account_type_payable","vn_template","True"
"chart3382","Trade union fees",3382,"account.data_account_type_payable","vn_template","True"
"chart3383","Social insurance",3383,"account.data_account_type_payable","vn_template","True"
"chart3384","Health insurance",3384,"account.data_account_type_payable","vn_template","True"
"chart3385","Payables on equitization",3385,"account.data_account_type_payable","vn_template","True"
"chart3386","Unemployment insurance",3386,"account.data_account_type_payable","vn_template","True"
"chart3387","Unearned revenue",3387,"account.data_account_type_payable","vn_template","True"
"chart3388","Others",3388,"account.data_account_type_payable","vn_template","True"
"chart3411","Borrowing loans liabilities",3411,"account.data_account_type_current_liabilities","vn_template","False"
"chart3412","Finance lease liabilities",3412,"account.data_account_type_current_liabilities","vn_template","False"
"chart3431","Ordinary bonds",3431,"account.data_account_type_current_liabilities","vn_template","False"
"chart34311","Par value of bonds",34311,"account.data_account_type_current_liabilities","vn_template","False"
"chart34312","Bond discounts",34312,"account.data_account_type_current_liabilities","vn_template","False"
"chart34313","Bond premiums",34313,"account.data_account_type_current_liabilities","vn_template","False"
"chart3432","Convertible bonds",3432,"account.data_account_type_current_liabilities","vn_template","False"
"chart344","Deposits received",344,"account.data_account_type_current_liabilities","vn_template","False"
"chart347","Deferred tax liabilities",347,"account.data_account_type_current_liabilities","vn_template","False"
"chart3521","Product warranty provisions",3521,"account.data_account_type_current_liabilities","vn_template","False"
"chart3522","Construction warranty provisions",3522,"account.data_account_type_current_liabilities","vn_template","False"
"chart3523","Enterprise restructuring provisions",3523,"account.data_account_type_current_liabilities","vn_template","False"
"chart3524","Other provisions",3524,"account.data_account_type_current_liabilities","vn_template","False"
"chart3531","Bonus fund",3531,"account.data_account_type_current_liabilities","vn_template","False"
"chart3532","Welfare fund",3532,"account.data_account_type_current_liabilities","vn_template","False"
"chart3533","Welfare fund used for fixed asset acquisitions",3533,"account.data_account_type_current_liabilities","vn_template","False"
"chart3534","Management bonus fund",3534,"account.data_account_type_current_liabilities","vn_template","False"
"chart3561","Science and technology development fund",3561,"account.data_account_type_current_liabilities","vn_template","False"
"chart3562","Science and technology development fund used for fixed asset acquisition",3562,"account.data_account_type_current_liabilities","vn_template","False"
"chart357","Price stabilization fund",357,"account.data_account_type_current_liabilities","vn_template","False"
"chart41111","Ordinary shares with voting rights",41111,"account.data_account_type_equity","vn_template","False"
"chart41112","Preference shares",41112,"account.data_account_type_equity","vn_template","False"
"chart4112","Capital surplus",4112,"account.data_account_type_equity","vn_template","False"
"chart4113","Conversion options on convertible bonds",4113,"account.data_account_type_equity","vn_template","False"
"chart4118","Other capital",4118,"account.data_account_type_equity","vn_template","False"
"chart412","Differences upon asset revaluation",412,"account.data_account_type_equity","vn_template","False"
"chart4131","Exchange rate differences on revaluation of monetary items denominated in foreign currency ",4131,"account.data_account_type_equity","vn_template","False"
"chart4132","Exchange rate differences in pre-operating period",4132,"account.data_account_type_equity","vn_template","False"
"chart414","Investment and development fund",414,"account.data_account_type_equity","vn_template","False"
"chart417","Enterprise reorganization assistance fund",417,"account.data_account_type_equity","vn_template","False"
"chart418","Other equity funds",418,"account.data_account_type_equity","vn_template","False"
"chart419","Treasury shares",419,"account.data_account_type_equity","vn_template","False"
"chart4211","Undistributed profit after tax brought forward",4211,"account.data_account_type_equity","vn_template","False"
"chart4212","Undistributed profit(loss) after tax for the current year",4212,"account.data_account_type_equity","vn_template","False"
"chart441","Capital expenditure funds",441,"account.data_account_type_equity","vn_template","False"
"chart4611","Non-business funds bought forward",4611,"account.data_account_type_equity","vn_template","False"
"chart4612","Non-business funds for current year",4612,"account.data_account_type_equity","vn_template","False"
"chart466","Non-business funds used for fixed asset acquisitions",466,"account.data_account_type_equity","vn_template","False"
"chart5111","Revenue from sales of merchandises",5111,"account.data_account_type_revenue","vn_template","False"
"chart5112","Revenue from sales of finished goods",5112,"account.data_account_type_revenue","vn_template","False"
"chart5113","Revenue from services rendered",5113,"account.data_account_type_revenue","vn_template","False"
"chart5114","Revenue from government grants",5114,"account.data_account_type_revenue","vn_template","False"
"chart5117","Revenue from investment properties",5117,"account.data_account_type_revenue","vn_template","False"
"chart5118","Other revenue",5118,"account.data_account_type_revenue","vn_template","False"
"chart515","Financial income",515,"account.data_account_type_revenue","vn_template","False"
"chart5211","Trade discounts",5211,"account.data_account_type_revenue","vn_template","False"
"chart5212","Sales returns",5212,"account.data_account_type_revenue","vn_template","False"
"chart5213","Sales rebates",5213,"account.data_account_type_revenue","vn_template","False"
"chart6111","Purchases of raw materials",6111,"account.data_account_type_direct_costs","vn_template","False"
"chart621","Direct raw material costs",621,"account.data_account_type_direct_costs","vn_template","False"
"chart622","Direct labour costs",622,"account.data_account_type_direct_costs","vn_template","False"
"chart6231","Labour costs",6231,"account.data_account_type_expenses","vn_template","False"
"chart6232","Material costs",6232,"account.data_account_type_expenses","vn_template","False"
"chart6233","Production tools and instruments",6233,"account.data_account_type_expenses","vn_template","False"
"chart6234","Depreciation expense",6234,"account.data_account_type_expenses","vn_template","False"
"chart6237","Outside services ",6237,"account.data_account_type_expenses","vn_template","False"
"chart6238","Other expenses",6238,"account.data_account_type_expenses","vn_template","False"
"chart6271","Factory staff costs",6271,"account.data_account_type_expenses","vn_template","False"
"chart6272","Material costs",6272,"account.data_account_type_expenses","vn_template","False"
"chart6273","Production tools and instruments",6273,"account.data_account_type_expenses","vn_template","False"
"chart6274","Fixed asset depreciation",6274,"account.data_account_type_expenses","vn_template","False"
"chart6277","Outside services",6277,"account.data_account_type_expenses","vn_template","False"
"chart6278","Other expenses",6278,"account.data_account_type_expenses","vn_template","False"
"chart631","Production costs",631,"account.data_account_type_direct_costs","vn_template","False"
"chart632","Costs of goods sold",632,"account.data_account_type_direct_costs","vn_template","False"
"chart635","Financial expenses",635,"account.data_account_type_expenses","vn_template","False"
"chart6411","Staff expenses",6411,"account.data_account_type_expenses","vn_template","False"
"chart6412","Materials and packing materials",6412,"account.data_account_type_expenses","vn_template","False"
"chart6413","Tools and instruments",6413,"account.data_account_type_expenses","vn_template","False"
"chart6414","Fixed asset deprecation",6414,"account.data_account_type_expenses","vn_template","False"
"chart6415","Warranty expenses",6415,"account.data_account_type_expenses","vn_template","False"
"chart6417","Outside services",6417,"account.data_account_type_expenses","vn_template","False"
"chart6418","Other expenses",6418,"account.data_account_type_expenses","vn_template","False"
"chart6421","Staff expenses",6421,"account.data_account_type_expenses","vn_template","False"
"chart6422","Office supply expenses",6422,"account.data_account_type_expenses","vn_template","False"
"chart6423","Office equipment expenses",6423,"account.data_account_type_expenses","vn_template","False"
"chart6424","Fixed asset depreciation",6424,"account.data_account_type_expenses","vn_template","False"
"chart6425","Taxes, fees and charges",6425,"account.data_account_type_expenses","vn_template","False"
"chart6426","Provision expenses",6426,"account.data_account_type_expenses","vn_template","False"
"chart6427","Outside services",6427,"account.data_account_type_expenses","vn_template","False"
"chart6428","Other expenses",6428,"account.data_account_type_expenses","vn_template","False"
"chart711","Other Income",711,"account.data_account_type_other_income","vn_template","False"
"chart811","Other Expenses",811,"account.data_account_type_expenses","vn_template","False"
"chart8211","Current tax expense",8211,"account.data_account_type_expenses","vn_template","False"
"chart8212","Deferred tax expense",8212,"account.data_account_type_expenses","vn_template","False"
"chart911","Income Summary",911,"account.data_unaffected_earnings","vn_template","False"

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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
        </record>
        <record id="tax_group_5" model="account.tax.group">
            <field name="name">VAT 5%</field>
        </record>
        <record id="tax_group_10" model="account.tax.group">
            <field name="name">VAT 10%</field>
        </record>

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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_03_02_01_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'plus_report_line_ids': [ref('account_tax_report_line_03_01_01_vn')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_03_02_01_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'minus_report_line_ids': [ref('account_tax_report_line_03_01_01_vn')],
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_02_02_01_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'plus_report_line_ids': [ref('account_tax_report_line_02_01_01_vn')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_02_02_01_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart1331'),
                    'minus_report_line_ids': [ref('account_tax_report_line_02_01_01_vn')],
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_01_02_01_vn')],
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
                    'minus_report_line_ids': [ref('account_tax_report_line_01_02_01_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_03_02_02_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'plus_report_line_ids': [ref('account_tax_report_line_03_01_02_vn')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_03_02_02_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'minus_report_line_ids': [ref('account_tax_report_line_03_01_02_vn')],
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_02_02_02_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'plus_report_line_ids': [ref('account_tax_report_line_02_01_02_vn')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_02_02_02_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart33311'),
                    'minus_report_line_ids': [ref('account_tax_report_line_02_01_02_vn')],
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_01_02_02_vn')],
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
                    'minus_report_line_ids': [ref('account_tax_report_line_01_02_02_vn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.vn"/>
    </record>

     <record id="account_tax_report_line_01_vn" model="account.tax.report.line">
        <field name="name">Purchase of Goods and Services</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="account_tax_report_line_01_01_vn" model="account.tax.report.line">
        <field name="name">VAT on purchase of goods and services</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_01_vn"/>
    </record>

    <record id="account_tax_report_line_01_01_01_vn" model="account.tax.report.line">
        <field name="name">VAT on purchase of goods and services 0%</field>
        <field name="tag_name">VAT on purchase of goods and services 0%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_01_01_vn"/>
    </record>

    <record id="account_tax_report_line_02_01_01_vn" model="account.tax.report.line">
        <field name="name">VAT on purchase of goods and services 5%</field>
        <field name="tag_name">VAT on purchase of goods and services 5%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_01_01_vn"/>
    </record>

    <record id="account_tax_report_line_03_01_01_vn" model="account.tax.report.line">
        <field name="name">VAT on purchase of goods and services 10%</field>
        <field name="tag_name">VAT on purchase of goods and services 10%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_01_01_vn"/>
    </record>

    <record id="account_tax_report_line_02_01_vn" model="account.tax.report.line">
        <field name="name">Untaxed Purchase of Goods and Services</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_01_vn"/>
    </record>

    <record id="account_tax_report_line_01_02_01_vn" model="account.tax.report.line">
        <field name="name">Untaxed Purchase of Goods and Services taxed 0%</field>
        <field name="tag_name">Untaxed Purchase of Goods and Services taxed 0%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_02_01_vn"/>
    </record>

    <record id="account_tax_report_line_02_02_01_vn" model="account.tax.report.line">
        <field name="name">Untaxed Purchase of Goods and Services taxed 5%</field>
        <field name="tag_name">Untaxed Purchase of Goods and Services taxed 5%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_02_01_vn"/>
    </record>

    <record id="account_tax_report_line_03_02_01_vn" model="account.tax.report.line">
        <field name="name">Untaxed Purchase of Goods and Services taxed 10%</field>
        <field name="tag_name">Untaxed Purchase of Goods and Services taxed 10%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_02_01_vn"/>
    </record>

    <record id="account_tax_report_line_02_vn" model="account.tax.report.line">
        <field name="name">Sales of Goods and Services</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="account_tax_report_line_01_02_vn" model="account.tax.report.line">
        <field name="name">VAT on sales of goods and services</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_02_vn"/>
    </record>

    <record id="account_tax_report_line_01_01_02_vn" model="account.tax.report.line">
        <field name="name">VAT on sales of goods and services 0%</field>
        <field name="tag_name">VAT on sales of goods and services 0%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_01_02_vn"/>
    </record>

    <record id="account_tax_report_line_02_01_02_vn" model="account.tax.report.line">
        <field name="name">VAT on sales of goods and services 5%</field>
        <field name="tag_name">VAT on sales of goods and services 5%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_01_02_vn"/>
    </record>

    <record id="account_tax_report_line_03_01_02_vn" model="account.tax.report.line">
        <field name="name">VAT on sales of goods and services 10%</field>
        <field name="tag_name">VAT on sales of goods and services 10%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_01_02_vn"/>
    </record>

    <record id="account_tax_report_line_02_02_vn" model="account.tax.report.line">
        <field name="name">Untaxed Sales of Goods and Services</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_02_vn"/>
    </record>

    <record id="account_tax_report_line_01_02_02_vn" model="account.tax.report.line">
        <field name="name">Untaxed sales of goods and services taxed 0%</field>
        <field name="tag_name">Untaxed sales of goods and services taxed 0%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_02_02_vn"/>
    </record>

    <record id="account_tax_report_line_02_02_02_vn" model="account.tax.report.line">
        <field name="name">Untaxed sales of goods and services taxed 5%</field>
        <field name="tag_name">Untaxed sales of goods and services taxed 5%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_02_02_vn"/>
    </record>

    <record id="account_tax_report_line_03_02_02_vn" model="account.tax.report.line">
        <field name="name">Untaxed sales of goods and services taxed 10%</field>
        <field name="tag_name">Untaxed sales of goods and services taxed 10%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_02_02_vn"/>
    </record>
</odoo>

```

## File: data\l10n_vn_chart_data.xml

```xml
﻿<?xml version="1.0" encoding="utf-8"?>
<odoo>
<menuitem id="account_reports_vn_statements_menu" name="Vietnam" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

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
        <field name="use_anglo_saxon" eval="False"/>
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
        <field name="default_pos_receivable_account_id" ref="chart132" />
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
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_01_vn
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
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_02_vn
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
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_02_01_vn
msgid "Untaxed Purchase of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_01_02_01_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_01_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 0%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_03_02_01_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_03_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 10%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_02_02_01_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_02_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 5%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_02_02_vn
msgid "Untaxed Sales of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_01_02_02_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_01_02_02_vn
msgid "Untaxed sales of goods and services taxed 0%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_03_02_02_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_03_02_02_vn
msgid "Untaxed sales of goods and services taxed 10%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_02_02_02_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_02_02_02_vn
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
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_01_01_vn
msgid "VAT on purchase of goods and services"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_01_01_01_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_01_01_01_vn
msgid "VAT on purchase of goods and services 0%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_03_01_01_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_03_01_01_vn
msgid "VAT on purchase of goods and services 10%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_02_01_01_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_02_01_01_vn
msgid "VAT on purchase of goods and services 5%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_01_02_vn
msgid "VAT on sales of goods and services"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_01_01_02_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_01_01_02_vn
msgid "VAT on sales of goods and services 0%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_03_01_02_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_03_01_02_vn
msgid "VAT on sales of goods and services 10%"
msgstr ""

#. module: l10n_vn
#: model:account.tax.report.line,name:l10n_vn.account_tax_report_line_02_01_02_vn
#: model:account.tax.report.line,tag_name:l10n_vn.account_tax_report_line_02_01_02_vn
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

