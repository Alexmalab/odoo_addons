# Odoo Module: l10n_au

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2015 Willow IT Pty Ltd (<http://www.willowit.com.au>).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2015 Willow IT Pty Ltd (<http://www.willowit.com.au>).

{
    'name': 'Australian - Accounting',
    'version': '1.1',
    'category': 'Localization',
    'description': """
Australian Accounting Module
============================

Australian accounting basic charts and localizations.

Also:
    - activates a number of regional currencies.
    - sets up Australian taxes.
    """,
    'author': 'Richard deMeester - Willow IT',
    'website': 'http://www.willowit.com',
    'depends': ['account'],
    'data': [
             'data/l10n_au_chart_data.xml',
             'data/account.account.template.csv',
             'data/account_chart_template_data.xml',
             'data/account_tax_report_data.xml',
             'data/account.tax.group.csv',
             'data/account_tax_template_data.xml',
             'data/account_fiscal_position_tax_template_data.xml',
             'data/account_chart_template_configure_data.xml',
             'data/res_currency_data.xml',
             ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,chart_template_id:id,code,name,user_type_id:id,reconcile
au_11110,l10n_au.l10n_au_chart_template,11110,Bank Account,account.data_account_type_liquidity,FALSE
au_11130,l10n_au.l10n_au_chart_template,11130,Petty Cash,account.data_account_type_liquidity,FALSE
au_11140,l10n_au.l10n_au_chart_template,11140,Cash Drawer,account.data_account_type_liquidity,FALSE
au_11180,l10n_au.l10n_au_chart_template,11180,Undeposited Funds,account.data_account_type_current_assets,TRUE
au_11190,l10n_au.l10n_au_chart_template,11190,Electronic Clearing Account,account.data_account_type_current_assets,TRUE
au_11200,l10n_au.l10n_au_chart_template,11200,Trade Debtors,account.data_account_type_receivable,TRUE
au_11201,l10n_au.l10n_au_chart_template,11201,Trade Debtors (PoS),account.data_account_type_receivable,TRUE
au_11210,l10n_au.l10n_au_chart_template,11210,Less Prov'n for Doubtful Debts,account.data_account_type_current_assets,FALSE
au_11310,l10n_au.l10n_au_chart_template,11310,Raw Materials,account.data_account_type_current_assets,FALSE
au_11320,l10n_au.l10n_au_chart_template,11320,Finished Goods,account.data_account_type_current_assets,FALSE
au_11330,l10n_au.l10n_au_chart_template,11330,Trading Stock on Hand,account.data_account_type_current_assets,FALSE
au_12100,l10n_au.l10n_au_chart_template,12100,Deposits Paid,account.data_account_type_prepayments,FALSE
au_12200,l10n_au.l10n_au_chart_template,12200,Prepaid Insurance,account.data_account_type_current_assets,FALSE
au_13110,l10n_au.l10n_au_chart_template,13110,Manufacturing Plant at Cost,account.data_account_type_fixed_assets,FALSE
au_13120,l10n_au.l10n_au_chart_template,13120,Manufac. Plant Accum Dep,account.data_account_type_fixed_assets,FALSE
au_13130,l10n_au.l10n_au_chart_template,13130,Manufacturing Equipment Cost,account.data_account_type_fixed_assets,FALSE
au_13140,l10n_au.l10n_au_chart_template,13140,Manufac. Equip Accum Dep,account.data_account_type_fixed_assets,FALSE
au_13210,l10n_au.l10n_au_chart_template,13210,Furniture & Fixtures at Cost,account.data_account_type_fixed_assets,FALSE
au_13220,l10n_au.l10n_au_chart_template,13220,Furniture & Fixtures Accum Dep,account.data_account_type_fixed_assets,FALSE
au_13310,l10n_au.l10n_au_chart_template,13310,Office Equip at Cost,account.data_account_type_fixed_assets,FALSE
au_13320,l10n_au.l10n_au_chart_template,13320,Office Equip Accum Dep,account.data_account_type_fixed_assets,FALSE
au_13410,l10n_au.l10n_au_chart_template,13410,Motor Vehicles at Cost,account.data_account_type_fixed_assets,FALSE
au_13420,l10n_au.l10n_au_chart_template,13420,Motor Vehicles Accum Dep,account.data_account_type_fixed_assets,FALSE
au_21110,l10n_au.l10n_au_chart_template,21110,Credit Card,account.data_account_type_current_liabilities,FALSE
au_21200,l10n_au.l10n_au_chart_template,21200,Trade Creditors,account.data_account_type_payable,TRUE
au_21210,l10n_au.l10n_au_chart_template,21210,A/P Accrual - Inventory,account.data_account_type_current_liabilities,FALSE
au_21310,l10n_au.l10n_au_chart_template,21310,GST Collected,account.data_account_type_current_liabilities,FALSE
au_21330,l10n_au.l10n_au_chart_template,21330,GST Paid,account.data_account_type_current_assets,FALSE
au_21350,l10n_au.l10n_au_chart_template,21350,Fuel Tax Credits Accrued,account.data_account_type_current_liabilities,FALSE
au_21355,l10n_au.l10n_au_chart_template,21355,WET Payable,account.data_account_type_current_liabilities,FALSE
au_21360,l10n_au.l10n_au_chart_template,21360,Import Duty Payable,account.data_account_type_current_liabilities,FALSE
au_21370,l10n_au.l10n_au_chart_template,21370,Voluntary Withholdings Payable,account.data_account_type_current_liabilities,FALSE
au_21380,l10n_au.l10n_au_chart_template,21380,ABN Withholdings Payable,account.data_account_type_current_liabilities,FALSE
au_21410,l10n_au.l10n_au_chart_template,21410,Payroll Accruals Payable,account.data_account_type_current_liabilities,FALSE
au_21420,l10n_au.l10n_au_chart_template,21420,PAYG Withholding Payable,account.data_account_type_current_liabilities,FALSE
au_21600,l10n_au.l10n_au_chart_template,21600,Customer Deposits,account.data_account_type_current_liabilities,FALSE
au_21700,l10n_au.l10n_au_chart_template,21700,Other Current Liabilities,account.data_account_type_current_liabilities,FALSE
au_22100,l10n_au.l10n_au_chart_template,22100,Mortgages Payable,account.data_account_type_non_current_liabilities,FALSE
au_22200,l10n_au.l10n_au_chart_template,22200,Notes Payable,account.data_account_type_non_current_liabilities,FALSE
au_22300,l10n_au.l10n_au_chart_template,22300,Other Long Term Liabilities,account.data_account_type_non_current_liabilities,FALSE
au_31100,l10n_au.l10n_au_chart_template,31100,Capital Investment,account.data_account_type_equity,FALSE
au_31200,l10n_au.l10n_au_chart_template,31200,Capital Drawings,account.data_account_type_equity,FALSE
au_38000,l10n_au.l10n_au_chart_template,38000,Retained Earnings,account.data_account_type_equity,FALSE
au_39000,l10n_au.l10n_au_chart_template,39000,Current Year Earnings,account.data_unaffected_earnings,FALSE
au_39999,l10n_au.l10n_au_chart_template,39999,Historical Balancing,account.data_account_type_equity,FALSE
au_41110,l10n_au.l10n_au_chart_template,41110,Sales Product #1,account.data_account_type_revenue,FALSE
au_41120,l10n_au.l10n_au_chart_template,41120,Sales Product #2,account.data_account_type_revenue,FALSE
au_41130,l10n_au.l10n_au_chart_template,41130,Sales Product #3,account.data_account_type_revenue,FALSE
au_42000,l10n_au.l10n_au_chart_template,42000,Wholesale Sales,account.data_account_type_revenue,FALSE
au_43000,l10n_au.l10n_au_chart_template,43000,Consignment Sales,account.data_account_type_revenue,FALSE
au_44000,l10n_au.l10n_au_chart_template,44000,Freight Income,account.data_account_type_revenue,FALSE
au_45000,l10n_au.l10n_au_chart_template,45000,Late Fees Collected,account.data_account_type_other_income,FALSE
au_46000,l10n_au.l10n_au_chart_template,46000,Miscellaneous Income,account.data_account_type_other_income,FALSE
au_47000,l10n_au.l10n_au_chart_template,47000,Fuel Tax Credits,account.data_account_type_other_income,FALSE
au_51110,l10n_au.l10n_au_chart_template,51110,Cost of Goods Sold #1,account.data_account_type_direct_costs,FALSE
au_51120,l10n_au.l10n_au_chart_template,51120,Cost of Goods Sold # 2,account.data_account_type_direct_costs,FALSE
au_51130,l10n_au.l10n_au_chart_template,51130,Cost of Goods Sold # 3,account.data_account_type_direct_costs,FALSE
au_52000,l10n_au.l10n_au_chart_template,52000,Wholesale Cost of Sales,account.data_account_type_direct_costs,FALSE
au_53000,l10n_au.l10n_au_chart_template,53000,Consignment Cost of Sales,account.data_account_type_direct_costs,FALSE
au_54000,l10n_au.l10n_au_chart_template,54000,Wages for Production Labour,account.data_account_type_direct_costs,FALSE
au_55000,l10n_au.l10n_au_chart_template,55000,Materials & Supplies,account.data_account_type_direct_costs,FALSE
au_56000,l10n_au.l10n_au_chart_template,56000,Freight,account.data_account_type_direct_costs,FALSE
au_57000,l10n_au.l10n_au_chart_template,57000,Other Costs,account.data_account_type_direct_costs,FALSE
au_61000,l10n_au.l10n_au_chart_template,61000,Advertising,account.data_account_type_expenses,FALSE
au_61200,l10n_au.l10n_au_chart_template,61200,Car & Truck Expenses,account.data_account_type_expenses,FALSE
au_61300,l10n_au.l10n_au_chart_template,61300,Commissions Paid,account.data_account_type_expenses,FALSE
au_61500,l10n_au.l10n_au_chart_template,61500,Depreciation Expense,account.data_account_type_depreciation,FALSE
au_61610,l10n_au.l10n_au_chart_template,61610,Discounts Given,account.data_account_type_expenses,FALSE
au_61620,l10n_au.l10n_au_chart_template,61620,Discounts Taken,account.data_account_type_expenses,FALSE
au_61630,l10n_au.l10n_au_chart_template,61630,Exchange Rate Losses (Gains),account.data_account_type_expenses,FALSE
au_61700,l10n_au.l10n_au_chart_template,61700,Freight Paid,account.data_account_type_expenses,FALSE
au_61800,l10n_au.l10n_au_chart_template,61800,Insurance,account.data_account_type_expenses,FALSE
au_61910,l10n_au.l10n_au_chart_template,61910,Overdraft Interest,account.data_account_type_expenses,FALSE
au_61920,l10n_au.l10n_au_chart_template,61920,Mortgage Interest,account.data_account_type_expenses,FALSE
au_61930,l10n_au.l10n_au_chart_template,61930,Other Interest,account.data_account_type_expenses,FALSE
au_62000,l10n_au.l10n_au_chart_template,62000,Late Fees Paid,account.data_account_type_expenses,FALSE
au_62110,l10n_au.l10n_au_chart_template,62110,Machinery & Equipment,account.data_account_type_expenses,FALSE
au_62120,l10n_au.l10n_au_chart_template,62120,Other Business Property,account.data_account_type_expenses,FALSE
au_62200,l10n_au.l10n_au_chart_template,62200,Legal & Professional Services,account.data_account_type_expenses,FALSE
au_62300,l10n_au.l10n_au_chart_template,62300,Office Expenses,account.data_account_type_expenses,FALSE
au_62410,l10n_au.l10n_au_chart_template,62410,Staff Amenities,account.data_account_type_expenses,FALSE
au_62420,l10n_au.l10n_au_chart_template,62420,Superannuation,account.data_account_type_expenses,FALSE
au_62430,l10n_au.l10n_au_chart_template,62430,Wages & Salaries,account.data_account_type_expenses,FALSE
au_62440,l10n_au.l10n_au_chart_template,62440,Workers' Compensation,account.data_account_type_expenses,FALSE
au_62450,l10n_au.l10n_au_chart_template,62450,Other Employer Expenses,account.data_account_type_expenses,FALSE
au_62500,l10n_au.l10n_au_chart_template,62500,Repairs,account.data_account_type_expenses,FALSE
au_62550,l10n_au.l10n_au_chart_template,62550,Shrinkage/Spoilage,account.data_account_type_expenses,FALSE
au_62600,l10n_au.l10n_au_chart_template,62600,Supplies,account.data_account_type_expenses,FALSE
au_62700,l10n_au.l10n_au_chart_template,62700,Taxes,account.data_account_type_expenses,FALSE
au_62800,l10n_au.l10n_au_chart_template,62800,Telephone,account.data_account_type_expenses,FALSE
au_62910,l10n_au.l10n_au_chart_template,62910,Gas,account.data_account_type_expenses,FALSE
au_62920,l10n_au.l10n_au_chart_template,62920,Electricity,account.data_account_type_expenses,FALSE
au_62930,l10n_au.l10n_au_chart_template,62930,Water,account.data_account_type_expenses,FALSE
au_63110,l10n_au.l10n_au_chart_template,63110,Travel,account.data_account_type_expenses,FALSE
au_63120,l10n_au.l10n_au_chart_template,63120,Meals & Entertainment,account.data_account_type_expenses,FALSE
au_81000,l10n_au.l10n_au_chart_template,81000,Interest Income,account.data_account_type_other_income,FALSE
au_91000,l10n_au.l10n_au_chart_template,91000,Interest Expense,account.data_account_type_expenses,FALSE
au_92000,l10n_au.l10n_au_chart_template,92000,Income Tax Expense,account.data_account_type_expenses,FALSE

```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_gst_0,GST 0%
tax_group_gst_10,GST 10%
tax_group_gst_100000000,GST 100000000%

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_au.l10n_au_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_au_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="au_11200"/>
        <field name="property_account_payable_id" ref="au_21200"/>
        <field name="property_account_expense_categ_id" ref="au_51110"/>
        <field name="property_account_income_categ_id" ref="au_41110"/>
        <field name="property_stock_account_input_categ_id" ref="au_21210"/>
        <field name="property_stock_account_output_categ_id" ref="au_51110"/>
        <field name="property_stock_valuation_account_id" ref="au_11330"/>
        <field name="expense_currency_exchange_account_id" ref="au_61630"/>
        <field name="income_currency_exchange_account_id" ref="au_61630"/>
        <field name="default_pos_receivable_account_id" ref="au_11201" />
    </record>
</odoo>

```

## File: data\account_fiscal_position_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="fiscal_position_os_partner" model="account.fiscal.position.template">
            <field name="name">OS Partner</field>
            <field name="chart_template_id" ref="l10n_au_chart_template"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_sale" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="au_tax_sale_10"/>
            <field name="tax_dest_id" ref="au_tax_sale_0"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_sale2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="au_tax_sale_inc_10"/>
            <field name="tax_dest_id" ref="au_tax_sale_0"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_purch1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="au_tax_purchase_10"/>
            <field name="tax_dest_id" ref="au_tax_purchase_0"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_purch3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="au_tax_purchase_inc_10"/>
            <field name="tax_dest_id" ref="au_tax_purchase_0"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_purch2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="au_tax_purchase_capital"/>
            <field name="tax_dest_id" ref="au_tax_purchase_0"/>
        </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_tax_report_gstrpt_sale_total" model="account.tax.report.line">
        <field name="name">GST amounts you owe the Tax Office from sales</field>
        <field name="sequence" eval="1"/>
        <field name="formula">None</field>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g1" model="account.tax.report.line">
        <field name="name">G1: Total Sales (including any GST)</field>
        <field name="tag_name">G1</field>
        <field name="code">G1</field>
        <field name="sequence" eval="2"/>
        <field name="country_id" ref="base.au"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_sale_total"/>
    </record>

    <record id="account_tax_report_gstrpt_g2" model="account.tax.report.line">
        <field name="name">G2: Export sales</field>
        <field name="tag_name">G2</field>
        <field name="code">G2</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g1"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g3" model="account.tax.report.line">
        <field name="name">G3: Other GST-free sales</field>
        <field name="tag_name">G3</field>
        <field name="code">G3</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g1"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g4" model="account.tax.report.line">
        <field name="name">G4: Input taxed sales</field>
        <field name="tag_name">G4</field>
        <field name="code">G4</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g1"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g5" model="account.tax.report.line">
        <field name="name">G5: G2 + G3 + G4</field>
        <field name="formula">G2+G3+G4</field>
        <field name="sequence" eval="6"/>
        <field name="country_id" ref="base.au"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_sale_total"/>
    </record>

    <record id="account_tax_report_gstrpt_g6" model="account.tax.report.line">
        <field name="name">G6: Total sales subject to GST (G1 minus G5)</field>
        <field name="sequence" eval="7"/>
        <field name="formula">G1-(G2+G3+G4)</field>
        <field name="parent_id" ref="account_tax_report_gstrpt_sale_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g7" model="account.tax.report.line">
        <field name="name">G7: Adjustments (if applicable)</field>
        <field name="tag_name">G7</field>
        <field name="code">G7</field>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g6"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g8" model="account.tax.report.line">
        <field name="name">G8: Total sales subject to GST after adjustments (G6 + G7)</field>
        <field name="sequence" eval="9"/>
        <field name="formula">(G1-(G2+G3+G4))+G7</field>
        <field name="parent_id" ref="account_tax_report_gstrpt_sale_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g9" model="account.tax.report.line">
        <field name="name">G9: GST on sales (G8 divided by eleven)</field>
        <field name="code">G9</field>
        <field name="sequence" eval="10"/>
        <field name="formula">((G1-(G2+G3+G4))+G7)/11</field>
        <field name="parent_id" ref="account_tax_report_gstrpt_sale_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <!-- Purchase Report -->
    <record id="account_tax_report_gstrpt_purchase_total" model="account.tax.report.line">
        <field name="name">GST amounts the Tax Office owes you from purchases</field>
        <field name="sequence" eval="101"/>
        <field name="formula">None</field>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g10" model="account.tax.report.line">
        <field name="name">G10: Capital purchases (including any GST)</field>
        <field name="tag_name">G10</field>
        <field name="code">G10</field>
        <field name="sequence" eval="102"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g11" model="account.tax.report.line">
        <field name="name">G11: Non-capital purchases (including GST)</field>
        <field name="tag_name">G11</field>
        <field name="code">G11</field>
        <field name="sequence" eval="103"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g12" model="account.tax.report.line">
        <field name="name">G12: G10 + G11</field>
        <field name="sequence" eval="104"/>
        <field name="formula">G10+G11</field>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g13" model="account.tax.report.line">
        <field name="name">G13: Purchases for making input taxed sales</field>
        <field name="tag_name">G13</field>
        <field name="code">G13</field>
        <field name="sequence" eval="105"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g12"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g14" model="account.tax.report.line">
        <field name="name">G14: Purchases without GST in the price</field>
        <field name="tag_name">G14</field>
        <field name="code">G14</field>
        <field name="sequence" eval="106"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g12"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g15" model="account.tax.report.line">
        <field name="name">G15: Estimated purchases for private use or not income tax deductible</field>
        <field name="tag_name">G15</field>
        <field name="code">G15</field>
        <field name="sequence" eval="107"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g12"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g16" model="account.tax.report.line">
        <field name="name">G16: G13 + G14 + G15</field>
        <field name="sequence" eval="108"/>
        <field name="formula">G13+G14+G15</field>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g17" model="account.tax.report.line">
        <field name="name">G17: Total purchases subject to GST (G12 minus G16) </field>
        <field name="sequence" eval="109"/>
        <field name="formula">(G10+G11)-(G13+G14+G15)</field>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g18" model="account.tax.report.line">
        <field name="name">G18: Adjustments (if applicable)</field>
        <field name="tag_name">G18</field>
        <field name="code">G18</field>
        <field name="sequence" eval="110"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_g17"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g19" model="account.tax.report.line">
        <field name="name">G19: Total purchases subject to GST after adjustments (G17 + G18) </field>
        <field name="sequence" eval="111"/>
        <field name="formula">(G10+G11)-(G13+G14+G15)+G18</field>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g20a" model="account.tax.report.line">
        <field name="name">GST on purchases (G19 divided by eleven)</field>
        <field name="formula">((G10+G11)-(G13+G14+G15)+G18)/11</field>
        <field name="sequence" eval="112"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_gstonly" model="account.tax.report.line">
        <field name="name">GST only purchases</field>
        <field name="tag_name">ONLY</field>
        <field name="code">ONLY</field>
        <field name="sequence" eval="113"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_g20b" model="account.tax.report.line">
        <field name="name">G20: GST on purchases</field>
        <field name="formula">(((G10+G11)-(G13+G14+G15)+G18)/11)+ONLY</field>
        <field name="code">G20</field>
        <field name="sequence" eval="114"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_purchase_total"/>
        <field name="country_id" ref="base.au"/>
    </record>


    <!-- Summary -->
    <record id="account_tax_report_gstrpt_summary" model="account.tax.report.line">
        <field name="name">Summary</field>
        <field name="sequence" eval="201"/>
        <field name="formula">None</field>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_summary_1a" model="account.tax.report.line">
        <field name="name">1A: GST on sales</field>
        <field name="code">1A</field>
        <field name="sequence" eval="201"/>
        <field name="formula">((G1-(G2+G3+G4))+G7)/11</field>
        <field name="parent_id" eval="account_tax_report_gstrpt_summary"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_summary_1b" model="account.tax.report.line">
        <field name="name">1B: GST on purchases</field>
        <field name="code">1B</field>
        <field name="sequence" eval="202"/>
        <field name="formula">(((G10+G11)-(G13+G14+G15)+G18)/11)+ONLY</field>
        <field name="parent_id" eval="account_tax_report_gstrpt_summary"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_summary_9" model="account.tax.report.line">
        <field name="name">9: Your payment</field>
        <field name="sequence" eval="203"/>
        <field name="formula">(((G1-(G2+G3+G4))+G7)/11)-((((G10+G11)-(G13+G14+G15)+G18)/11)+ONLY)</field>
        <field name="parent_id" eval="account_tax_report_gstrpt_summary"/>
        <field name="country_id" ref="base.au"/>
    </record>


    <!-- Comparison -->
    <record id="account_tax_report_gstrpt_comparison" model="account.tax.report.line">
        <field name="name">Comparison</field>
        <field name="tag_name" eval="False"/>
        <field name="sequence" eval="301"/>
        <field name="parent_id" eval="False"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_comparison" model="account.tax.report.line">
        <field name="formula">None</field>
    </record>

    <record id="account_tax_report_gstrpt_comparison_worksheet" model="account.tax.report.line">
        <field name="name">GST from worksheet (G20-G9)</field>
        <field name="formula">((((G10+G11)-(G13+G14+G15)+G18)/11)+ONLY)-(((G1-(G2+G3+G4))+G7)/11)</field>
        <field name="sequence" eval="202"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_comparison"/>
        <field name="country_id" ref="base.au"/>
    </record>

    <record id="account_tax_report_gstrpt_comparison_gl" model="account.tax.report.line">
        <field name="name">GST from General Ledger</field>
        <field name="tag_name">GST from General Ledger</field>
        <field name="sequence" eval="203"/>
        <field name="parent_id" ref="account_tax_report_gstrpt_comparison"/>
        <field name="country_id" ref="base.au"/>
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

     <record id="au_tax_sale_10" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Sale (10%)</field>
        <field name="sequence">1</field>
        <field name="description">GST Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21310'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21310'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>
    <record id="au_tax_sale_inc_10" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">GST Inc Sale (10%)</field>
        <field name="sequence">2</field>
        <field name="description">GST Inclusive Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21310'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21310'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g1')],
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>
    <record id="au_tax_sale_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Zero (Export) Sale</field>
        <field name="sequence">3</field>
        <field name="description">Zero Rated (Export) Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g1'), ref('account_tax_report_gstrpt_g2')],
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
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g1'), ref('account_tax_report_gstrpt_g2')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record id="au_tax_sale_exempt" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Exempt Sale</field>
        <field name="sequence">4</field>
        <field name="description">Exempt Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g1'), ref('account_tax_report_gstrpt_g3')],
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
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g1'), ref('account_tax_report_gstrpt_g3')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record id="au_tax_sale_input" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Input Taxed</field>
        <field name="sequence">5</field>
        <field name="description">Input Taxed Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g1'), ref('account_tax_report_gstrpt_g4')],
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
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g1'), ref('account_tax_report_gstrpt_g4')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record id="au_tax_sale_adj" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Sale Adj (10%)</field>
        <field name="sequence">6</field>
        <field name="description">Tax Adjustments (Sales)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g7')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21310'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g7')],
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g7')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21310'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g7')],
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_10" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Purch (10%)</field>
        <field name="sequence">1</field>
        <field name="description">GST Purchases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_inc_10" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">GST Inc Purch (10%)</field>
        <field name="sequence">2</field>
        <field name="description">GST Inclusive Purchases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_capital" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Capital (10.0%)</field>
        <field name="sequence">3</field>
        <field name="description">Capital Purchases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g10')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g10'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g10')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g10'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Zero Rated Purch</field>
        <field name="sequence">4</field>
        <field name="description">Capital Purchases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g14')],
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
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g14')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_taxable_import" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Purch (Taxable Imports)</field>
        <field name="sequence">5</field>
        <field name="description">Purchase (Taxable Imports) - Tax Paid Separately</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g14')],
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
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g14')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_input" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Purch for Input Sales</field>
        <field name="sequence">6</field>
        <field name="description">Purchases for Input Taxed Sales</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g13')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g13')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g13')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g13')],
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_private" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Purch Private (10%)</field>
        <field name="sequence">7</field>
        <field name="description">Purchases for Private use or not deductible</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g15')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g15')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g15')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g11'), ref('account_tax_report_gstrpt_g15')],
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_gst_only" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">GST Only on Imports</field>
        <field name="sequence">8</field>
        <field name="description">GST Only on Imports</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">100000000000</field>
        <!--
          The tax percentage is so high because on imported goods we
          needed to link the tax line acknowledgment (not to be paid)
          on the customer invoice and what need to actually be
          paid from another invoice given by a clearance house
          (i.e. customs)
          For more info see the complete discussion below
          https://github.com/odoo/odoo/pull/48700#issuecomment-607586417
        -->
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_gst_100000000"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_gstonly'), ref('account_tax_report_gstrpt_comparison_gl')],
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
                    'account_id': ref('au_21330'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_gstonly'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>
    <record id="au_tax_purchase_adj" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_au_chart_template"/>
        <field name="name">Purch Adj (10%)</field>
        <field name="sequence">9</field>
        <field name="description">Tax Adjustments (Purchases)</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_gst_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'plus_report_line_ids': [ref('account_tax_report_gstrpt_g18'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('au_21330'),
                    'minus_report_line_ids': [ref('account_tax_report_gstrpt_g18'), ref('account_tax_report_gstrpt_comparison_gl')],
                }),
            ]"/>
    </record>

 </odoo>

```

## File: data\l10n_au_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_au_chart_template" model="account.chart.template">
        <field name="name">Australian Tax and Account Chart Template (by Willow IT)</field>
        <field name="bank_account_code_prefix">1111</field>
        <field name="cash_account_code_prefix">1113</field>
        <field name="transfer_account_code_prefix">11170</field>
        <field name="code_digits">5</field>
        <field name="currency_id" ref="base.AUD"/>
    </record>
</odoo>

```

## File: data\res_currency_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
	<data noupdate="1">
		<record id="base.AUD" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<!-- Common trading partner currencies, and especially for NZ, subsidiaries and branches -->

		<record id="base.NZD" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<record id="base.JPY" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<record id="base.INR" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<record id="base.GBP" model="res.currency">
			<field name="active" eval="True"/>
		</record>
	</data>
</odoo>

```

