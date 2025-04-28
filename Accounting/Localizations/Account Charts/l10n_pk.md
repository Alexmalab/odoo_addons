# Odoo Module: l10n_pk

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Pakistan - Accounting',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'website': 'https://www.odoo.com/documentation/16.0/applications/finance/fiscal_localizations.html',
    'description': """
Pakistan Accounting Module
=======================================================
Pakistan accounting basic charts and localization.

Activates:

- Chart of Accounts
- Taxes
    """,
    'depends': ['account'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/l10n_pk_chart_data.xml',
        'data/account.group.template.csv',
        'data/account_tax_group.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml'
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","tag_ids/id","reconcile"
"l10n_pk_1111001","Building","1111001","asset_fixed","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1111002","Furniture and Fixture","1111002","asset_fixed","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1111003","Tools and Equipment","1111003","asset_fixed","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1111004","Plant and Machinery","1111004","asset_fixed","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1112001","ERP System","1112001","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1113000","Investment Property","1113000","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1114000","Long Term Investments","1114000","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1115000","Long Term Deposits","1115000","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1116000","Biological Assets","1116000","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1117000","Investments in Associates","1117000","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1118000","Investments in Jointly Controlled Entities","1118000","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1119000","Other Financial Assets","1119000","asset_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1121001","Receivable from Customers","1121001","asset_receivable","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_1122001","Advances to Suppliers","1122001","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1122002","Withholding Tax- Advance","1122002","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1122003","Loan to Employees","1122003","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1123001","Security Deposits","1123001","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1123002","Prepaid Rent","1123002","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1123003","Income Tax Refundable","1123003","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1123004","Sales Tax Refundable","1123004","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1124000","Other Current Assets","1124000","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1125001","Stock in Hand","1125001","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1125002","Work in Process","1125002","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_1125003","Finished Goods","1125003","asset_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2111000","Share Capital","2111000","equity","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2111002","Reserves","2112000","equity","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2113000","Revaluation Surplus on Property and Equipment","2113000","equity","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2211001","Long term Loan","2211001","liability_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2211002","Liabilities Against Assets Subject to Finance Lease","2211002","liability_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2212000","Deferred Liabilities","2212000","liability_non_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2221001","Payable to Suppliers","2221001","liability_payable","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_2221002","Accrued Expenses","2221002","liability_current","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_2221003","Salaries Payable","2221003","liability_payable","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_2221004","Mark-up accrued","2221004","liability_payable","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_2221005","Sales Tax Payable","2221005","liability_current","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_2221006","Withholding Tax Payable","2221006","liability_payable","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_2221007","Payable to Canteen","2221007","liability_payable","l10n_pk.l10n_pk_chart_template","","True",
"l10n_pk_2222001","Current Portion of Long Term Liabilities","2222001","liability_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2223001","Provision for Employee Benefit","2223001","liability_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2223002","Other Provision","2223002","liability_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2224000","Unpaid Dividend","2224000","liability_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2225000","Unclaimed Dividend","2225000","liability_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_2226000","Suspense Account","2226000","liability_current","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_3111001","Sales Income","3111001","income","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_3112001","Bank Profit","3112001","income","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_3112002","Employee Fine","3112002","income","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_3112003","Misc Income","3112003","income_other","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_3112004","Cash Discount Gain","3112004","income_other","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4111001","Purchases","4111001","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4111002","Carriage Inwards","4111002","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4111003","Cost of Goods Sold","4111003","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4211001","Salary and Wages","4211001","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4211002","Other Staff Benefits","4211002","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221001","Rent Rates and Taxes","4221001","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221002","Traveling and Conveyance","4221002","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221003","Vehicle Running Expenses","4221003","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221004","Telephone and Internet","4221004","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221005","Electricity Fitting","4221005","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221006","Printing and Stationery","4221006","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221007","Fee and Subscription","4221007","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221008","Legal and Professional Charges","4221008","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221009","Office Consumables and Supplies","4221009","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221010","Entertainment Expenses","4221010","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221011","Repair and maintenance","4221011","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221012","Postage Expenses","4221012","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221013","Canteen Expenses","4221013","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221014","Bad Debts Written Off","4221014","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221015","Charity and Donations","4221015","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221016","Insurance","4221016","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221017","Misc Expenses","4221017","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221018","Auditor's Remuneration","4221018","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221019","Depreciation Expenses","4221019","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221020","Electricity Expenses","4221020","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4221021","Suspense Account","4221021","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4311001","Advertisement and Marketing","4311001","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4311002","Commission on Sales","4311002","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4311003","Freight Outward","4311003","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4311004","Sample Expense","4311004","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4411001","Bank Charges and Commission","4411001","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4411002","Foreign Exchange Difference","4411002","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4411003","Cash Discount Loss","4411003","expense","l10n_pk.l10n_pk_chart_template","","False",
"l10n_pk_4511000","Income Tax Expense","4511000","expense","l10n_pk.l10n_pk_chart_template","","False",

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
l10n_pk_group_1,1,,"Assets",l10n_pk.l10n_pk_chart_template
l10n_pk_group_111,111,,"Non Current Assets",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1111,1111,,"Property, Plant and Equipment",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1112,1112,,"Intangible Assets",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1113,1113,,"Investment Property",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1114,1114,,"Long Term Investments",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1115,1115,,"Long Term Deposits",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1116,1116,,"Biological Assets",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1117,1117,,"Investments in Associates",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1118,1118,,"Investments in Jointly Controlled Entities",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1119,1119,,"Other Financial Assets",l10n_pk.l10n_pk_chart_template
l10n_pk_group_112,112,,"Current Assets",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1121,1121,,"Trade Receivable",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1122,1122,,"Loans and Advances",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1123,1123,,"Trade Deposits and Short Term Prepayments",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1124,1124,,"Other Current Assets",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1125,1125,,"Stock in Trade",l10n_pk.l10n_pk_chart_template
l10n_pk_group_1126,1126,,"Cash and Bank",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2,2,,"Equity and Liabilities",l10n_pk.l10n_pk_chart_template
l10n_pk_group_21,21,,"Equity",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2111,2111,,"Share Capital",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2112,2112,,"Reserves",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2113,2113,,"Revaluation Surplus on Property and Equipment",l10n_pk.l10n_pk_chart_template
l10n_pk_group_22,22,,"Liabilities",l10n_pk.l10n_pk_chart_template
l10n_pk_group_221,221,,"Non Current Liabilities",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2211,2211,,"Long Term Financing",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2212,2212,,"Deferred Liabilities",l10n_pk.l10n_pk_chart_template
l10n_pk_group_222,222,,"Current Liabilities",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2221,2221,,"Trade and Payables",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2222,2222,,"Short Term Borrowings",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2223,2223,,"Provisions",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2224,2224,,"Unpaid Dividend",l10n_pk.l10n_pk_chart_template
l10n_pk_group_2225,2225,,"Unclaimed Dividend",l10n_pk.l10n_pk_chart_template
l10n_pk_group_3,3,,"Revenue",l10n_pk.l10n_pk_chart_template
l10n_pk_group_3111,3111,,"Sales",l10n_pk.l10n_pk_chart_template
l10n_pk_group_3112,3112,,"Other Income",l10n_pk.l10n_pk_chart_template
l10n_pk_group_4,4,,"Expenses",l10n_pk.l10n_pk_chart_template
l10n_pk_group_41,41,,"Cost of Sales",l10n_pk.l10n_pk_chart_template
l10n_pk_group_42,42,,"General and Administrative Expenses",l10n_pk.l10n_pk_chart_template
l10n_pk_group_421,421,,"Staff Cost",l10n_pk.l10n_pk_chart_template
l10n_pk_group_422,422,,"Other Operating and General Expenses",l10n_pk.l10n_pk_chart_template
l10n_pk_group_43,43,,"Selling and Distribution Expenses",l10n_pk.l10n_pk_chart_template
l10n_pk_group_44,44,,"Banking and Finance Costs",l10n_pk.l10n_pk_chart_template
l10n_pk_group_45,45,,"Income Tax Expense",l10n_pk.l10n_pk_chart_template

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_pk.l10n_pk_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Chart Template -->
    <record id="l10n_pk_chart_template" model="account.chart.template">
        <field name="name">Pakistan Chart of Account</field>
        <field name="code_digits">7</field>
        <field name="bank_account_code_prefix">112600</field>
        <field name="cash_account_code_prefix">112600</field>
        <field name="transfer_account_code_prefix">112600</field>
        <field name="currency_id" ref="base.PKR"/>
        <field name="country_id" ref="base.pk"/>
    </record>
</odoo>

```

## File: data\account_tax_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <!-- Account Tax Group -->
        <record id="tax_group_taxes_sales" model="account.tax.group">
            <field name="name">Sales Taxes</field>
            <field name="country_id" ref="base.pk"/>
        </record>
        <record id="tax_group_taxes_purchases" model="account.tax.group">
            <field name="name">Purchases Taxes</field>
            <field name="country_id" ref="base.pk"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="pk_sales_tax_17" model="account.tax.template">
            <field name="name">Standard Sales Tax 17%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">17</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Sales Tax 17%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_19_5" model="account.tax.template">
            <field name="name">Sales Tax Telecommunication Services 19.5%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">19.5</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Telecommunication Services 19.5%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_17" model="account.tax.template">
            <field name="name">Sales Tax Services 17%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">17</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 17%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_16" model="account.tax.template">
            <field name="name">Sales Tax Services 16%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_16_punjab" model="account.tax.template">
            <field name="name">Standard Sales Service Tax Punjab 16%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Sales Service Tax Punjab 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_16_ict" model="account.tax.template">
            <field name="name">Standard Sales Service Tax ICT 16%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Sales Service Tax Islamabad Capital Territory 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_16_ajk" model="account.tax.template">
            <field name="name">Standard Sales Service Tax AJK 16%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Sales Service Tax Azad Jammu and Kashmir 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_15" model="account.tax.template">
            <field name="name">Sales Tax Services 15%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 15%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_15_kp" model="account.tax.template">
            <field name="name">Standard Sales Service Tax KP 15%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Sales Service Tax Khyber Pakhtunkhwa 15%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_15_balochistan" model="account.tax.template">
            <field name="name">Standard Sales Service Tax Balochistan 15%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Sales Service Tax Balochistan 15%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_13" model="account.tax.template">
            <field name="name">Sales Tax Services 13%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 13%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_13_sindh" model="account.tax.template">
            <field name="name">Standard Sales Tax Services 13% Sindh</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Sales Tax Services 13% Sindh</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_10" model="account.tax.template">
            <field name="name">Sales Tax Services 10%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">10</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 10%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_8" model="account.tax.template">
            <field name="name">Sales Tax Services 8%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 8%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_5" model="account.tax.template">
            <field name="name">Sales Tax Services 5%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">5</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 5%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_3" model="account.tax.template">
            <field name="name">Sales Tax Services 3%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">3</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 3%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_2" model="account.tax.template">
            <field name="name">Sales Tax Services 2%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">2</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 2%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_1" model="account.tax.template">
            <field name="name">Sales Tax Services 1%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">1</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 1%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="pk_sales_tax_services_0" model="account.tax.template">
            <field name="name">Sales Tax Services 0%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="description">Sales Tax Services 0%</field>
            <field name="tax_group_id" ref="tax_group_taxes_sales"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_17" model="account.tax.template">
            <field name="name">Standard Purchases Tax 17%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">17</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Purchases Tax 17%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_19_5" model="account.tax.template">
            <field name="name">Purchases Tax Telecommunication Services 19.5%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">19.5</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Telecommunication Services 19.5%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_17" model="account.tax.template">
            <field name="name">Purchases Tax Services 17%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">17</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 17%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_16" model="account.tax.template">
            <field name="name">Purchases Tax Services 16%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_16_punjab" model="account.tax.template">
            <field name="name">Standard Purchases Service Tax Punjab 16%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Purchases Service Tax Punjab 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_16_ict" model="account.tax.template">
            <field name="name">Standard Purchases Service Tax ICT 16%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Purchases Service Tax Islamabad Capital Territory 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_16_ajk" model="account.tax.template">
            <field name="name">Standard Purchases Service Tax AJK 16%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Purchases Service Tax Azad Jammu and Kashmir 16%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_15" model="account.tax.template">
            <field name="name">Purchases Tax Services 15%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 15%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_15_kp" model="account.tax.template">
            <field name="name">Standard Purchases Service Tax KP 15%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Purchases Service Tax Khyber Pakhtunkhwa 15%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_15_balochistan" model="account.tax.template">
            <field name="name">Standard Purchases Service Tax Balochistan 15%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Purchases Service Tax Balochistan 15%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_13" model="account.tax.template">
            <field name="name">Purchases Tax Services 13%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 13%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_13_sindh" model="account.tax.template">
            <field name="name">Standard Purchases Tax Services 13% Sindh</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="description">Standard Purchases Tax Services 13% Sindh</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_10" model="account.tax.template">
            <field name="name">Purchases Tax Services 10%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">10</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 10%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_8" model="account.tax.template">
            <field name="name">Purchases Tax Services 8%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 8%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_5" model="account.tax.template">
            <field name="name">Purchases Tax Services 5%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">5</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 5%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_3" model="account.tax.template">
            <field name="name">Purchases Tax Services 3%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">3</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 3%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_2" model="account.tax.template">
            <field name="name">Purchases Tax Services 2%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">2</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 2%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_1" model="account.tax.template">
            <field name="name">Purchases Tax Services 1%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">1</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 1%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
        <record id="purchases_tax_services_0" model="account.tax.template">
            <field name="name">Purchases Tax Services 0%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="description">Purchases Tax Services 0%</field>
            <field name="tax_group_id" ref="tax_group_taxes_purchases"/>
            <field name="chart_template_id" ref="l10n_pk_chart_template"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_pk_2221005'),
                }),
            ]"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_pk_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Chart Template -->
    <record id="l10n_pk_chart_template" model="account.chart.template">
        <field name="name">Pakistan - Chart of Accounts</field>
        <field name="property_account_receivable_id" ref="l10n_pk_1121001" />
        <field name="property_account_payable_id" ref="l10n_pk_2221001" />
        <field name="property_account_income_categ_id" ref="l10n_pk_3111001" />
        <field name="property_account_expense_categ_id" ref="l10n_pk_4111001" />
        <field name="account_journal_suspense_account_id" ref="l10n_pk_2226000"/>
        <field name="default_pos_receivable_account_id" ref="l10n_pk_1121001"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="l10n_pk_4411003"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="l10n_pk_3112004"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106"><defs><mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse"><path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill:#fff;fill-rule:evenodd"/></mask><mask id="b" x="4.08" y="7.33" width="48.45" height="32.23" maskUnits="userSpaceOnUse"><rect x="4.08" y="7.61" width="48.45" height="31.57" rx="1" style="fill:#fff"/></mask><symbol id="c" viewBox="0 0 106 106"><g style="mask:url(#a)"><path d="M0,0H106V106H0Z" style="fill:#5a5a64;fill-rule:evenodd"/><path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill:#fff;fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill:#393939;fill-rule:evenodd;isolation:isolate;opacity:0.324000000953674"/><g style="opacity:0.30000000000000004"><path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/><path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/><path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/><path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/><path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/><path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/><path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/><path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/></g><path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill:#a8a9ab"/><path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill:#a8a9ab"/><path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill:#a8a9ab"/><path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill:#a8a9ab"/><path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill:#a8a9ab"/><path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill:#a8a9ab"/><path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill:#a8a9ab"/><path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill:#a8a9ab"/></g></symbol></defs><rect x="4" y="10.57" width="48.45" height="31.57" rx="1" style="fill:#393939;isolation:isolate;opacity:0.44"/><use width="106" height="106" xlink:href="#c"/><g style="mask:url(#b)"><rect x="4.12" y="7.33" width="48.35" height="32.23" style="fill:#fff"/><rect x="16.21" y="7.33" width="36.26" height="32.23" style="fill:#01411c"/><circle cx="34.34" cy="23.45" r="9.67" style="fill:#fff"/><circle cx="36.81" cy="21.25" r="8.86" style="fill:#01411c"/><polygon points="37.86 16.21 41.94 20.8 35.95 19.48 41.56 17.03 38.47 22.32 37.86 16.21" style="fill:#fff"/></g></svg>
```

