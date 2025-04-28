# Odoo Module: l10n_sa

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_sa.account_arabic_coa_general').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Saudi Arabia - Accounting',
    'version': '1.1',
    'author': 'DVIT.ME',
    'category': 'Localization',
    'description': """
Odoo Arabic localization for most arabic countries and Saudi Arabia.

This initially includes chart of accounts of USA translated to Arabic.

In future this module will include some payroll rules for ME .
""",
    'website': 'http://www.dvit.me',
    'depends': ['account', 'l10n_multilang'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/l10n_sa_chart_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'post_init_hook': 'load_translations',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,user_type_id:id,chart_template_id:id,reconcile
account_receivable,12001,Account Receivable,account.data_account_type_receivable,account_arabic_coa_general,True
account_receivable_pos,12002,Account Receivable (PoS),account.data_account_type_receivable,account_arabic_coa_general,True
15000_furniture_equipments,15000,Furniture and Equipment,account.data_account_type_fixed_assets,account_arabic_coa_general,False
17000_accu_deprec,17000,Accumulated Depreciation,account.data_account_type_fixed_assets,account_arabic_coa_general,False
account_payable,20001,Account Payable,account.data_account_type_payable,account_arabic_coa_general,True
24000_payroll_exp,24000,Payroll Liabilities,account.data_account_type_current_liabilities,account_arabic_coa_general,False
25500_sales_tax_pay,25500,Sales Tax Payable,account.data_account_type_current_liabilities,account_arabic_coa_general,False
26500_use_tax_pay,26500,Use Tax Payable,account.data_account_type_current_liabilities,account_arabic_coa_general,False
30100_capital_stock,30100,Capital Stock,account.data_account_type_equity,account_arabic_coa_general,False
30200_divid_paid,30200,Dividends Paid,account.data_account_type_equity,account_arabic_coa_general,False
30000_open_balance,30000,Opening Balance Equity,account.data_account_type_equity,account_arabic_coa_general,False
32000_retain_earn,32000,Retained Earnings,account.data_account_type_equity,account_arabic_coa_general,False
60000_advert_promo,60000,Advertising and Promotion,account.data_account_type_expenses,account_arabic_coa_general,False
60200_auto_exp,60200,Automobile Expense,account.data_account_type_expenses,account_arabic_coa_general,False
60400_bank_charge,60400,Bank Service Charges,account.data_account_type_expenses,account_arabic_coa_general,False
61000_license_permit,61000,Business Licenses and Permits,account.data_account_type_expenses,account_arabic_coa_general,False
62000_financial_services,62000,Continuing Education,account.data_account_type_expenses,account_arabic_coa_general,False
61400_charit_contrib,61400,Charitable Contributions,account.data_account_type_expenses,account_arabic_coa_general,False
61700_computer_internet,61700,Computer and Internet Expenses,account.data_account_type_expenses,account_arabic_coa_general,False
62400_depreciation_exp,62400,Depreciation Expense,account.data_account_type_expenses,account_arabic_coa_general,False
62500_dues_subscrip,62500,Dues and Subscriptions,account.data_account_type_expenses,account_arabic_coa_general,False
62600_equip_rent,62600,Equipment Rental,account.data_account_type_expenses,account_arabic_coa_general,False
63400_interest_exp,63400,Interest Expense,account.data_account_type_expenses,account_arabic_coa_general,False
64300_meals_entertain,64300,Meals and Entertainment,account.data_account_type_expenses,account_arabic_coa_general,False
64900_office_supp,64900,Office Supplies,account.data_account_type_expenses,account_arabic_coa_general,False
66000_payroll_exp,66000,Payroll Expenses,account.data_account_type_expenses,account_arabic_coa_general,False
66500_postage_deliver,66500,Postage and Delivery,account.data_account_type_expenses,account_arabic_coa_general,False
66660_printing_reporduction,66600,Printing and Reproduction,account.data_account_type_expenses,account_arabic_coa_general,False
66700_prof_fees,66700,Professional Fees,account.data_account_type_expenses,account_arabic_coa_general,False
67100_rent_exp,67100,Rent Expense,account.data_account_type_expenses,account_arabic_coa_general,False
67200_repair_mainten,67200,Repairs and Maintenance,account.data_account_type_expenses,account_arabic_coa_general,False
68000_tax_property,68000,Taxes - Property,account.data_account_type_expenses,account_arabic_coa_general,False
68100_telephone_exp,68100,Telephone Expense,account.data_account_type_expenses,account_arabic_coa_general,False
68400_transport_travel,68400,Travel Expense,account.data_account_type_expenses,account_arabic_coa_general,False
68600_utilities,68600,Utilities,account.data_account_type_expenses,account_arabic_coa_general,False
64200_marketing_exp,64200,Marketing Expense,account.data_account_type_expenses,account_arabic_coa_general,False
63200_gas_oil_exp,63200,"Gasoline, Fuel and Oil",account.data_account_type_expenses,account_arabic_coa_general,False
base_miscexpense,69000,Miscellaneous Expense,account.data_account_type_expenses,account_arabic_coa_general,False
69310_general_liab_insurance,69310,General Liability Insurance,account.data_account_type_expenses,account_arabic_coa_general,False
69320_health_insurance,69320,Health Insurance,account.data_account_type_expenses,account_arabic_coa_general,False
69330_life_insurance,69330,Life and Disability Insurance,account.data_account_type_expenses,account_arabic_coa_general,False
69350_prof_insurance,69350,Professional Liability,account.data_account_type_expenses,account_arabic_coa_general,False
69360_worker_compons,69360,Worker's Compensation,account.data_account_type_expenses,account_arabic_coa_general,False
42400_commission_income,42400,Commission income,account.data_account_type_revenue,account_arabic_coa_general,False
47400_rent_income,47400,Rent Income,account.data_account_type_revenue,account_arabic_coa_general,False
47900_general_service_sales,47900,Service Sales,account.data_account_type_revenue,account_arabic_coa_general,False
47910_general_product_sales,47910,Product Sales,account.data_account_type_revenue,account_arabic_coa_general,False
48310_product_sales_discount,48310,Sales Discounts,account.data_account_type_revenue,account_arabic_coa_general,False
48900_product_shipping_income,48900,Shipping and Delivery Income,account.data_account_type_revenue,account_arabic_coa_general,False
base_miscincome,49000,Miscellaneous Income,account.data_account_type_revenue,account_arabic_coa_general,False
70000_finance_income,70000,Finance Charge Income,account.data_account_type_other_income,account_arabic_coa_general,False
70200_interest_income,70200,Interest Income,account.data_account_type_other_income,account_arabic_coa_general,False
70500_assets_sold,70500,Proceeds from Sale of Assets,account.data_account_type_other_income,account_arabic_coa_general,False
70100_insurance_income,70100,Insurance Proceeds Received,account.data_account_type_other_income,account_arabic_coa_general,False
53600_other_related_costs,53600,Other Job Related Costs,account.data_account_type_direct_costs,account_arabic_coa_general,False
51100_product_shipping_cost,51100,Freight and Shipping Costs,account.data_account_type_direct_costs,account_arabic_coa_general,False
52500_product_purchase_discount,52500,Purchase Discounts,account.data_account_type_direct_costs,account_arabic_coa_general,False
52900_general_product_cost,52900,Purchases - Resale Items,account.data_account_type_direct_costs,account_arabic_coa_general,False
50300_commissions_paid,50300,Commissions Paid,account.data_account_type_direct_costs,account_arabic_coa_general,False
53500_subcontracted_services,53500,Subcontractors Expense,account.data_account_type_direct_costs,account_arabic_coa_general,False
52000_merchant_fees,52000,Merchant Account Fees,account.data_account_type_direct_costs,account_arabic_coa_general,False
53800_subcontract_exp,53800,Subcontractors Expense,account.data_account_type_direct_costs,account_arabic_coa_general,False
54100_tools_equip,54100,Tools and Small Equipment,account.data_account_type_direct_costs,account_arabic_coa_general,False
24600_customer_deposit,24600,Customer Deposits Received,account.data_account_type_current_liabilities,account_arabic_coa_general,False

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_sa.account_arabic_coa_general')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="account_arabic_coa_general" model="account.chart.template">
            <field name="name">Saudi Arabia - Chart of Accounts</field>
            <field name="bank_account_code_prefix">1</field>
            <field name="cash_account_code_prefix">1</field>
            <field name="transfer_account_code_prefix">18</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.SAR"/>
            <field name="spoken_languages" eval="'en_US;ar_EG;ar_SY'"/>
        </record>
</odoo>

```

## File: data\l10n_sa_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart Template -->
    <record id="account_arabic_coa_general" model="account.chart.template">
        <field name="property_account_receivable_id" ref="account_receivable"/>
        <field name="property_account_payable_id" ref="account_payable"/>
        <field name="property_account_expense_id" ref="52900_general_product_cost"/>
        <field name="property_account_income_id" ref="47910_general_product_sales"/>
        <field name="property_account_expense_categ_id" ref="52900_general_product_cost"/>
        <field name="property_account_income_categ_id" ref="47910_general_product_sales"/>
        <field name="income_currency_exchange_account_id" ref="base_miscincome"/>
        <field name="expense_currency_exchange_account_id" ref="base_miscexpense"/>
        <field name="default_pos_receivable_account_id" ref="account_receivable_pos" />
    </record>
</odoo>

```

