# Odoo Module: l10n_eg

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from odoo import api, SUPERUSER_ID
from . import models


def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_eg.egypt_chart_template_standard').process_coa_translations()

```

## File: __manifest__.py

```python
{
    'name': "Egypt - Accounting",
    'description': """
This is the base module to manage the accounting chart for Egypt in Odoo.
==============================================================================
    """,
    'author': "Odoo SA",
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.0',
    'depends': ['account','l10n_multilang'],
    'data': [
        'data/l10n_eg_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_eg_chart_post_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template_data.xml',
        'data/fiscal_templates_data.xml',
        'data/account_chart_template_data.xml',
        'views/account_tax.xml'
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/demo_partner.xml'
    ],
    'post_init_hook': 'load_translations',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,name,chart_template_id/id,code,user_type_id/id,reconcile
egy_account_100101,Right of use Asset (IFRS 16),l10n_eg.egypt_chart_template_standard,100101,account.data_account_type_fixed_assets,False
egy_account_100102,Accumulated Depreciation right use asset (IFRS 16),l10n_eg.egypt_chart_template_standard,100102,account.data_account_type_fixed_assets,False
egy_account_100103,VAT Receivable,l10n_eg.egypt_chart_template_standard,100103,account.data_account_type_non_current_assets,False
egy_account_101004,Outstanding Receipts,l10n_eg.egypt_chart_template_standard,101004,account.data_account_type_current_assets,False
egy_account_101005,Main Safe,l10n_eg.egypt_chart_template_standard,101005,account.data_account_type_current_assets,False
egy_account_101006,Main Safe - Foreign Currency,l10n_eg.egypt_chart_template_standard,101006,account.data_account_type_current_assets,False
egy_account_101007,Visa & Master Credit Cards,l10n_eg.egypt_chart_template_standard,101007,account.data_account_type_current_assets,False
egy_account_101008,Gateway Credit Cards,l10n_eg.egypt_chart_template_standard,101008,account.data_account_type_current_assets,False
egy_account_101009,Manual Visa & Master Cards,l10n_eg.egypt_chart_template_standard,101009,account.data_account_type_current_assets,False
egy_account_101010,PayPal Account,l10n_eg.egypt_chart_template_standard,101010,account.data_account_type_current_assets,False
egy_account_102011,Accounts Receivable,l10n_eg.egypt_chart_template_standard,102011,account.data_account_type_receivable,True
egy_account_102012,Accounts Receivable (PoS),l10n_eg.egypt_chart_template_standard,102012,account.data_account_type_receivable,True
egy_account_102013,Post Dated Cheques Received,l10n_eg.egypt_chart_template_standard,102013,account.data_account_type_current_assets,False
egy_account_102014,Other Receivable,l10n_eg.egypt_chart_template_standard,102014,account.data_account_type_current_assets,False
egy_account_102015,Other Debtors,l10n_eg.egypt_chart_template_standard,102015,account.data_account_type_current_assets,False
egy_account_103016,Shipment Insurance,l10n_eg.egypt_chart_template_standard,103016,account.data_account_type_current_assets,False
egy_account_103017,Shipments Documentation Charges,l10n_eg.egypt_chart_template_standard,103017,account.data_account_type_current_assets,False
egy_account_103018,Shipment Other Charges,l10n_eg.egypt_chart_template_standard,103018,account.data_account_type_current_assets,False
egy_account_103019,Handling Difference in Inventory,l10n_eg.egypt_chart_template_standard,103019,account.data_account_type_current_assets,False
egy_account_103020,Items Delivered to Customs on temprary Base,l10n_eg.egypt_chart_template_standard,103020,account.data_account_type_current_assets,False
egy_account_104021,Prepaid Medical Insurance,l10n_eg.egypt_chart_template_standard,104021,account.data_account_type_current_assets,False
egy_account_104022,Prepaid Life Insurance,l10n_eg.egypt_chart_template_standard,104022,account.data_account_type_current_assets,False
egy_account_104023,Prepaid Office Rent,l10n_eg.egypt_chart_template_standard,104023,account.data_account_type_current_assets,False
egy_account_104024,Prepaid Other Insurance,l10n_eg.egypt_chart_template_standard,104024,account.data_account_type_current_assets,False
egy_account_104025,Prepaid License Fees,l10n_eg.egypt_chart_template_standard,104025,account.data_account_type_current_assets,False
egy_account_104026,Prepaid Maintenance,l10n_eg.egypt_chart_template_standard,104026,account.data_account_type_current_assets,False
egy_account_104027,Prepaid Site Hosting Fees,l10n_eg.egypt_chart_template_standard,104027,account.data_account_type_current_assets,False
egy_account_104028,Prepaid Employees Housing,l10n_eg.egypt_chart_template_standard,104028,account.data_account_type_current_assets,False
egy_account_104029,Prepaid Schooling Fees,l10n_eg.egypt_chart_template_standard,104029,account.data_account_type_current_assets,False
egy_account_104030,Prepaid Consultancy Fees,l10n_eg.egypt_chart_template_standard,104030,account.data_account_type_current_assets,False
egy_account_104031,Prepaid Legal Fees,l10n_eg.egypt_chart_template_standard,104031,account.data_account_type_current_assets,False
egy_account_104033,PrePaid Advertisement Expenses,l10n_eg.egypt_chart_template_standard,104033,account.data_account_type_current_assets,False
egy_account_104034,Prepaid Bank Guarantee,l10n_eg.egypt_chart_template_standard,104034,account.data_account_type_current_assets,False
egy_account_104035,Other Prepayments,l10n_eg.egypt_chart_template_standard,104035,account.data_account_type_current_assets,False
egy_account_104036,Prepaid Finance charge for Loans,l10n_eg.egypt_chart_template_standard,104036,account.data_account_type_current_assets,False
egy_account_104037,Deposit - Office Rent,l10n_eg.egypt_chart_template_standard,104037,account.data_account_type_current_assets,False
egy_account_104038,Deposits - Customs,l10n_eg.egypt_chart_template_standard,104038,account.data_account_type_current_assets,False
egy_account_104040,Deposit Others,l10n_eg.egypt_chart_template_standard,104040,account.data_account_type_current_assets,False
egy_account_104041,VAT Input,l10n_eg.egypt_chart_template_standard,104041,account.data_account_type_current_assets,False
egy_account_104042,WH tax Advance with Customers - On behalf of my company,l10n_eg.egypt_chart_template_standard,104042,account.data_account_type_current_assets,False
egy_account_105003,Outstanding Payments,l10n_eg.egypt_chart_template_standard,105003,account.data_account_type_current_assets,False
egy_account_106001,Leasehold Improvement,l10n_eg.egypt_chart_template_standard,106001,account.data_account_type_current_assets,False
egy_account_106002,Furniture and Equipment,l10n_eg.egypt_chart_template_standard,106002,account.data_account_type_fixed_assets,False
egy_account_106003,Computer Hardware & Software,l10n_eg.egypt_chart_template_standard,106003,account.data_account_type_fixed_assets,False
egy_account_106004,Motor Vehicles,l10n_eg.egypt_chart_template_standard,106004,account.data_account_type_fixed_assets,False
egy_account_106006,Amortisation on Leasehold Improvement,l10n_eg.egypt_chart_template_standard,106006,account.data_account_type_current_assets,False
egy_account_106007,Acc.Deprn.of Furniture & Office Equipment,l10n_eg.egypt_chart_template_standard,106007,account.data_account_type_current_assets,False
egy_account_106008,Acc. Deprn.Computer Hardware & Software,l10n_eg.egypt_chart_template_standard,106008,account.data_account_type_current_assets,False
egy_account_106009,Acc. Depreciation of Motor Vehicles,l10n_eg.egypt_chart_template_standard,106009,account.data_account_type_current_assets,False
egy_account_106010,Registration of Trademarks,l10n_eg.egypt_chart_template_standard,106010,account.data_account_type_current_assets,False
egy_account_106011,Computer Card Renewal,l10n_eg.egypt_chart_template_standard,106011,account.data_account_type_current_assets,False
egy_account_201001,Bank Suspense Account,l10n_eg.egypt_chart_template_standard,201001,account.data_account_type_current_liabilities,False
egy_account_201002,Payables,l10n_eg.egypt_chart_template_standard,201002,account.data_account_type_payable,True
egy_account_201003,Credit Notes to Customers,l10n_eg.egypt_chart_template_standard,201003,account.data_account_type_current_liabilities,False
egy_account_201004,Accrued - Salaries,l10n_eg.egypt_chart_template_standard,201004,account.data_account_type_current_liabilities,False
egy_account_201005,Leave Tickets Provision,l10n_eg.egypt_chart_template_standard,201005,account.data_account_type_current_liabilities,False
egy_account_201006,Leave Days Provision,l10n_eg.egypt_chart_template_standard,201006,account.data_account_type_current_liabilities,False
egy_account_201007,Accrued - Commissions,l10n_eg.egypt_chart_template_standard,201007,account.data_account_type_current_liabilities,False
egy_account_201008,Accrued Salaries Increment,l10n_eg.egypt_chart_template_standard,201008,account.data_account_type_current_liabilities,False
egy_account_201009,Accrued-Staff Bonus,l10n_eg.egypt_chart_template_standard,201009,account.data_account_type_current_liabilities,False
egy_account_201010,Accrued Other Personnel Cost,l10n_eg.egypt_chart_template_standard,201010,account.data_account_type_current_liabilities,False
egy_account_201011,Accrued - Utilities,l10n_eg.egypt_chart_template_standard,201011,account.data_account_type_current_liabilities,False
egy_account_201012,Accrued - Telephone,l10n_eg.egypt_chart_template_standard,201012,account.data_account_type_current_liabilities,False
egy_account_201013,Accrued - Sponsorship,l10n_eg.egypt_chart_template_standard,201013,account.data_account_type_current_liabilities,False
egy_account_201014,Accrued - Audit Fees,l10n_eg.egypt_chart_template_standard,201014,account.data_account_type_current_liabilities,False
egy_account_201015,Accrued - Office Rent,l10n_eg.egypt_chart_template_standard,201015,account.data_account_type_current_liabilities,False
egy_account_201016,Accrued Others,l10n_eg.egypt_chart_template_standard,201016,account.data_account_type_current_liabilities,False
egy_account_201017,VAT Output,l10n_eg.egypt_chart_template_standard,201017,account.data_account_type_current_liabilities,False
egy_account_201018,Deferred income,l10n_eg.egypt_chart_template_standard,201018,account.data_account_type_current_liabilities,False
egy_account_201020,WHTax Payable - On behalf of suppliers,l10n_eg.egypt_chart_template_standard,201020,account.data_account_type_current_liabilities,False
egy_account_201021,Legal Reserve,l10n_eg.egypt_chart_template_standard,201021,account.data_account_type_current_liabilities,False
egy_account_201022,Taxes Provision,l10n_eg.egypt_chart_template_standard,201022,account.data_account_type_current_liabilities,False
egy_account_201023,Customer Provision,l10n_eg.egypt_chart_template_standard,201023,account.data_account_type_current_liabilities,False
egy_account_201024,Schedule Tax collected & payable,l10n_eg.egypt_chart_template_standard,201024,account.data_account_type_current_liabilities,False
egy_account_201025,Stamp Tax payable,l10n_eg.egypt_chart_template_standard,201025,account.data_account_type_current_liabilities,False
egy_account_201026,Social Contribution - Payable to authorities,l10n_eg.egypt_chart_template_standard,201026,account.data_account_type_current_liabilities,False
egy_account_201027,Income Tax payable to Authority - Deducted from employee's salaries,l10n_eg.egypt_chart_template_standard,201027,account.data_account_type_current_liabilities,False
egy_account_202001,End of Service Provision,l10n_eg.egypt_chart_template_standard,202001,account.data_account_type_non_current_liabilities,False
egy_account_202002,Reservations,l10n_eg.egypt_chart_template_standard,202002,account.data_account_type_non_current_liabilities,False
egy_account_202003,VAT Payable,l10n_eg.egypt_chart_template_standard,202003,account.data_account_type_non_current_liabilities,False
egy_account_400001,Cost of Goods Sold in Trading,l10n_eg.egypt_chart_template_standard,400001,account.data_account_type_direct_costs,False
egy_account_400002,Cost Of Goods Sold I/C Sales,l10n_eg.egypt_chart_template_standard,400002,account.data_account_type_direct_costs,False
egy_account_400003,Basic Salary,l10n_eg.egypt_chart_template_standard,400003,account.data_account_type_expenses,False
egy_account_400004,Housing Allowance,l10n_eg.egypt_chart_template_standard,400004,account.data_account_type_expenses,False
egy_account_400005,Transportation Allowance,l10n_eg.egypt_chart_template_standard,400005,account.data_account_type_expenses,False
egy_account_400006,Leave Ticket,l10n_eg.egypt_chart_template_standard,400006,account.data_account_type_expenses,False
egy_account_400007,Leave Salary,l10n_eg.egypt_chart_template_standard,400007,account.data_account_type_expenses,False
egy_account_400008,End Of Service Indemnity,l10n_eg.egypt_chart_template_standard,400008,account.data_account_type_expenses,False
egy_account_400009,Medical Insurance,l10n_eg.egypt_chart_template_standard,400009,account.data_account_type_expenses,False
egy_account_400010,Life Insurance,l10n_eg.egypt_chart_template_standard,400010,account.data_account_type_expenses,False
egy_account_400011,Sales Commission,l10n_eg.egypt_chart_template_standard,400011,account.data_account_type_expenses,False
egy_account_400012,Staff Other Allowances,l10n_eg.egypt_chart_template_standard,400012,account.data_account_type_expenses,False
egy_account_400013,Uniform,l10n_eg.egypt_chart_template_standard,400013,account.data_account_type_expenses,False
egy_account_400014,Visa Expenses,l10n_eg.egypt_chart_template_standard,400014,account.data_account_type_expenses,False
egy_account_400015,Personnel Cost Others,l10n_eg.egypt_chart_template_standard,400015,account.data_account_type_expenses,False
egy_account_400016,Office Rent,l10n_eg.egypt_chart_template_standard,400016,account.data_account_type_expenses,False
egy_account_400017,Warehouse Rent,l10n_eg.egypt_chart_template_standard,400017,account.data_account_type_expenses,False
egy_account_400018,Water & Electricity,l10n_eg.egypt_chart_template_standard,400018,account.data_account_type_expenses,False
egy_account_400019,Other Utility Cahrges,l10n_eg.egypt_chart_template_standard,400019,account.data_account_type_expenses,False
egy_account_400020,Telephone,l10n_eg.egypt_chart_template_standard,400020,account.data_account_type_expenses,False
egy_account_400021,Courrier,l10n_eg.egypt_chart_template_standard,400021,account.data_account_type_expenses,False
egy_account_400022,Web Site Hosting Fees,l10n_eg.egypt_chart_template_standard,400022,account.data_account_type_expenses,False
egy_account_400023,Others - Communication,l10n_eg.egypt_chart_template_standard,400023,account.data_account_type_expenses,False
egy_account_400024,Air tickets,l10n_eg.egypt_chart_template_standard,400024,account.data_account_type_expenses,False
egy_account_400025,Hotel,l10n_eg.egypt_chart_template_standard,400025,account.data_account_type_expenses,False
egy_account_400026,Meals,l10n_eg.egypt_chart_template_standard,400026,account.data_account_type_expenses,False
egy_account_400027,Per Diem,l10n_eg.egypt_chart_template_standard,400027,account.data_account_type_expenses,False
egy_account_400028,Others,l10n_eg.egypt_chart_template_standard,400028,account.data_account_type_expenses,False
egy_account_400029,Audit Fees,l10n_eg.egypt_chart_template_standard,400029,account.data_account_type_expenses,False
egy_account_400031,Legal fees,l10n_eg.egypt_chart_template_standard,400031,account.data_account_type_expenses,False
egy_account_400032,Trade License Fees,l10n_eg.egypt_chart_template_standard,400032,account.data_account_type_expenses,False
egy_account_400033,Others - Professional Fees,l10n_eg.egypt_chart_template_standard,400033,account.data_account_type_expenses,False
egy_account_400034,Other - Advertising Expenses,l10n_eg.egypt_chart_template_standard,400034,account.data_account_type_expenses,False
egy_account_400035,Write Off Receivables & Payables,l10n_eg.egypt_chart_template_standard,400035,account.data_account_type_expenses,False
egy_account_400036,Write Off Inventory,l10n_eg.egypt_chart_template_standard,400036,account.data_account_type_expenses,False
egy_account_400037,Amortisation of Preoperating Expenses,l10n_eg.egypt_chart_template_standard,400037,account.data_account_type_expenses,False
egy_account_400038,Cash Shortage,l10n_eg.egypt_chart_template_standard,400038,account.data_account_type_expenses,False
egy_account_400039,Others - Provision & Write off,l10n_eg.egypt_chart_template_standard,400039,account.data_account_type_expenses,False
egy_account_400040,Insurance,l10n_eg.egypt_chart_template_standard,400040,account.data_account_type_expenses,False
egy_account_400041,Training,l10n_eg.egypt_chart_template_standard,400041,account.data_account_type_expenses,False
egy_account_400042,Maintenance,l10n_eg.egypt_chart_template_standard,400042,account.data_account_type_expenses,False
egy_account_400043,Security & Guard,l10n_eg.egypt_chart_template_standard,400043,account.data_account_type_expenses,False
egy_account_400044,Cleaning,l10n_eg.egypt_chart_template_standard,400044,account.data_account_type_expenses,False
egy_account_400045,Subscriptions,l10n_eg.egypt_chart_template_standard,400045,account.data_account_type_expenses,False
egy_account_400046,Gifts & Donations,l10n_eg.egypt_chart_template_standard,400046,account.data_account_type_expenses,False
egy_account_400047,Kitchen and Buffet Expenses,l10n_eg.egypt_chart_template_standard,400047,account.data_account_type_expenses,False
egy_account_400048,Vehicle Expenses,l10n_eg.egypt_chart_template_standard,400048,account.data_account_type_expenses,False
egy_account_400049,Convoyance Expenses,l10n_eg.egypt_chart_template_standard,400049,account.data_account_type_expenses,False
egy_account_400050,Others - Office Various Expenses,l10n_eg.egypt_chart_template_standard,400050,account.data_account_type_expenses,False
egy_account_400051,Other Bank Charges,l10n_eg.egypt_chart_template_standard,400051,account.data_account_type_expenses,False
egy_account_400052,Loss On Fixed Assets Disposal,l10n_eg.egypt_chart_template_standard,400052,account.data_account_type_expenses,False
egy_account_400053,Loss on Difference on Exchange,l10n_eg.egypt_chart_template_standard,400053,account.data_account_type_expenses,False
egy_account_400054,Disposal of Business Branch,l10n_eg.egypt_chart_template_standard,400054,account.data_account_type_expenses,False
egy_account_400055,Income Tax,l10n_eg.egypt_chart_template_standard,400055,account.data_account_type_expenses,False
egy_account_400056,Previous Year Adjustments Account,l10n_eg.egypt_chart_template_standard,400056,account.data_account_type_expenses,False
egy_account_400057,Other Non Operating Expenses,l10n_eg.egypt_chart_template_standard,400057,account.data_account_type_expenses,False
egy_account_400058,Credit Card Charges,l10n_eg.egypt_chart_template_standard,400058,account.data_account_type_expenses,False
egy_account_400059,Bank Finance & Loan Charges,l10n_eg.egypt_chart_template_standard,400059,account.data_account_type_expenses,False
egy_account_400060,Air Miles Card Charges,l10n_eg.egypt_chart_template_standard,400060,account.data_account_type_expenses,False
egy_account_400061,Credit Card Swipe Charges,l10n_eg.egypt_chart_template_standard,400061,account.data_account_type_expenses,False
egy_account_400062,PayPal Charges,l10n_eg.egypt_chart_template_standard,400062,account.data_account_type_expenses,False
egy_account_400063,Amortization on Leasehold Improvement,l10n_eg.egypt_chart_template_standard,400063,account.data_account_type_expenses,False
egy_account_400064,Depreciation Of Furniture & Office Equipment,l10n_eg.egypt_chart_template_standard,400064,account.data_account_type_expenses,False
egy_account_400065,Depreciation Of Computer Hard & Soft,l10n_eg.egypt_chart_template_standard,400065,account.data_account_type_expenses,False
egy_account_400066,Depreciation Of Motor Vehicles,l10n_eg.egypt_chart_template_standard,400066,account.data_account_type_expenses,False
egy_account_400067,Consultancy Fees,l10n_eg.egypt_chart_template_standard,400067,account.data_account_type_expenses,False
egy_account_400068,Provision for Doubtful Debts,l10n_eg.egypt_chart_template_standard,400068,account.data_account_type_expenses,False
egy_account_400069,Closing Account,l10n_eg.egypt_chart_template_standard,400069,account.data_account_type_expenses,False
egy_account_400070,Depreciation on right of use asset (IFRS 16),l10n_eg.egypt_chart_template_standard,400070,account.data_account_type_expenses,False
egy_account_400072,Interest Expense,l10n_eg.egypt_chart_template_standard,400072,account.data_account_type_expenses,False
egy_account_400074,Bad Debts,l10n_eg.egypt_chart_template_standard,400074,account.data_account_type_expenses,False
egy_account_400075,Schedule Tax Expense,l10n_eg.egypt_chart_template_standard,400075,account.data_account_type_expenses,False
egy_account_400076,WH Tax Expense,l10n_eg.egypt_chart_template_standard,400076,account.data_account_type_expenses,False
egy_account_400077,Stamp tax expense,l10n_eg.egypt_chart_template_standard,400077,account.data_account_type_expenses,False
egy_account_400078,Social Contibution - Company portion expense,l10n_eg.egypt_chart_template_standard,400078,account.data_account_type_expenses,False
egy_account_500001,Sales Account,l10n_eg.egypt_chart_template_standard,500001,account.data_account_type_revenue,False
egy_account_500002,Sales of I/C,l10n_eg.egypt_chart_template_standard,500002,account.data_account_type_revenue,False
egy_account_500003,Management Consultancy Fees,l10n_eg.egypt_chart_template_standard,500003,account.data_account_type_revenue,False
egy_account_500004,Sales from Other Region,l10n_eg.egypt_chart_template_standard,500004,account.data_account_type_revenue,False
egy_account_500005,Advertising Income,l10n_eg.egypt_chart_template_standard,500005,account.data_account_type_revenue,False
egy_account_500006,Branding Income,l10n_eg.egypt_chart_template_standard,500006,account.data_account_type_revenue,False
egy_account_500007,Space Rental Income,l10n_eg.egypt_chart_template_standard,500007,account.data_account_type_revenue,False
egy_account_500008,Service Income,l10n_eg.egypt_chart_template_standard,500008,account.data_account_type_revenue,False
egy_account_500009,Interest Revenue,l10n_eg.egypt_chart_template_standard,500009,account.data_account_type_revenue,False
egy_account_500010,Capital Gain,l10n_eg.egypt_chart_template_standard,500010,account.data_account_type_revenue,False
egy_account_500011,Gain On Difference Of Exchange,l10n_eg.egypt_chart_template_standard,500011,account.data_account_type_revenue,False
egy_account_500013,Other Income,l10n_eg.egypt_chart_template_standard,500013,account.data_account_type_revenue,False
egy_account_999001,Cash Difference Loss,l10n_eg.egypt_chart_template_standard,999001,account.data_account_type_expenses,False
egy_account_999002,Cash Difference Gain,l10n_eg.egypt_chart_template_standard,999002,account.data_account_type_revenue,False
egy_account_999999,Undistributed Profits/Losses,l10n_eg.egypt_chart_template_standard,999999,account.data_unaffected_earnings,False

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_eg.egypt_chart_template_standard')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Account Tax Group -->

        <record id="eg_tax_vat" model="account.tax.group">
            <field name="name">VAT 14%</field>
            <field name="country_id" ref="base.eg"/>
        </record>
        
        <record id="eg_tax_group_other" model="account.tax.group">
            <field name="name">Other Taxes</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_stamp" model="account.tax.group">
            <field name="name">Stamp Tax 20%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_withholding_half" model="account.tax.group">
            <field name="name">Withholding Tax -0.5%</field>
            <field name="preceding_subtotal">Subtotal W/O WHTax</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_withholding_1" model="account.tax.group">
            <field name="name">Withholding Tax -1%</field>
            <field name="preceding_subtotal">Subtotal W/O WHTax</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_withholding_3" model="account.tax.group">
            <field name="name">Withholding Tax -3%</field>
            <field name="preceding_subtotal">Subtotal W/O WHTax</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_withholding_5" model="account.tax.group">
            <field name="name">Withholding Tax -5%</field>
            <field name="preceding_subtotal">Subtotal W/O WHTax</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_schedule_half" model="account.tax.group">
            <field name="name">Schedule Tax 0.5%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_schedule_1" model="account.tax.group">
            <field name="name">Schedule Tax 1%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_schedule_5" model="account.tax.group">
            <field name="name">Schedule Tax 5%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_schedule_8" model="account.tax.group">
            <field name="name">Schedule Tax 8%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_schedule_10" model="account.tax.group">
            <field name="name">Schedule Tax 10%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_schedule_15" model="account.tax.group">
            <field name="name">Schedule Tax 15%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

        <record id="eg_tax_group_schedule_30" model="account.tax.group">
            <field name="name">Schedule Tax 30%</field>
            <field name="country_id" ref="base.eg"/>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_vat_return" model="account.tax.report">
        <field name="name">1. VAT Return</field>
        <field name="country_id" ref="base.eg"/>
    </record>

    <record id="tax_report_vat_return_sale_base" model="account.tax.report.line">
        <field name="name">VAT on Sales and all other Outputs (Base)</field>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_vat_return_sale_base_fourteen" model="account.tax.report.line">
        <field name="name">1. Standard Rated 14% (Base)</field>
        <field name="tag_name">1. VAT 14% (Base)</field>
        <field name="code">STD_SALE_B</field>
        <field name="parent_id" ref="tax_report_vat_return_sale_base"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_vat_return_sale_base_zero" model="account.tax.report.line">
        <field name="name">2. Zero Rated (Base)</field>
        <field name="tag_name">2. Zero Rated (Base)</field>
        <field name="code">ZERO_SALE_B</field>
        <field name="parent_id" ref="tax_report_vat_return_sale_base"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_vat_return_sale_base_exempt" model="account.tax.report.line">
        <field name="name">3. Exempt Sales (Base)</field>
        <field name="tag_name">3. Exempt Sales (Base)</field>
        <field name="code">EXM_SALE_B</field>
        <field name="parent_id" ref="tax_report_vat_return_sale_base"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_vat_return_sale_tax" model="account.tax.report.line">
        <field name="name">VAT on Sales and all other Outputs (Tax)</field>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_vat_return_sale_tax_fourteen" model="account.tax.report.line">
        <field name="name">1. Standard Rated 14% (Tax)</field>
        <field name="tag_name">1. VAT 14% (Tax)</field>
        <field name="code">STD_SALE_T</field>
        <field name="parent_id" ref="tax_report_vat_return_sale_tax"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_vat_return_sale_tax_zero" model="account.tax.report.line">
        <field name="name">2. Zero Rated (Tax)</field>
        <field name="tag_name">2. Zero Rated (Tax)</field>
        <field name="code">ZERO_SALE_T</field>
        <field name="parent_id" ref="tax_report_vat_return_sale_tax"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_vat_return_sale_tax_exempt" model="account.tax.report.line">
        <field name="name">3. Exempt Sales (Tax)</field>
        <field name="tag_name">3. Exempt Sales (Tax)</field>
        <field name="code">EXM_SALE_T</field>
        <field name="parent_id" ref="tax_report_vat_return_sale_tax"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_vat_return_expense_base" model="account.tax.report.line">
        <field name="name">VAT on Expenses and all other Inputs (Base)</field>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_vat_return_expense_base_fourteen" model="account.tax.report.line">
        <field name="name">5. Standard Rated 14% Expenses (Base)</field>
        <field name="tag_name">5. VAT 14% Expenses (Base)</field>
        <field name="code">STD_PUR_B</field>
        <field name="parent_id" ref="tax_report_vat_return_expense_base"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_vat_return_expense_base_zero" model="account.tax.report.line">
        <field name="name">6. Zero Rated (Base)</field>
        <field name="tag_name">6. Zero Rated (Base)</field>
        <field name="code">ZERO_PUR_B</field>
        <field name="parent_id" ref="tax_report_vat_return_expense_base"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_vat_return_expense_base_exempt" model="account.tax.report.line">
        <field name="name">7. Exempt Expenses (Base)</field>
        <field name="tag_name">7. Exempt Expenses (Base)</field>
        <field name="code">EXM_PUR_B</field>
        <field name="parent_id" ref="tax_report_vat_return_expense_base"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="3"/>
    </record>


    <record id="tax_report_vat_return_expense_tax" model="account.tax.report.line">
        <field name="name">VAT on Expenses and all other Inputs (Tax)</field>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_vat_return_expense_tax_fourteen" model="account.tax.report.line">
        <field name="name">5. Standard Rated 14% Expenses (Tax)</field>
        <field name="tag_name">5. VAT 14% Expenses (Tax)</field>
        <field name="code">STD_PUR_T</field>
        <field name="parent_id" ref="tax_report_vat_return_expense_tax"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_vat_return_expense_tax_zero" model="account.tax.report.line">
        <field name="name">6. Zero Rated (Tax)</field>
        <field name="tag_name">6. Zero Rated (Tax)</field>
        <field name="code">ZERO_PUR_T</field>
        <field name="parent_id" ref="tax_report_vat_return_expense_tax"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_vat_return_expense_tax_exempt" model="account.tax.report.line">
        <field name="name">7. Exempt Expenses (Tax)</field>
        <field name="tag_name">7. Exempt Expenses (Tax)</field>
        <field name="code">EXM_PUR_T</field>
        <field name="parent_id" ref="tax_report_vat_return_expense_tax"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="3"/>
    </record>


    <record id="tax_report_vat_return_net" model="account.tax.report.line">
        <field name="name">Net VAT Due</field>
        <field name="formula">None</field>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_vat_return_net_1" model="account.tax.report.line">
        <field name="name">Total value of due tax for the period</field>
        <field name="formula">STD_SALE_T + ZERO_SALE_T + EXM_SALE_T</field>
        <field name="parent_id" ref="tax_report_vat_return_net"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_vat_return_net_2" model="account.tax.report.line">
        <field name="name">Total value of recoverable tax for the period</field>
        <field name="formula">STD_PUR_T + ZERO_PUR_T + EXM_PUR_T</field>
        <field name="parent_id" ref="tax_report_vat_return_net"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_vat_return_net_3" model="account.tax.report.line">
        <field name="name">Net VAT due (or reclaimed) for the period</field>
        <field name="formula">STD_SALE_T + ZERO_SALE_T + EXM_SALE_T - (STD_PUR_T + ZERO_PUR_T + EXM_PUR_T)</field>
        <field name="parent_id" ref="tax_report_vat_return_net"/>
        <field name="report_id" ref="tax_report_vat_return"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_withholding_tax" model="account.tax.report">
        <field name="name">2. Withholding Tax</field>
        <field name="country_id" ref="base.eg"/>
    </record>

    <record id="tax_report_withholding_tax_sale_base" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales (Base)</field>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_withholding_tax_sale_base_half" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -0.5% (Base)</field>
        <field name="tag_name">WH Sales -0.5% (Base)</field>
        <field name="code">H_SALE_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_withholding_tax_sale_base_one" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -1% (Base)</field>
        <field name="tag_name">WH on Sales -1% (Base)</field>
        <field name="code">O_SALE_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_withholding_tax_sale_base_three" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -3% (Base)</field>
        <field name="tag_name">WH on Sales -3% (Base)</field>
        <field name="code">T_SALE_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_withholding_tax_sale_base_five" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -5% (Base)</field>
        <field name="tag_name">WH on Sales -5% (Base)</field>
        <field name="code">F_SALE_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_withholding_tax_sale_tax" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales (Tax)</field>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_withholding_tax_sale_tax_half" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -0.5% (Tax)</field>
        <field name="tag_name">WH Sales -0.5% (Tax)</field>
        <field name="code">H_SALE_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_withholding_tax_sale_tax_one" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -1% (Tax)</field>
        <field name="tag_name">WH Sales -1% (Tax)</field>
        <field name="code">O_SALE_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_withholding_tax_sale_tax_three" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -3% (Tax)</field>
        <field name="tag_name">WH Sales -3% (Tax)</field>
        <field name="code">T_SALE_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_withholding_tax_sale_tax_five" model="account.tax.report.line">
        <field name="name">Withholding Tax on Sales -5% (Tax)</field>
        <field name="tag_name">WH Sales -5% (Tax)</field>
        <field name="code">F_SALE_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_base" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases (Base)</field>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_base_half" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -0.5% (Base)</field>
        <field name="tag_name">WH Purchases -0.5% (Base)</field>
        <field name="code">H_PUR_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_base_one" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -1% (Base)</field>
        <field name="tag_name">WH Purchases -1% (Base)</field>
        <field name="code">O_PUR_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_base_three" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -3% (Base)</field>
        <field name="tag_name">WH Purchases -3% (Base)</field>
        <field name="code">T_PUR_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_base_five" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -5% (Base)</field>
        <field name="tag_name">WH Purchases -5% (Base)</field>
        <field name="code">F_PUR_B</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_tax" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases (Tax)</field>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_tax_half" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -0.5% (Tax)</field>
        <field name="tag_name">WH Purchases -0.5% (Tax)</field>
        <field name="code">H_PUR_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_tax_one" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -1% (Tax)</field>
        <field name="tag_name">WH Purchases -1% (Tax)</field>
        <field name="code">O_PUR_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_tax_three" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -3% (Tax)</field>
        <field name="tag_name">WH Purchases -3% (Tax)</field>
        <field name="code">T_PUR_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_withholding_tax_purchase_tax_five" model="account.tax.report.line">
        <field name="name">Withholding Tax on Purchases -5% (Tax)</field>
        <field name="tag_name">WH Purchases -5% (Tax)</field>
        <field name="code">F_PUR_T</field>
        <field name="parent_id" ref="tax_report_withholding_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_withholding_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_schedule_tax" model="account.tax.report">
        <field name="name">3. Schedule Tax</field>
        <field name="country_id" ref="base.eg"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales (Base)</field>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base_half" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 0.5% (Base)</field>
        <field name="tag_name">SCHD Sales 0.5% (Base)</field>
        <field name="code">H_SALE_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base_one" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 1% (Base)</field>
        <field name="tag_name">SCHD Sales 1% (Base)</field>
        <field name="code">O_SALE_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base_five" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 5% (Base)</field>
        <field name="tag_name">SCHD Sales 5% (Base)</field>
        <field name="code">F_SALE_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base_eight" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 8% (Base)</field>
        <field name="tag_name">SCHD Sales 8% (Base)</field>
        <field name="code">E_SALE_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base_ten" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 10% (Base)</field>
        <field name="tag_name">SCHD Sales 10% (Base)</field>
        <field name="code">T_SALE_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base_fifteen" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 15% (Base)</field>
        <field name="tag_name">SCHD Sales 15% (Base)</field>
        <field name="code">FF_SALE_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_base_thirty" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 30% (Base)</field>
        <field name="tag_name">SCHD Sales 30% (Base)</field>
        <field name="code">TY_SALE_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales (Tax)</field>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_half" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 0.5% (Tax)</field>
        <field name="tag_name">SCHD Sales 0.5% (Tax)</field>
        <field name="code">H_SALE_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_one" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 1% (Tax)</field>
        <field name="tag_name">SCHD Sales 1% (Tax)</field>
        <field name="code">O_SALE_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_five" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 5% (Tax)</field>
        <field name="tag_name">SCHD Sales 5% (Tax)</field>
        <field name="code">F_SALE_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_eight" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 8% (Tax)</field>
        <field name="tag_name">SCHD Sales 8% (Tax)</field>
        <field name="code">E_SALE_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_ten" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 10% (Tax)</field>
        <field name="tag_name">SCHD Sales 10% (Tax)</field>
        <field name="code">T_SALE_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_fifteen" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 15% (Tax)</field>
        <field name="tag_name">SCHD Sales 15% (Tax)</field>
        <field name="code">FF_SALE_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_thirty" model="account.tax.report.line">
        <field name="name">Schedule Tax on Sales 30% (Tax)</field>
        <field name="tag_name">SCHD Sales 30% (Tax)</field>
        <field name="code">TY_SALE_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_sale_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases (Base)</field>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_half" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 0.5% (Base)</field>
        <field name="tag_name">SCHD Purchases 0.5% (Base)</field>
        <field name="code">H_PUR_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_one" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 1% (Base)</field>
        <field name="tag_name">SCHD Purchases 1% (Base)</field>
        <field name="code">O_PUR_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_five" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 5% (Base)</field>
        <field name="tag_name">SCHD Purchases 5% (Base)</field>
        <field name="code">F_PUR_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_eight" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 8% (Base)</field>
        <field name="tag_name">SCHD Purchases 8% (Base)</field>
        <field name="code">E_PUR_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_ten" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 10% (Base)</field>
        <field name="tag_name">SCHD Purchases 10% (Base)</field>
        <field name="code">T_PUR_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_fifteen" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 15% (Base)</field>
        <field name="tag_name">SCHD Purchases 15% (Base)</field>
        <field name="code">FF_PUR_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_thirty" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 30% (Base)</field>
        <field name="tag_name">SCHD Purchases 30% (Base)</field>
        <field name="code">TY_PUR_SB</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_base"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases (Tax)</field>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_half" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 0.5% (Tax)</field>
        <field name="tag_name">SCHD Purchases 0.5% (Tax)</field>
        <field name="code">H_PUR_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_one" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 1% (Tax)</field>
        <field name="tag_name">SCHD Purchases 1% (Tax)</field>
        <field name="code">O_PUR_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_five" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 5% (Tax)</field>
        <field name="tag_name">SCHD Purchases 5% (Tax)</field>
        <field name="code">F_PUR_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_eight" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 8% (Tax)</field>
        <field name="tag_name">SCHD Purchases 8% (Tax)</field>
        <field name="code">E_PUR_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_ten" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 10% (Tax)</field>
        <field name="tag_name">SCHD Purchases 10% (Tax)</field>
        <field name="code">T_PUR_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 15% (Tax)</field>
        <field name="tag_name">SCHD Purchases 15% (Tax)</field>
        <field name="code">FF_PUR_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_thirty" model="account.tax.report.line">
        <field name="name">Schedule Tax on Purchases 30% (Tax)</field>
        <field name="tag_name">SCHD Purchases 30% (Tax)</field>
        <field name="code">TY_PUR_ST</field>
        <field name="parent_id" ref="tax_report_schedule_tax_schedule_tax_purchase_tax"/>
        <field name="report_id" ref="tax_report_schedule_tax"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_other_taxes" model="account.tax.report">
        <field name="name">4. Other Taxes</field>
        <field name="country_id" ref="base.eg"/>
    </record>

    <record id="tax_report_other_taxes_stamp_tax_base" model="account.tax.report.line">
        <field name="name">Stamp Tax Sales (Base)</field>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_other_taxes_stamp_tax_base_sales" model="account.tax.report.line">
        <field name="name">Stamp Tax Sales 20% (Base)</field>
        <field name="tag_name">Stamp Tax Sales 20% (Base)</field>
        <field name="code">STMP_TW_SB</field>
        <field name="parent_id" ref="tax_report_other_taxes_stamp_tax_base"/>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_other_taxes_stamp_tax_tax" model="account.tax.report.line">
        <field name="name">Stamp Tax Sales (Tax)</field>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_other_taxes_stamp_tax_tax_sales" model="account.tax.report.line">
        <field name="name">Stamp Tax Sales 20% (Tax)</field>
        <field name="tag_name">Stamp Tax Sales 20% (Tax)</field>
        <field name="code">STMP_TW_ST</field>
        <field name="parent_id" ref="tax_report_other_taxes_stamp_tax_tax"/>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_other_taxes_stamp_purchase_tax_base" model="account.tax.report.line">
        <field name="name">Stamp Tax Purchases (Base)</field>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_other_taxes_stamp_purchase_tax_base_purchase" model="account.tax.report.line">
        <field name="name">Stamp Tax Purchases 20% (Base)</field>
        <field name="tag_name">Stamp Tax Purchases 20% (Base)</field>
        <field name="code">STMP_TW_PB</field>
        <field name="parent_id" ref="tax_report_other_taxes_stamp_purchase_tax_base"/>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="1"/>
    </record>
    
    <record id="tax_report_other_taxes_stamp_purchase_tax_tax" model="account.tax.report.line">
        <field name="name">Stamp Tax Purchases (Tax)</field>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_other_taxes_stamp_purchase_tax_tax_purchase" model="account.tax.report.line">
        <field name="name">Stamp Tax Purchases 20% (Tax)</field>
        <field name="tag_name">Stamp Tax Purchases 20% (Tax)</field>
        <field name="code">STMP_TW_PT</field>
        <field name="parent_id" ref="tax_report_other_taxes_stamp_purchase_tax_tax"/>
        <field name="report_id" ref="tax_report_other_taxes"/>
        <field name="sequence" eval="1"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="eg_standard_sale_14" model="account.tax.template">
        <field name="name">VAT 14%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 14%</field>
        <field name="l10n_eg_eta_code">t1_v009</field>
        <field name="tax_group_id" ref="eg_tax_vat"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_vat_return_sale_base_fourteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201017'),
                'plus_report_line_ids': [ref('tax_report_vat_return_sale_tax_fourteen')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_vat_return_sale_base_fourteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201017'),
                'minus_report_line_ids': [ref('tax_report_vat_return_sale_tax_fourteen')],
            }),
        ]"/>
    </record>

    <record id="eg_standard_purchase_14" model="account.tax.template">
        <field name="name">VAT 14%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 14%</field>
        <field name="l10n_eg_eta_code">t1_v009</field>
        <field name="tax_group_id" ref="eg_tax_vat"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_vat_return_expense_base_fourteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104041'),
                'plus_report_line_ids': [ref('tax_report_vat_return_expense_tax_fourteen')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_vat_return_expense_base_fourteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104041'),
                'minus_report_line_ids': [ref('tax_report_vat_return_expense_tax_fourteen')],
            }),
        ]"/>
    </record>

    <record id="eg_zero_sale_0" model="account.tax.template">
        <field name="name">Zero Rated 0%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Zero Rated 0%</field>
        <field name="tax_group_id" ref="eg_tax_group_other"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_vat_return_sale_base_zero')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'plus_report_line_ids': [ref('tax_report_vat_return_sale_tax_zero')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_vat_return_sale_base_zero')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'minus_report_line_ids': [ref('tax_report_vat_return_sale_tax_zero')],
            }),
        ]"/>
    </record>

    <record id="eg_zero_purchase_0" model="account.tax.template">
        <field name="name">Zero Rated 0%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Zero Rated 0%</field>
        <field name="tax_group_id" ref="eg_tax_group_other"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_vat_return_expense_base_zero')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'plus_report_line_ids': [ref('tax_report_vat_return_expense_tax_zero')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_vat_return_expense_base_zero')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'minus_report_line_ids': [ref('tax_report_vat_return_expense_tax_zero')],
            }),
        ]"/>
    </record>

    <record id="eg_exempt_sale" model="account.tax.template">
        <field name="name">Exempt</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Exempt</field>
        <field name="l10n_eg_eta_code">t1_v003</field>
        <field name="tax_group_id" ref="eg_tax_group_other"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_vat_return_sale_base_exempt')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'plus_report_line_ids': [ref('tax_report_vat_return_sale_tax_exempt')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_vat_return_sale_base_exempt')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'minus_report_line_ids': [ref('tax_report_vat_return_sale_tax_exempt')],
            }),
        ]"/>
    </record>

    <record id="eg_exempt_purchase" model="account.tax.template">
        <field name="name">Exempt</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Exempt</field>
        <field name="l10n_eg_eta_code">t1_v003</field>
        <field name="tax_group_id" ref="eg_tax_group_other"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_vat_return_expense_base_exempt')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'plus_report_line_ids': [ref('tax_report_vat_return_expense_tax_exempt')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_vat_return_expense_base_exempt')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': False,
                'minus_report_line_ids': [ref('tax_report_vat_return_expense_tax_exempt')],
            }),
        ]"/>
    </record>

    <record id="eg_stamp_tax_20_sale" model="account.tax.template">
        <field name="name">Stamp</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">Stamp</field>
        <field name="l10n_eg_eta_code">t5_st01</field>
        <field name="tax_group_id" ref="eg_tax_group_stamp"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_other_taxes_stamp_tax_base_sales')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201025'),
                'plus_report_line_ids': [ref('tax_report_other_taxes_stamp_tax_tax_sales')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_other_taxes_stamp_tax_base_sales')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201025'),
                'minus_report_line_ids': [ref('tax_report_other_taxes_stamp_tax_tax_sales')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_stamp_tax_20_purchase" model="account.tax.template">
        <field name="name">Stamp</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">Stamp</field>
        <field name="l10n_eg_eta_code">t5_st01</field>
        <field name="tax_group_id" ref="eg_tax_group_stamp"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_other_taxes_stamp_purchase_tax_base_purchase')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400077'),
                'plus_report_line_ids': [ref('tax_report_other_taxes_stamp_purchase_tax_tax_purchase')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_other_taxes_stamp_purchase_tax_base_purchase')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400077'),
                'minus_report_line_ids': [ref('tax_report_other_taxes_stamp_purchase_tax_tax_purchase')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_8_purchase" model="account.tax.template">
        <field name="name">Schedule 8%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 8%</field>
        <field name="l10n_eg_eta_code">t2_tbl01</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_8"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_eight')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_eight')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_eight')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_eight')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_8_sale" model="account.tax.template">
        <field name="name">Schedule 8%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 8%</field>
        <field name="l10n_eg_eta_code">t2_tbl01</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_8"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_eight')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_eight')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_eight')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_eight')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_1_sale" model="account.tax.template">
        <field name="name">Withholding -1%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">-1</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -1%</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_1"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_10_purchase" model="account.tax.template">
        <field name="name">Schedule 10%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 10%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_10"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_ten')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_ten')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_ten')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_ten')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_05_sale" model="account.tax.template">
        <field name="name">Withholding -0.5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">-0.5</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -0.5%</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_half"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_half')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_half')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_05_purchase" model="account.tax.template">
        <field name="name">Withholding -0.5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-0.5</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -0.5%</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_half"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_half')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_half')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_1_purchase" model="account.tax.template">
        <field name="name">Withholding -1%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-1</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -1%</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_1"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_10_sale" model="account.tax.template">
        <field name="name">Schedule 10%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 10%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_10"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_ten')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_ten')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_ten')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_ten')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_3_sale" model="account.tax.template">
        <field name="name">Withholding -3%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">-3</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -3%</field>
        <field name="l10n_eg_eta_code">t4_w004</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_3"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_three')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_three')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_three')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_three')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_1_sale" model="account.tax.template">
        <field name="name">Schedule 1%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">1</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 1%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_1"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_3_purchase" model="account.tax.template">
        <field name="name">Withholding -3%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-3</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -3%</field>
        <field name="l10n_eg_eta_code">t4_w004</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_3"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_three')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_three')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_three')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_three')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_1_purchase" model="account.tax.template">
        <field name="name">Schedule 1%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">1</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 1%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_1"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_one')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_one')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_5_sale" model="account.tax.template">
        <field name="name">Withholding -5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -5%</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_5"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_sale_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_104042'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_sale_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_15_purchase" model="account.tax.template">
        <field name="name">Schedule 15%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">15</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 15%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_15"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_fifteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_fifteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_withholding_5_purchase" model="account.tax.template">
        <field name="name">Withholding -5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="description">WH -5%</field>
        <field name="tax_group_id" ref="eg_tax_group_withholding_5"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_withholding_tax_purchase_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201020'),
                'plus_report_line_ids': [ref('tax_report_withholding_tax_purchase_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_15_sale" model="account.tax.template">
        <field name="name">Schedule 15%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">15</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 15%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_15"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_fifteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_fifteen')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_fifteen')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_fifteen')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_30_sale" model="account.tax.template">
        <field name="name">Schedule 30%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">30</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 30%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_30"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_thirty')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_thirty')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_thirty')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_thirty')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_30_purchase" model="account.tax.template">
        <field name="name">Schedule 30%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">30</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 30%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_30"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_thirty')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_thirty')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_thirty')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_thirty')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_05_purchase" model="account.tax.template">
        <field name="name">Schedule 0.5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0.5</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 0.5%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_half"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_half')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_half')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_05_sale" model="account.tax.template">
        <field name="name">Schedule 0.5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0.5</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 0.5%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_half"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_half')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
             Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_half')],
            }),
             Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_half')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_5_purchase" model="account.tax.template">
        <field name="name">Schedule 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 5%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_5"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_400075'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_purchase_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>

    <record id="eg_schedule_tax_5_sale" model="account.tax.template">
        <field name="name">Schedule 5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">SCHD 5%</field>
        <field name="tax_group_id" ref="eg_tax_group_schedule_5"/>
        <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'plus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[Command.clear(),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_base_five')],
            }),
            Command.create({
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('egy_account_201024'),
                'minus_report_line_ids': [ref('tax_report_schedule_tax_schedule_tax_sale_tax_five')],
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\fiscal_templates_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="account_fiscal_position_egypt" model="account.fiscal.position.template">
            <field name="name">Egypt</field>
            <field name="sequence">19</field>
            <field name="auto_apply" eval="True"/>
            <field name="country_id" ref="base.eg"/>
            <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        </record>

        <record id="account_fiscal_position_non_egypt" model="account.fiscal.position.template">
            <field name="name">Non-Egypt</field>
            <field name="sequence">20</field>
            <field name="auto_apply" eval="True"/>
            <field name="chart_template_id" ref="egypt_chart_template_standard"/>
        </record>

        <record id="account_fiscal_position_tax_non_egypt_01" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="eg_standard_sale_14"/>
            <field name="tax_dest_id" ref="eg_zero_sale_0"/>
            <field name="position_id" ref="account_fiscal_position_non_egypt"/>
        </record>

        <record id="account_fiscal_position_tax_non_egypt_02" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="eg_standard_purchase_14"/>
            <field name="tax_dest_id" ref="eg_zero_purchase_0"/>
            <field name="position_id" ref="account_fiscal_position_non_egypt"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_eg_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <record id="egypt_chart_template_standard" model="account.chart.template">
         <field name="name">Egypt Chart of Accounts - Standard</field>
         <field name="code_digits">6</field>
         <field name="bank_account_code_prefix">101</field>
         <field name="cash_account_code_prefix">105</field>
         <field name="transfer_account_code_prefix">100</field>
         <field name="currency_id" ref="base.EGP"/>
         <field name="country_id" ref="base.eg"/>
         <field name="spoken_languages" eval="'en_US;ar_001;'"/>
     </record>
</odoo>

```

## File: data\l10n_eg_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="egypt_chart_template_standard" model="account.chart.template">
        <field name="account_journal_suspense_account_id" ref="egy_account_201001"/>
        <field name="property_account_receivable_id" ref="egy_account_102011"/>
        <field name="property_account_payable_id" ref="egy_account_201002"/>
        <field name="property_account_expense_categ_id" ref="egy_account_400028"/>
        <field name="property_account_income_categ_id" ref="egy_account_500001"/>
        <field name="property_account_expense_id" ref="egy_account_400028"/>
        <field name="property_account_income_id" ref="egy_account_500001"/>
        <field name="expense_currency_exchange_account_id" ref="egy_account_400053"/>
        <field name="income_currency_exchange_account_id" ref="egy_account_500011"/>
        <field name="default_pos_receivable_account_id" ref="egy_account_102012"/>
        <field name="default_cash_difference_income_account_id" ref="egy_account_999002"/>
        <field name="default_cash_difference_expense_account_id" ref="egy_account_999001"/>
        <field name="account_journal_payment_credit_account_id" ref="egy_account_105003"/>
        <field name="account_journal_payment_debit_account_id" ref="egy_account_101004"/>
        <field name="property_tax_payable_account_id" ref="egy_account_202003"/>
        <field name="property_tax_receivable_account_id" ref="egy_account_100103"/>
    </record>
</odoo>

```

## File: i18n_extra\l10n_eg.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_eg
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 15.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2022-02-18 10:29+0000\n"
"PO-Revision-Date: 2022-02-18 10:29+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_base_fourteen
msgid "1. Standard Rated 14% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_tax_fourteen
msgid "1. Standard Rated 14% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_base_fourteen
msgid "1. VAT 14% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_tax_fourteen
msgid "1. VAT 14% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_vat_return
msgid "1. VAT Return"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_withholding_tax
msgid "2. Withholding Tax"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_base_zero
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_base_zero
msgid "2. Zero Rated (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_tax_zero
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_tax_zero
msgid "2. Zero Rated (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_base_exempt
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_base_exempt
msgid "3. Exempt Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_tax_exempt
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_tax_exempt
msgid "3. Exempt Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_schedule_tax
msgid "3. Schedule Tax"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_other_taxes
msgid "4. Other Taxes"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_base_fourteen
msgid "5. Standard Rated 14% Expenses (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_tax_fourteen
msgid "5. Standard Rated 14% Expenses (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_base_fourteen
msgid "5. VAT 14% Expenses (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_tax_fourteen
msgid "5. VAT 14% Expenses (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_base_zero
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_base_zero
msgid "6. Zero Rated (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_tax_zero
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_tax_zero
msgid "6. Zero Rated (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_base_exempt
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_base_exempt
msgid "7. Exempt Expenses (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_tax_exempt
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_tax_exempt
msgid "7. Exempt Expenses (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106009
#: model:account.account,name:l10n_eg.2_egy_account_106009
#: model:account.account.template,name:l10n_eg.egy_account_106009
msgid "Acc. Depreciation of Motor Vehicles"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106008
#: model:account.account,name:l10n_eg.2_egy_account_106008
#: model:account.account.template,name:l10n_eg.egy_account_106008
msgid "Acc. Deprn.Computer Hardware & Software"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106007
#: model:account.account,name:l10n_eg.2_egy_account_106007
#: model:account.account.template,name:l10n_eg.egy_account_106007
msgid "Acc.Deprn.of Furniture & Office Equipment"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_tax_report
msgid "Account Tax Report"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_102011
#: model:account.account,name:l10n_eg.2_egy_account_102011
#: model:account.account.template,name:l10n_eg.egy_account_102011
msgid "Accounts Receivable"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_102012
#: model:account.account,name:l10n_eg.2_egy_account_102012
#: model:account.account.template,name:l10n_eg.egy_account_102012
msgid "Accounts Receivable (PoS)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201014
#: model:account.account,name:l10n_eg.2_egy_account_201014
#: model:account.account.template,name:l10n_eg.egy_account_201014
msgid "Accrued - Audit Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201007
#: model:account.account,name:l10n_eg.2_egy_account_201007
#: model:account.account.template,name:l10n_eg.egy_account_201007
msgid "Accrued - Commissions"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201015
#: model:account.account,name:l10n_eg.2_egy_account_201015
#: model:account.account.template,name:l10n_eg.egy_account_201015
msgid "Accrued - Office Rent"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201004
#: model:account.account,name:l10n_eg.2_egy_account_201004
#: model:account.account.template,name:l10n_eg.egy_account_201004
msgid "Accrued - Salaries"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201013
#: model:account.account,name:l10n_eg.2_egy_account_201013
#: model:account.account.template,name:l10n_eg.egy_account_201013
msgid "Accrued - Sponsorship"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201012
#: model:account.account,name:l10n_eg.2_egy_account_201012
#: model:account.account.template,name:l10n_eg.egy_account_201012
msgid "Accrued - Telephone"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201011
#: model:account.account,name:l10n_eg.2_egy_account_201011
#: model:account.account.template,name:l10n_eg.egy_account_201011
msgid "Accrued - Utilities"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201010
#: model:account.account,name:l10n_eg.2_egy_account_201010
#: model:account.account.template,name:l10n_eg.egy_account_201010
msgid "Accrued Other Personnel Cost"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201016
#: model:account.account,name:l10n_eg.2_egy_account_201016
#: model:account.account.template,name:l10n_eg.egy_account_201016
msgid "Accrued Others"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201008
#: model:account.account,name:l10n_eg.2_egy_account_201008
#: model:account.account.template,name:l10n_eg.egy_account_201008
msgid "Accrued Salaries Increment"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201009
#: model:account.account,name:l10n_eg.2_egy_account_201009
#: model:account.account.template,name:l10n_eg.egy_account_201009
msgid "Accrued-Staff Bonus"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_100102
#: model:account.account,name:l10n_eg.2_egy_account_100102
#: model:account.account.template,name:l10n_eg.egy_account_100102
msgid "Accumulated Depreciation right use asset (IFRS 16)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500005
#: model:account.account,name:l10n_eg.2_egy_account_500005
#: model:account.account.template,name:l10n_eg.egy_account_500005
msgid "Advertising Income"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400060
#: model:account.account,name:l10n_eg.2_egy_account_400060
#: model:account.account.template,name:l10n_eg.egy_account_400060
msgid "Air Miles Card Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400024
#: model:account.account,name:l10n_eg.2_egy_account_400024
#: model:account.account.template,name:l10n_eg.egy_account_400024
msgid "Air tickets"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400037
#: model:account.account,name:l10n_eg.2_egy_account_400037
#: model:account.account.template,name:l10n_eg.egy_account_400037
msgid "Amortisation of Preoperating Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106006
#: model:account.account,name:l10n_eg.2_egy_account_106006
#: model:account.account.template,name:l10n_eg.egy_account_106006
msgid "Amortisation on Leasehold Improvement"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400063
#: model:account.account,name:l10n_eg.2_egy_account_400063
#: model:account.account.template,name:l10n_eg.egy_account_400063
msgid "Amortization on Leasehold Improvement"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400029
#: model:account.account,name:l10n_eg.2_egy_account_400029
#: model:account.account.template,name:l10n_eg.egy_account_400029
msgid "Audit Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400074
#: model:account.account,name:l10n_eg.2_egy_account_400074
#: model:account.account.template,name:l10n_eg.egy_account_400074
msgid "Bad Debts"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400059
#: model:account.account,name:l10n_eg.2_egy_account_400059
#: model:account.account.template,name:l10n_eg.egy_account_400059
msgid "Bank Finance & Loan Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201001
#: model:account.account,name:l10n_eg.2_egy_account_201001
#: model:account.account.template,name:l10n_eg.egy_account_201001
msgid "Bank Suspense Account"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400003
#: model:account.account,name:l10n_eg.2_egy_account_400003
#: model:account.account.template,name:l10n_eg.egy_account_400003
msgid "Basic Salary"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500006
#: model:account.account,name:l10n_eg.2_egy_account_500006
#: model:account.account.template,name:l10n_eg.egy_account_500006
msgid "Branding Income"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500010
#: model:account.account,name:l10n_eg.2_egy_account_500010
#: model:account.account.template,name:l10n_eg.egy_account_500010
msgid "Capital Gain"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_999002
#: model:account.account,name:l10n_eg.2_egy_account_999002
#: model:account.account.template,name:l10n_eg.egy_account_999002
msgid "Cash Difference Gain"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_999001
#: model:account.account,name:l10n_eg.2_egy_account_999001
#: model:account.account.template,name:l10n_eg.egy_account_999001
msgid "Cash Difference Loss"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400038
#: model:account.account,name:l10n_eg.2_egy_account_400038
#: model:account.account.template,name:l10n_eg.egy_account_400038
msgid "Cash Shortage"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400044
#: model:account.account,name:l10n_eg.2_egy_account_400044
#: model:account.account.template,name:l10n_eg.egy_account_400044
msgid "Cleaning"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400069
#: model:account.account,name:l10n_eg.2_egy_account_400069
#: model:account.account.template,name:l10n_eg.egy_account_400069
msgid "Closing Account"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106011
#: model:account.account,name:l10n_eg.2_egy_account_106011
#: model:account.account.template,name:l10n_eg.egy_account_106011
msgid "Computer Card Renewal"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106003
#: model:account.account,name:l10n_eg.2_egy_account_106003
#: model:account.account.template,name:l10n_eg.egy_account_106003
msgid "Computer Hardware & Software"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400067
#: model:account.account,name:l10n_eg.2_egy_account_400067
#: model:account.account.template,name:l10n_eg.egy_account_400067
msgid "Consultancy Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400049
#: model:account.account,name:l10n_eg.2_egy_account_400049
#: model:account.account.template,name:l10n_eg.egy_account_400049
msgid "Convoyance Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400002
#: model:account.account,name:l10n_eg.2_egy_account_400002
#: model:account.account.template,name:l10n_eg.egy_account_400002
msgid "Cost Of Goods Sold I/C Sales"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400001
#: model:account.account,name:l10n_eg.2_egy_account_400001
#: model:account.account.template,name:l10n_eg.egy_account_400001
msgid "Cost of Goods Sold in Trading"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400021
#: model:account.account,name:l10n_eg.2_egy_account_400021
#: model:account.account.template,name:l10n_eg.egy_account_400021
msgid "Courrier"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400058
#: model:account.account,name:l10n_eg.2_egy_account_400058
#: model:account.account.template,name:l10n_eg.egy_account_400058
msgid "Credit Card Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400061
#: model:account.account,name:l10n_eg.2_egy_account_400061
#: model:account.account.template,name:l10n_eg.egy_account_400061
msgid "Credit Card Swipe Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201003
#: model:account.account,name:l10n_eg.2_egy_account_201003
#: model:account.account.template,name:l10n_eg.egy_account_201003
msgid "Credit Notes to Customers"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201023
#: model:account.account,name:l10n_eg.2_egy_account_201023
#: model:account.account.template,name:l10n_eg.egy_account_201023
msgid "Customer Provision"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201018
#: model:account.account,name:l10n_eg.2_egy_account_201018
#: model:account.account.template,name:l10n_eg.egy_account_201018
msgid "Deferred income"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104037
#: model:account.account,name:l10n_eg.2_egy_account_104037
#: model:account.account.template,name:l10n_eg.egy_account_104037
msgid "Deposit - Office Rent"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104040
#: model:account.account,name:l10n_eg.2_egy_account_104040
#: model:account.account.template,name:l10n_eg.egy_account_104040
msgid "Deposit Others"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104038
#: model:account.account,name:l10n_eg.2_egy_account_104038
#: model:account.account.template,name:l10n_eg.egy_account_104038
msgid "Deposits - Customs"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400065
#: model:account.account,name:l10n_eg.2_egy_account_400065
#: model:account.account.template,name:l10n_eg.egy_account_400065
msgid "Depreciation Of Computer Hard & Soft"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400064
#: model:account.account,name:l10n_eg.2_egy_account_400064
#: model:account.account.template,name:l10n_eg.egy_account_400064
msgid "Depreciation Of Furniture & Office Equipment"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400066
#: model:account.account,name:l10n_eg.2_egy_account_400066
#: model:account.account.template,name:l10n_eg.egy_account_400066
msgid "Depreciation Of Motor Vehicles"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400070
#: model:account.account,name:l10n_eg.2_egy_account_400070
#: model:account.account.template,name:l10n_eg.egy_account_400070
msgid "Depreciation on right of use asset (IFRS 16)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400054
#: model:account.account,name:l10n_eg.2_egy_account_400054
#: model:account.account.template,name:l10n_eg.egy_account_400054
msgid "Disposal of Business Branch"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields,field_description:l10n_eg.field_account_tax__l10n_eg_eta_code
#: model:ir.model.fields,field_description:l10n_eg.field_account_tax_template__l10n_eg_eta_code
#: model:ir.model.fields,field_description:l10n_eg.field_l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code
msgid "ETA Code (Egypt)"
msgstr ""

#. module: l10n_eg
#: model:account.fiscal.position,name:l10n_eg.1_account_fiscal_position_egypt
#: model:account.fiscal.position,name:l10n_eg.2_account_fiscal_position_egypt
#: model:account.fiscal.position.template,name:l10n_eg.account_fiscal_position_egypt
msgid "Egypt"
msgstr ""

#. module: l10n_eg
#: model:account.chart.template,name:l10n_eg.egypt_chart_template_standard
msgid "Egypt Chart of Accounts - Standard"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400008
#: model:account.account,name:l10n_eg.2_egy_account_400008
#: model:account.account.template,name:l10n_eg.egy_account_400008
msgid "End Of Service Indemnity"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_202001
#: model:account.account,name:l10n_eg.2_egy_account_202001
#: model:account.account.template,name:l10n_eg.egy_account_202001
msgid "End of Service Provision"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_exempt_purchase
#: model:account.tax,description:l10n_eg.1_eg_exempt_sale
#: model:account.tax,description:l10n_eg.2_eg_exempt_purchase
#: model:account.tax,description:l10n_eg.2_eg_exempt_sale
#: model:account.tax,name:l10n_eg.1_eg_exempt_purchase
#: model:account.tax,name:l10n_eg.1_eg_exempt_sale
#: model:account.tax,name:l10n_eg.2_eg_exempt_purchase
#: model:account.tax,name:l10n_eg.2_eg_exempt_sale
#: model:account.tax.template,description:l10n_eg.eg_exempt_purchase
#: model:account.tax.template,description:l10n_eg.eg_exempt_sale
#: model:account.tax.template,name:l10n_eg.eg_exempt_purchase
#: model:account.tax.template,name:l10n_eg.eg_exempt_sale
msgid "Exempt"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106002
#: model:account.account,name:l10n_eg.2_egy_account_106002
#: model:account.account.template,name:l10n_eg.egy_account_106002
msgid "Furniture and Equipment"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500011
#: model:account.account,name:l10n_eg.2_egy_account_500011
#: model:account.account.template,name:l10n_eg.egy_account_500011
msgid "Gain On Difference Of Exchange"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_101008
#: model:account.account,name:l10n_eg.2_egy_account_101008
#: model:account.account.template,name:l10n_eg.egy_account_101008
msgid "Gateway Credit Cards"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400046
#: model:account.account,name:l10n_eg.2_egy_account_400046
#: model:account.account.template,name:l10n_eg.egy_account_400046
msgid "Gifts & Donations"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_103019
#: model:account.account,name:l10n_eg.2_egy_account_103019
#: model:account.account.template,name:l10n_eg.egy_account_103019
msgid "Handling Difference in Inventory"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400025
#: model:account.account,name:l10n_eg.2_egy_account_400025
#: model:account.account.template,name:l10n_eg.egy_account_400025
msgid "Hotel"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400004
#: model:account.account,name:l10n_eg.2_egy_account_400004
#: model:account.account.template,name:l10n_eg.egy_account_400004
msgid "Housing Allowance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400055
#: model:account.account,name:l10n_eg.2_egy_account_400055
#: model:account.account.template,name:l10n_eg.egy_account_400055
msgid "Income Tax"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201027
#: model:account.account,name:l10n_eg.2_egy_account_201027
#: model:account.account.template,name:l10n_eg.egy_account_201027
msgid "Income Tax payable to Authority - Deducted from employee's salaries"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400040
#: model:account.account,name:l10n_eg.2_egy_account_400040
#: model:account.account.template,name:l10n_eg.egy_account_400040
msgid "Insurance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400072
#: model:account.account,name:l10n_eg.2_egy_account_400072
#: model:account.account.template,name:l10n_eg.egy_account_400072
msgid "Interest Expense"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500009
#: model:account.account,name:l10n_eg.2_egy_account_500009
#: model:account.account.template,name:l10n_eg.egy_account_500009
msgid "Interest Revenue"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_103020
#: model:account.account,name:l10n_eg.2_egy_account_103020
#: model:account.account.template,name:l10n_eg.egy_account_103020
msgid "Items Delivered to Customs on temprary Base"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400047
#: model:account.account,name:l10n_eg.2_egy_account_400047
#: model:account.account.template,name:l10n_eg.egy_account_400047
msgid "Kitchen and Buffet Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106001
#: model:account.account,name:l10n_eg.2_egy_account_106001
#: model:account.account.template,name:l10n_eg.egy_account_106001
msgid "Leasehold Improvement"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201006
#: model:account.account,name:l10n_eg.2_egy_account_201006
#: model:account.account.template,name:l10n_eg.egy_account_201006
msgid "Leave Days Provision"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400007
#: model:account.account,name:l10n_eg.2_egy_account_400007
#: model:account.account.template,name:l10n_eg.egy_account_400007
msgid "Leave Salary"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400006
#: model:account.account,name:l10n_eg.2_egy_account_400006
#: model:account.account.template,name:l10n_eg.egy_account_400006
msgid "Leave Ticket"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201005
#: model:account.account,name:l10n_eg.2_egy_account_201005
#: model:account.account.template,name:l10n_eg.egy_account_201005
msgid "Leave Tickets Provision"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201021
#: model:account.account,name:l10n_eg.2_egy_account_201021
#: model:account.account.template,name:l10n_eg.egy_account_201021
msgid "Legal Reserve"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400031
#: model:account.account,name:l10n_eg.2_egy_account_400031
#: model:account.account.template,name:l10n_eg.egy_account_400031
msgid "Legal fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400010
#: model:account.account,name:l10n_eg.2_egy_account_400010
#: model:account.account.template,name:l10n_eg.egy_account_400010
msgid "Life Insurance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egypt_chart_template_standard_liquidity_transfer
#: model:account.account,name:l10n_eg.2_egypt_chart_template_standard_liquidity_transfer
#: model:account.account.template,name:l10n_eg.egypt_chart_template_standard_liquidity_transfer
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400052
#: model:account.account,name:l10n_eg.2_egy_account_400052
#: model:account.account.template,name:l10n_eg.egy_account_400052
msgid "Loss On Fixed Assets Disposal"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400053
#: model:account.account,name:l10n_eg.2_egy_account_400053
#: model:account.account.template,name:l10n_eg.egy_account_400053
msgid "Loss on Difference on Exchange"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_101005
#: model:account.account,name:l10n_eg.2_egy_account_101005
#: model:account.account.template,name:l10n_eg.egy_account_101005
msgid "Main Safe"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_101006
#: model:account.account,name:l10n_eg.2_egy_account_101006
#: model:account.account.template,name:l10n_eg.egy_account_101006
msgid "Main Safe - Foreign Currency"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400042
#: model:account.account,name:l10n_eg.2_egy_account_400042
#: model:account.account.template,name:l10n_eg.egy_account_400042
msgid "Maintenance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500003
#: model:account.account,name:l10n_eg.2_egy_account_500003
#: model:account.account.template,name:l10n_eg.egy_account_500003
msgid "Management Consultancy Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_101009
#: model:account.account,name:l10n_eg.2_egy_account_101009
#: model:account.account.template,name:l10n_eg.egy_account_101009
msgid "Manual Visa & Master Cards"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400026
#: model:account.account,name:l10n_eg.2_egy_account_400026
#: model:account.account.template,name:l10n_eg.egy_account_400026
msgid "Meals"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400009
#: model:account.account,name:l10n_eg.2_egy_account_400009
#: model:account.account.template,name:l10n_eg.egy_account_400009
msgid "Medical Insurance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106004
#: model:account.account,name:l10n_eg.2_egy_account_106004
#: model:account.account.template,name:l10n_eg.egy_account_106004
msgid "Motor Vehicles"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields,field_description:l10n_eg.field_account_tax_report__name
msgid "Name"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields,help:l10n_eg.field_account_tax_report__name
msgid "Name of this tax report"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_net
msgid "Net VAT Due"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_net_3
msgid "Net VAT due (or reclaimed) for the period"
msgstr ""

#. module: l10n_eg
#: model:account.fiscal.position,name:l10n_eg.1_account_fiscal_position_non_egypt
#: model:account.fiscal.position,name:l10n_eg.2_account_fiscal_position_non_egypt
#: model:account.fiscal.position.template,name:l10n_eg.account_fiscal_position_non_egypt
msgid "Non-Egypt"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400016
#: model:account.account,name:l10n_eg.2_egy_account_400016
#: model:account.account.template,name:l10n_eg.egy_account_400016
msgid "Office Rent"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400034
#: model:account.account,name:l10n_eg.2_egy_account_400034
#: model:account.account.template,name:l10n_eg.egy_account_400034
msgid "Other - Advertising Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400051
#: model:account.account,name:l10n_eg.2_egy_account_400051
#: model:account.account.template,name:l10n_eg.egy_account_400051
msgid "Other Bank Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_102015
#: model:account.account,name:l10n_eg.2_egy_account_102015
#: model:account.account.template,name:l10n_eg.egy_account_102015
msgid "Other Debtors"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500013
#: model:account.account,name:l10n_eg.2_egy_account_500013
#: model:account.account.template,name:l10n_eg.egy_account_500013
msgid "Other Income"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400057
#: model:account.account,name:l10n_eg.2_egy_account_400057
#: model:account.account.template,name:l10n_eg.egy_account_400057
msgid "Other Non Operating Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104035
#: model:account.account,name:l10n_eg.2_egy_account_104035
#: model:account.account.template,name:l10n_eg.egy_account_104035
msgid "Other Prepayments"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_102014
#: model:account.account,name:l10n_eg.2_egy_account_102014
#: model:account.account.template,name:l10n_eg.egy_account_102014
msgid "Other Receivable"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_other
msgid "Other Taxes"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400019
#: model:account.account,name:l10n_eg.2_egy_account_400019
#: model:account.account.template,name:l10n_eg.egy_account_400019
msgid "Other Utility Cahrges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400028
#: model:account.account,name:l10n_eg.2_egy_account_400028
#: model:account.account.template,name:l10n_eg.egy_account_400028
msgid "Others"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400023
#: model:account.account,name:l10n_eg.2_egy_account_400023
#: model:account.account.template,name:l10n_eg.egy_account_400023
msgid "Others - Communication"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400050
#: model:account.account,name:l10n_eg.2_egy_account_400050
#: model:account.account.template,name:l10n_eg.egy_account_400050
msgid "Others - Office Various Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400033
#: model:account.account,name:l10n_eg.2_egy_account_400033
#: model:account.account.template,name:l10n_eg.egy_account_400033
msgid "Others - Professional Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400039
#: model:account.account,name:l10n_eg.2_egy_account_400039
#: model:account.account.template,name:l10n_eg.egy_account_400039
msgid "Others - Provision & Write off"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_105003
#: model:account.account,name:l10n_eg.2_egy_account_105003
#: model:account.account.template,name:l10n_eg.egy_account_105003
msgid "Outstanding Payments"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_101004
#: model:account.account,name:l10n_eg.2_egy_account_101004
#: model:account.account.template,name:l10n_eg.egy_account_101004
msgid "Outstanding Receipts"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_101010
#: model:account.account,name:l10n_eg.2_egy_account_101010
#: model:account.account.template,name:l10n_eg.egy_account_101010
msgid "PayPal Account"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400062
#: model:account.account,name:l10n_eg.2_egy_account_400062
#: model:account.account.template,name:l10n_eg.egy_account_400062
msgid "PayPal Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201002
#: model:account.account,name:l10n_eg.2_egy_account_201002
#: model:account.account.template,name:l10n_eg.egy_account_201002
msgid "Payables"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400027
#: model:account.account,name:l10n_eg.2_egy_account_400027
#: model:account.account.template,name:l10n_eg.egy_account_400027
msgid "Per Diem"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400015
#: model:account.account,name:l10n_eg.2_egy_account_400015
#: model:account.account.template,name:l10n_eg.egy_account_400015
msgid "Personnel Cost Others"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_102013
#: model:account.account,name:l10n_eg.2_egy_account_102013
#: model:account.account.template,name:l10n_eg.egy_account_102013
msgid "Post Dated Cheques Received"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104033
#: model:account.account,name:l10n_eg.2_egy_account_104033
#: model:account.account.template,name:l10n_eg.egy_account_104033
msgid "PrePaid Advertisement Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104034
#: model:account.account,name:l10n_eg.2_egy_account_104034
#: model:account.account.template,name:l10n_eg.egy_account_104034
msgid "Prepaid Bank Guarantee"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104030
#: model:account.account,name:l10n_eg.2_egy_account_104030
#: model:account.account.template,name:l10n_eg.egy_account_104030
msgid "Prepaid Consultancy Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104028
#: model:account.account,name:l10n_eg.2_egy_account_104028
#: model:account.account.template,name:l10n_eg.egy_account_104028
msgid "Prepaid Employees Housing"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104036
#: model:account.account,name:l10n_eg.2_egy_account_104036
#: model:account.account.template,name:l10n_eg.egy_account_104036
msgid "Prepaid Finance charge for Loans"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104031
#: model:account.account,name:l10n_eg.2_egy_account_104031
#: model:account.account.template,name:l10n_eg.egy_account_104031
msgid "Prepaid Legal Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104025
#: model:account.account,name:l10n_eg.2_egy_account_104025
#: model:account.account.template,name:l10n_eg.egy_account_104025
msgid "Prepaid License Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104022
#: model:account.account,name:l10n_eg.2_egy_account_104022
#: model:account.account.template,name:l10n_eg.egy_account_104022
msgid "Prepaid Life Insurance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104026
#: model:account.account,name:l10n_eg.2_egy_account_104026
#: model:account.account.template,name:l10n_eg.egy_account_104026
msgid "Prepaid Maintenance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104021
#: model:account.account,name:l10n_eg.2_egy_account_104021
#: model:account.account.template,name:l10n_eg.egy_account_104021
msgid "Prepaid Medical Insurance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104023
#: model:account.account,name:l10n_eg.2_egy_account_104023
#: model:account.account.template,name:l10n_eg.egy_account_104023
msgid "Prepaid Office Rent"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104024
#: model:account.account,name:l10n_eg.2_egy_account_104024
#: model:account.account.template,name:l10n_eg.egy_account_104024
msgid "Prepaid Other Insurance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104029
#: model:account.account,name:l10n_eg.2_egy_account_104029
#: model:account.account.template,name:l10n_eg.egy_account_104029
msgid "Prepaid Schooling Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104027
#: model:account.account,name:l10n_eg.2_egy_account_104027
#: model:account.account.template,name:l10n_eg.egy_account_104027
msgid "Prepaid Site Hosting Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400056
#: model:account.account,name:l10n_eg.2_egy_account_400056
#: model:account.account.template,name:l10n_eg.egy_account_400056
msgid "Previous Year Adjustments Account"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400068
#: model:account.account,name:l10n_eg.2_egy_account_400068
#: model:account.account.template,name:l10n_eg.egy_account_400068
msgid "Provision for Doubtful Debts"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_106010
#: model:account.account,name:l10n_eg.2_egy_account_106010
#: model:account.account.template,name:l10n_eg.egy_account_106010
msgid "Registration of Trademarks"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_202002
#: model:account.account,name:l10n_eg.2_egy_account_202002
#: model:account.account.template,name:l10n_eg.egy_account_202002
msgid "Reservations"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_100101
#: model:account.account,name:l10n_eg.2_egy_account_100101
#: model:account.account.template,name:l10n_eg.egy_account_100101
msgid "Right of use Asset (IFRS 16)"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_05_purchase
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_05_sale
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_05_purchase
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_05_sale
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_05_purchase
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_05_sale
msgid "SCHD 0.5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_1_purchase
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_1_sale
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_1_purchase
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_1_sale
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_1_purchase
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_1_sale
msgid "SCHD 1%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_10_purchase
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_10_sale
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_10_purchase
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_10_sale
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_10_purchase
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_10_sale
msgid "SCHD 10%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_15_purchase
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_15_sale
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_15_purchase
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_15_sale
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_15_purchase
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_15_sale
msgid "SCHD 15%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_30_purchase
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_30_sale
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_30_purchase
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_30_sale
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_30_purchase
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_30_sale
msgid "SCHD 30%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_5_purchase
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_5_sale
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_5_purchase
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_5_sale
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_5_purchase
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_5_sale
msgid "SCHD 5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_8_purchase
#: model:account.tax,description:l10n_eg.1_eg_schedule_tax_8_sale
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_8_purchase
#: model:account.tax,description:l10n_eg.2_eg_schedule_tax_8_sale
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_8_purchase
#: model:account.tax.template,description:l10n_eg.eg_schedule_tax_8_sale
msgid "SCHD 8%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_half
msgid "SCHD Purchases 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_half
msgid "SCHD Purchases 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_one
msgid "SCHD Purchases 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_one
msgid "SCHD Purchases 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_ten
msgid "SCHD Purchases 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_ten
msgid "SCHD Purchases 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_fifteen
msgid "SCHD Purchases 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen
msgid "SCHD Purchases 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_thirty
msgid "SCHD Purchases 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_thirty
msgid "SCHD Purchases 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_five
msgid "SCHD Purchases 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_five
msgid "SCHD Purchases 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_eight
msgid "SCHD Purchases 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_eight
msgid "SCHD Purchases 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_half
msgid "SCHD Sales 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_half
msgid "SCHD Sales 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_one
msgid "SCHD Sales 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_one
msgid "SCHD Sales 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_ten
msgid "SCHD Sales 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_ten
msgid "SCHD Sales 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_fifteen
msgid "SCHD Sales 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_fifteen
msgid "SCHD Sales 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_thirty
msgid "SCHD Sales 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_thirty
msgid "SCHD Sales 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_five
msgid "SCHD Sales 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_five
msgid "SCHD Sales 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_eight
msgid "SCHD Sales 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_eight
msgid "SCHD Sales 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500001
#: model:account.account,name:l10n_eg.2_egy_account_500001
#: model:account.account.template,name:l10n_eg.egy_account_500001
msgid "Sales Account"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400011
#: model:account.account,name:l10n_eg.2_egy_account_400011
#: model:account.account.template,name:l10n_eg.egy_account_400011
msgid "Sales Commission"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500004
#: model:account.account,name:l10n_eg.2_egy_account_500004
#: model:account.account.template,name:l10n_eg.egy_account_500004
msgid "Sales from Other Region"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500002
#: model:account.account,name:l10n_eg.2_egy_account_500002
#: model:account.account.template,name:l10n_eg.egy_account_500002
msgid "Sales of I/C"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_05_purchase
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_05_sale
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_05_purchase
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_05_sale
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_05_purchase
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_05_sale
msgid "Schedule 0.5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_1_purchase
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_1_sale
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_1_purchase
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_1_sale
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_1_purchase
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_1_sale
msgid "Schedule 1%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_10_purchase
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_10_sale
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_10_purchase
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_10_sale
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_10_purchase
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_10_sale
msgid "Schedule 10%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_15_purchase
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_15_sale
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_15_purchase
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_15_sale
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_15_purchase
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_15_sale
msgid "Schedule 15%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_30_purchase
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_30_sale
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_30_purchase
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_30_sale
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_30_purchase
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_30_sale
msgid "Schedule 30%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_5_purchase
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_5_sale
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_5_purchase
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_5_sale
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_5_purchase
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_5_sale
msgid "Schedule 5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_8_purchase
#: model:account.tax,name:l10n_eg.1_eg_schedule_tax_8_sale
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_8_purchase
#: model:account.tax,name:l10n_eg.2_eg_schedule_tax_8_sale
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_8_purchase
#: model:account.tax.template,name:l10n_eg.eg_schedule_tax_8_sale
msgid "Schedule 8%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_schedule_half
msgid "Schedule Tax 0.5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_schedule_1
msgid "Schedule Tax 1%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_schedule_10
msgid "Schedule Tax 10%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_schedule_15
msgid "Schedule Tax 15%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_schedule_30
msgid "Schedule Tax 30%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_schedule_5
msgid "Schedule Tax 5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_schedule_8
msgid "Schedule Tax 8%"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400075
#: model:account.account,name:l10n_eg.2_egy_account_400075
#: model:account.account.template,name:l10n_eg.egy_account_400075
msgid "Schedule Tax Expense"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201024
#: model:account.account,name:l10n_eg.2_egy_account_201024
#: model:account.account.template,name:l10n_eg.egy_account_201024
msgid "Schedule Tax collected & payable"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base
msgid "Schedule Tax on Purchases (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax
msgid "Schedule Tax on Purchases (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_half
msgid "Schedule Tax on Purchases 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_half
msgid "Schedule Tax on Purchases 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_one
msgid "Schedule Tax on Purchases 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_one
msgid "Schedule Tax on Purchases 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_ten
msgid "Schedule Tax on Purchases 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_ten
msgid "Schedule Tax on Purchases 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_fifteen
msgid "Schedule Tax on Purchases 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen
msgid "Schedule Tax on Purchases 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_thirty
msgid "Schedule Tax on Purchases 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_thirty
msgid "Schedule Tax on Purchases 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_five
msgid "Schedule Tax on Purchases 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_five
msgid "Schedule Tax on Purchases 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_eight
msgid "Schedule Tax on Purchases 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_eight
msgid "Schedule Tax on Purchases 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base
msgid "Schedule Tax on Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax
msgid "Schedule Tax on Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_half
msgid "Schedule Tax on Sales 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_half
msgid "Schedule Tax on Sales 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_one
msgid "Schedule Tax on Sales 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_one
msgid "Schedule Tax on Sales 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_ten
msgid "Schedule Tax on Sales 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_ten
msgid "Schedule Tax on Sales 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_fifteen
msgid "Schedule Tax on Sales 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_fifteen
msgid "Schedule Tax on Sales 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_thirty
msgid "Schedule Tax on Sales 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_thirty
msgid "Schedule Tax on Sales 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_five
msgid "Schedule Tax on Sales 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_five
msgid "Schedule Tax on Sales 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_eight
msgid "Schedule Tax on Sales 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_eight
msgid "Schedule Tax on Sales 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400043
#: model:account.account,name:l10n_eg.2_egy_account_400043
#: model:account.account.template,name:l10n_eg.egy_account_400043
msgid "Security & Guard"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500008
#: model:account.account,name:l10n_eg.2_egy_account_500008
#: model:account.account.template,name:l10n_eg.egy_account_500008
msgid "Service Income"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_103016
#: model:account.account,name:l10n_eg.2_egy_account_103016
#: model:account.account.template,name:l10n_eg.egy_account_103016
msgid "Shipment Insurance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_103018
#: model:account.account,name:l10n_eg.2_egy_account_103018
#: model:account.account.template,name:l10n_eg.egy_account_103018
msgid "Shipment Other Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_103017
#: model:account.account,name:l10n_eg.2_egy_account_103017
#: model:account.account.template,name:l10n_eg.egy_account_103017
msgid "Shipments Documentation Charges"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400078
#: model:account.account,name:l10n_eg.2_egy_account_400078
#: model:account.account.template,name:l10n_eg.egy_account_400078
msgid "Social Contibution - Company portion expense"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201026
#: model:account.account,name:l10n_eg.2_egy_account_201026
#: model:account.account.template,name:l10n_eg.egy_account_201026
msgid "Social Contribution - Payable to authorities"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_500007
#: model:account.account,name:l10n_eg.2_egy_account_500007
#: model:account.account.template,name:l10n_eg.egy_account_500007
msgid "Space Rental Income"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400012
#: model:account.account,name:l10n_eg.2_egy_account_400012
#: model:account.account.template,name:l10n_eg.egy_account_400012
msgid "Staff Other Allowances"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_stamp_tax_20_purchase
#: model:account.tax,description:l10n_eg.1_eg_stamp_tax_20_sale
#: model:account.tax,description:l10n_eg.2_eg_stamp_tax_20_purchase
#: model:account.tax,description:l10n_eg.2_eg_stamp_tax_20_sale
#: model:account.tax,name:l10n_eg.1_eg_stamp_tax_20_purchase
#: model:account.tax,name:l10n_eg.1_eg_stamp_tax_20_sale
#: model:account.tax,name:l10n_eg.2_eg_stamp_tax_20_purchase
#: model:account.tax,name:l10n_eg.2_eg_stamp_tax_20_sale
#: model:account.tax.template,description:l10n_eg.eg_stamp_tax_20_purchase
#: model:account.tax.template,description:l10n_eg.eg_stamp_tax_20_sale
#: model:account.tax.template,name:l10n_eg.eg_stamp_tax_20_purchase
#: model:account.tax.template,name:l10n_eg.eg_stamp_tax_20_sale
msgid "Stamp"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_stamp
msgid "Stamp Tax 20%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_base
msgid "Stamp Tax Purchases (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_tax
msgid "Stamp Tax Purchases (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_base_purchase
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_base_purchase
msgid "Stamp Tax Purchases 20% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_tax_purchase
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_tax_purchase
msgid "Stamp Tax Purchases 20% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_base
msgid "Stamp Tax Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_tax
msgid "Stamp Tax Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_base_sales
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_tax_base_sales
msgid "Stamp Tax Sales 20% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_tax_sales
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_tax_tax_sales
msgid "Stamp Tax Sales 20% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201025
#: model:account.account,name:l10n_eg.2_egy_account_201025
#: model:account.account.template,name:l10n_eg.egy_account_201025
msgid "Stamp Tax payable"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400077
#: model:account.account,name:l10n_eg.2_egy_account_400077
#: model:account.account.template,name:l10n_eg.egy_account_400077
msgid "Stamp tax expense"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400045
#: model:account.account,name:l10n_eg.2_egy_account_400045
#: model:account.account.template,name:l10n_eg.egy_account_400045
msgid "Subscriptions"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v001
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v001
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v001
msgid "T1 - V001 - Export"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v002
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v002
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v002
msgid "T1 - V002 - Export to free areas and other areas"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v003
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v003
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v003
msgid "T1 - V003 - Exempted good or service"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v004
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v004
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v004
msgid "T1 - V004 - A non-taxable good or service"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v005
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v005
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v005
msgid "T1 - V005 - Exemptions for diplomats, consulates and embassies"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v006
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v006
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v006
msgid "T1 - V006 - Defence and National security Exemptions"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v007
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v007
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v007
msgid "T1 - V007 - Agreements exemptions"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v008
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v008
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v008
msgid "T1 - V008 - Special Exemptios and other reasons"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v009
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v009
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v009
msgid "T1 - V009 - General Item sales"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v010
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v010
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v010
msgid "T1 - V010 - Other Rates"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t10_mn01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t10_mn01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t10_mn01
msgid "T10 - Mn01 - Municipality Fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t10_mn02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t10_mn02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t10_mn02
msgid "T10 - Mn02 - Municipality Fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t11_mi01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t11_mi01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t11_mi01
msgid "T11 - MI01 - Medical insurance fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t11_mi02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t11_mi02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t11_mi02
msgid "T11 - MI02 - Medical insurance fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t12_of01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t12_of01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t12_of01
msgid "T12 - OF01 - Other fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t12_of02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t12_of02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t12_of02
msgid "T12 - OF02 - Other fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t13_st03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t13_st03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t13_st03
msgid "T13 - ST03 - Stamping tax (percentage)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t14_st04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t14_st04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t14_st04
msgid "T14 - ST04 - Stamping Tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t15_ent03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t15_ent03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t15_ent03
msgid "T15 - Ent03 - Entertainment tax (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t15_ent04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t15_ent04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t15_ent04
msgid "T15 - Ent04 - Entertainment tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t16_rd03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t16_rd03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t16_rd03
msgid "T16 - RD03 - Resource development fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t16_rd04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t16_rd04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t16_rd04
msgid "T16 - RD04 - Resource development fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t17_sc03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t17_sc03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t17_sc03
msgid "T17 - SC03 - Service charges (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t17_sc04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t17_sc04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t17_sc04
msgid "T17 - SC04 - Service charges (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t18_mn03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t18_mn03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t18_mn03
msgid "T18 - Mn03 - Municipality Fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t18_mn04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t18_mn04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t18_mn04
msgid "T18 - Mn04 - Municipality Fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t19_mi03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t19_mi03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t19_mi03
msgid "T19 - MI03 - Medical insurance fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t19_mi04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t19_mi04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t19_mi04
msgid "T19 - MI04 - Medical insurance fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t2_tbl01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t2_tbl01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t2_tbl01
msgid "T2 - Tbl01 - Table tax (percentage)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t20_of03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t20_of03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t20_of03
msgid "T20 - OF03 - Other fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t20_of04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t20_of04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t20_of04
msgid "T20 - OF04 - Other fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t3_tbl02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t3_tbl02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t3_tbl02
msgid "T3 - Tbl02 - Table tax (Fixed Amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w001
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w001
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w001
msgid "T4 - W001 - Contracting"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w002
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w002
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w002
msgid "T4 - W002 - Supplies"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w003
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w003
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w003
msgid "T4 - W003 - Purachases"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w004
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w004
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w004
msgid "T4 - W004 - Services"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w005
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w005
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w005
msgid ""
"T4 - W005 - Sums paid by the cooperative societies for car transportation to"
" their members"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w006
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w006
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w006
msgid "T4 - W006 - Commissionagency & brokerage"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w007
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w007
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w007
msgid ""
"T4 - W007 - Discounts & grants & additional exceptional incentives(smoke, "
"cement companies)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w008
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w008
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w008
msgid ""
"T4 - W008 - All discounts & grants & commissions (petroleum, "
"telecommunications, other companies)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w009
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w009
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w009
msgid "T4 - W009 - Supporting export subsidies"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w010
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w010
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w010
msgid "T4 - W010 - Professional fees"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w011
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w011
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w011
msgid "T4 - W011 - Commission & brokerage _A_57"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w012
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w012
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w012
msgid "T4 - W012 - Hospitals collecting from doctors"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w013
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w013
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w013
msgid "T4 - W013 - Royalties"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w014
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w014
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w014
msgid "T4 - W014 - Customs clearance"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w015
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w015
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w015
msgid "T4 - W015 - Exemption"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w016
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w016
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w016
msgid "T4 - W016 - advance payments"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t5_st01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t5_st01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t5_st01
msgid "T5 - ST01 - Stamping tax (percentage)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t6_st02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t6_st02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t6_st02
msgid "T6 - ST02 - Stamping Tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t7_ent01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t7_ent01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t7_ent01
msgid "T7 - Ent01 - Entertainment tax (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t7_ent02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t7_ent02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t7_ent02
msgid "T7 - Ent02 - Entertainment tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t8_rd01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t8_rd01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t8_rd01
msgid "T8 - RD01 - Resource development fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t8_rd02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t8_rd02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t8_rd02
msgid "T8 - RD02 - Resource development fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t9_sc01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t9_sc01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t9_sc01
msgid "T9 - SC01 - Service charges (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t9_sc02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t9_sc02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t9_sc02
msgid "T9 - SC02 - Service charges (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_tax
msgid "Tax"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201022
#: model:account.account,name:l10n_eg.2_egy_account_201022
#: model:account.account.template,name:l10n_eg.egy_account_201022
msgid "Taxes Provision"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400020
#: model:account.account,name:l10n_eg.2_egy_account_400020
#: model:account.account.template,name:l10n_eg.egy_account_400020
msgid "Telephone"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_tax_template
msgid "Templates for Taxes"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_net_1
msgid "Total value of due tax for the period"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_net_2
msgid "Total value of recoverable tax for the period"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400032
#: model:account.account,name:l10n_eg.2_egy_account_400032
#: model:account.account.template,name:l10n_eg.egy_account_400032
msgid "Trade License Fees"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400041
#: model:account.account,name:l10n_eg.2_egy_account_400041
#: model:account.account.template,name:l10n_eg.egy_account_400041
msgid "Training"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400005
#: model:account.account,name:l10n_eg.2_egy_account_400005
#: model:account.account.template,name:l10n_eg.egy_account_400005
msgid "Transportation Allowance"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_999999
#: model:account.account,name:l10n_eg.2_egy_account_999999
#: model:account.account.template,name:l10n_eg.egy_account_999999
msgid "Undistributed Profits/Losses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400013
#: model:account.account,name:l10n_eg.2_egy_account_400013
#: model:account.account.template,name:l10n_eg.egy_account_400013
msgid "Uniform"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_standard_purchase_14
#: model:account.tax,description:l10n_eg.1_eg_standard_sale_14
#: model:account.tax,description:l10n_eg.2_eg_standard_purchase_14
#: model:account.tax,description:l10n_eg.2_eg_standard_sale_14
#: model:account.tax,name:l10n_eg.1_eg_standard_purchase_14
#: model:account.tax,name:l10n_eg.1_eg_standard_sale_14
#: model:account.tax,name:l10n_eg.2_eg_standard_purchase_14
#: model:account.tax,name:l10n_eg.2_eg_standard_sale_14
#: model:account.tax.group,name:l10n_eg.eg_tax_vat
#: model:account.tax.template,description:l10n_eg.eg_standard_purchase_14
#: model:account.tax.template,description:l10n_eg.eg_standard_sale_14
#: model:account.tax.template,name:l10n_eg.eg_standard_purchase_14
#: model:account.tax.template,name:l10n_eg.eg_standard_sale_14
msgid "VAT 14%"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104041
#: model:account.account,name:l10n_eg.2_egy_account_104041
#: model:account.account.template,name:l10n_eg.egy_account_104041
msgid "VAT Input"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201017
#: model:account.account,name:l10n_eg.2_egy_account_201017
#: model:account.account.template,name:l10n_eg.egy_account_201017
msgid "VAT Output"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_202003
#: model:account.account,name:l10n_eg.2_egy_account_202003
#: model:account.account.template,name:l10n_eg.egy_account_202003
msgid "VAT Payable"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_100103
#: model:account.account,name:l10n_eg.2_egy_account_100103
#: model:account.account.template,name:l10n_eg.egy_account_100103
msgid "VAT Receivable"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_base
msgid "VAT on Expenses and all other Inputs (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_expense_tax
msgid "VAT on Expenses and all other Inputs (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_base
msgid "VAT on Sales and all other Outputs (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_vat_return_sale_tax
msgid "VAT on Sales and all other Outputs (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400048
#: model:account.account,name:l10n_eg.2_egy_account_400048
#: model:account.account.template,name:l10n_eg.egy_account_400048
msgid "Vehicle Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_101007
#: model:account.account,name:l10n_eg.2_egy_account_101007
#: model:account.account.template,name:l10n_eg.egy_account_101007
msgid "Visa & Master Credit Cards"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400014
#: model:account.account,name:l10n_eg.2_egy_account_400014
#: model:account.account.template,name:l10n_eg.egy_account_400014
msgid "Visa Expenses"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_withholding_05_purchase
#: model:account.tax,description:l10n_eg.1_eg_withholding_05_sale
#: model:account.tax,description:l10n_eg.2_eg_withholding_05_purchase
#: model:account.tax,description:l10n_eg.2_eg_withholding_05_sale
#: model:account.tax.template,description:l10n_eg.eg_withholding_05_purchase
#: model:account.tax.template,description:l10n_eg.eg_withholding_05_sale
msgid "WH -0.5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_withholding_1_purchase
#: model:account.tax,description:l10n_eg.1_eg_withholding_1_sale
#: model:account.tax,description:l10n_eg.2_eg_withholding_1_purchase
#: model:account.tax,description:l10n_eg.2_eg_withholding_1_sale
#: model:account.tax.template,description:l10n_eg.eg_withholding_1_purchase
#: model:account.tax.template,description:l10n_eg.eg_withholding_1_sale
msgid "WH -1%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_withholding_3_purchase
#: model:account.tax,description:l10n_eg.1_eg_withholding_3_sale
#: model:account.tax,description:l10n_eg.2_eg_withholding_3_purchase
#: model:account.tax,description:l10n_eg.2_eg_withholding_3_sale
#: model:account.tax.template,description:l10n_eg.eg_withholding_3_purchase
#: model:account.tax.template,description:l10n_eg.eg_withholding_3_sale
msgid "WH -3%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_withholding_5_purchase
#: model:account.tax,description:l10n_eg.1_eg_withholding_5_sale
#: model:account.tax,description:l10n_eg.2_eg_withholding_5_purchase
#: model:account.tax,description:l10n_eg.2_eg_withholding_5_sale
#: model:account.tax.template,description:l10n_eg.eg_withholding_5_purchase
#: model:account.tax.template,description:l10n_eg.eg_withholding_5_sale
msgid "WH -5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_half
msgid "WH Purchases -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_half
msgid "WH Purchases -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_one
msgid "WH Purchases -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_one
msgid "WH Purchases -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_three
msgid "WH Purchases -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_three
msgid "WH Purchases -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_five
msgid "WH Purchases -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_five
msgid "WH Purchases -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_half
msgid "WH Sales -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_half
msgid "WH Sales -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_one
msgid "WH Sales -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_three
msgid "WH Sales -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_five
msgid "WH Sales -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400076
#: model:account.account,name:l10n_eg.2_egy_account_400076
#: model:account.account.template,name:l10n_eg.egy_account_400076
msgid "WH Tax Expense"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_one
msgid "WH on Sales -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_three
msgid "WH on Sales -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_five
msgid "WH on Sales -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_104042
#: model:account.account,name:l10n_eg.2_egy_account_104042
#: model:account.account.template,name:l10n_eg.egy_account_104042
msgid "WH tax Advance with Customers - On behalf of my company"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_201020
#: model:account.account,name:l10n_eg.2_egy_account_201020
#: model:account.account.template,name:l10n_eg.egy_account_201020
msgid "WHTax Payable - On behalf of suppliers"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400017
#: model:account.account,name:l10n_eg.2_egy_account_400017
#: model:account.account.template,name:l10n_eg.egy_account_400017
msgid "Warehouse Rent"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400018
#: model:account.account,name:l10n_eg.2_egy_account_400018
#: model:account.account.template,name:l10n_eg.egy_account_400018
msgid "Water & Electricity"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400022
#: model:account.account,name:l10n_eg.2_egy_account_400022
#: model:account.account.template,name:l10n_eg.egy_account_400022
msgid "Web Site Hosting Fees"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_withholding_05_purchase
#: model:account.tax,name:l10n_eg.1_eg_withholding_05_sale
#: model:account.tax,name:l10n_eg.2_eg_withholding_05_purchase
#: model:account.tax,name:l10n_eg.2_eg_withholding_05_sale
#: model:account.tax.template,name:l10n_eg.eg_withholding_05_purchase
#: model:account.tax.template,name:l10n_eg.eg_withholding_05_sale
msgid "Withholding -0.5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_withholding_1_purchase
#: model:account.tax,name:l10n_eg.1_eg_withholding_1_sale
#: model:account.tax,name:l10n_eg.2_eg_withholding_1_purchase
#: model:account.tax,name:l10n_eg.2_eg_withholding_1_sale
#: model:account.tax.template,name:l10n_eg.eg_withholding_1_purchase
#: model:account.tax.template,name:l10n_eg.eg_withholding_1_sale
msgid "Withholding -1%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_withholding_3_purchase
#: model:account.tax,name:l10n_eg.1_eg_withholding_3_sale
#: model:account.tax,name:l10n_eg.2_eg_withholding_3_purchase
#: model:account.tax,name:l10n_eg.2_eg_withholding_3_sale
#: model:account.tax.template,name:l10n_eg.eg_withholding_3_purchase
#: model:account.tax.template,name:l10n_eg.eg_withholding_3_sale
msgid "Withholding -3%"
msgstr ""

#. module: l10n_eg
#: model:account.tax,name:l10n_eg.1_eg_withholding_5_purchase
#: model:account.tax,name:l10n_eg.1_eg_withholding_5_sale
#: model:account.tax,name:l10n_eg.2_eg_withholding_5_purchase
#: model:account.tax,name:l10n_eg.2_eg_withholding_5_sale
#: model:account.tax.template,name:l10n_eg.eg_withholding_5_purchase
#: model:account.tax.template,name:l10n_eg.eg_withholding_5_sale
msgid "Withholding -5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_withholding_half
msgid "Withholding Tax -0.5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_withholding_1
msgid "Withholding Tax -1%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_withholding_3
msgid "Withholding Tax -3%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.group,name:l10n_eg.eg_tax_group_withholding_5
msgid "Withholding Tax -5%"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base
msgid "Withholding Tax on Purchases (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax
msgid "Withholding Tax on Purchases (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_half
msgid "Withholding Tax on Purchases -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_half
msgid "Withholding Tax on Purchases -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_one
msgid "Withholding Tax on Purchases -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_one
msgid "Withholding Tax on Purchases -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_three
msgid "Withholding Tax on Purchases -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_three
msgid "Withholding Tax on Purchases -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_five
msgid "Withholding Tax on Purchases -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_five
msgid "Withholding Tax on Purchases -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base
msgid "Withholding Tax on Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax
msgid "Withholding Tax on Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_half
msgid "Withholding Tax on Sales -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_half
msgid "Withholding Tax on Sales -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_one
msgid "Withholding Tax on Sales -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_one
msgid "Withholding Tax on Sales -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_three
msgid "Withholding Tax on Sales -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_three
msgid "Withholding Tax on Sales -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_five
msgid "Withholding Tax on Sales -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_five
msgid "Withholding Tax on Sales -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400036
#: model:account.account,name:l10n_eg.2_egy_account_400036
#: model:account.account.template,name:l10n_eg.egy_account_400036
msgid "Write Off Inventory"
msgstr ""

#. module: l10n_eg
#: model:account.account,name:l10n_eg.1_egy_account_400035
#: model:account.account,name:l10n_eg.2_egy_account_400035
#: model:account.account.template,name:l10n_eg.egy_account_400035
msgid "Write Off Receivables & Payables"
msgstr ""

#. module: l10n_eg
#: model:account.tax,description:l10n_eg.1_eg_zero_purchase_0
#: model:account.tax,description:l10n_eg.1_eg_zero_sale_0
#: model:account.tax,description:l10n_eg.2_eg_zero_purchase_0
#: model:account.tax,description:l10n_eg.2_eg_zero_sale_0
#: model:account.tax,name:l10n_eg.1_eg_zero_purchase_0
#: model:account.tax,name:l10n_eg.1_eg_zero_sale_0
#: model:account.tax,name:l10n_eg.2_eg_zero_purchase_0
#: model:account.tax,name:l10n_eg.2_eg_zero_sale_0
#: model:account.tax.template,description:l10n_eg.eg_zero_purchase_0
#: model:account.tax.template,description:l10n_eg.eg_zero_sale_0
#: model:account.tax.template,name:l10n_eg.eg_zero_purchase_0
#: model:account.tax.template,name:l10n_eg.eg_zero_sale_0
msgid "Zero Rated 0%"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_l10n_eg_eta_account_tax_mixin
msgid "l10n_eg.eta.account.tax.mixin"
msgstr ""

```

## File: models\account_chart_template.py

```python
from odoo import models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        """ If EGYPT chart, we add 2 new journals TA and IFRS"""
        if self == self.env.ref('l10n_eg.egypt_chart_template_standard'):
            if not journals_dict:
                journals_dict = []
            journals_dict.extend(
                [{"name": "Tax Adjustments", "company_id": company.id, "code": "TA", "type": "general", "sequence": 1,
                  "favorite": True},
                 {"name": "IFRS 16", "company_id": company.id, "code": "IFRS", "type": "general", "favorite": True,
                  "sequence": 10}])
        return super()._prepare_all_journals(acc_template_ref, company, journals_dict=journals_dict)
   
```

## File: models\account_tax.py

```python
from odoo import models, fields


class ETAAccountTaxMixin(models.AbstractModel):
    _name = 'l10n_eg.eta.account.tax.mixin'
    _description = 'ETA tax codes mixin'

    l10n_eg_eta_code = fields.Selection(
        selection=[
            ('t1_v001', 'T1 - V001 - Export'),
            ('t1_v002', 'T1 - V002 - Export to free areas and other areas'),
            ('t1_v003', 'T1 - V003 - Exempted good or service'),
            ('t1_v004', 'T1 - V004 - A non-taxable good or service'),
            ('t1_v005', 'T1 - V005 - Exemptions for diplomats, consulates and embassies'),
            ('t1_v006', 'T1 - V006 - Defence and National security Exemptions'),
            ('t1_v007', 'T1 - V007 - Agreements exemptions'),
            ('t1_v008', 'T1 - V008 - Special Exemption and other reasons'),
            ('t1_v009', 'T1 - V009 - General Item sales'),
            ('t1_v010', 'T1 - V010 - Other Rates'),
            ('t2_tbl01', 'T2 - Tbl01 - Table tax (percentage)'),
            ('t3_tbl02', 'T3 - Tbl02 - Table tax (Fixed Amount)'),
            ('t4_w001', 'T4 - W001 - Contracting'),
            ('t4_w002', 'T4 - W002 - Supplies'),
            ('t4_w003', 'T4 - W003 - Purchases'),
            ('t4_w004', 'T4 - W004 - Services'),
            ('t4_w005', 'T4 - W005 - Sums paid by the cooperative societies for car transportation to their members'),
            ('t4_w006', 'T4 - W006 - Commission agency & brokerage'),
            ('t4_w007', 'T4 - W007 - Discounts & grants & additional exceptional incentives (smoke, cement companies)'),
            ('t4_w008', 'T4 - W008 - All discounts & grants & commissions (petroleum, telecommunications, and other)'),
            ('t4_w009', 'T4 - W009 - Supporting export subsidies'),
            ('t4_w010', 'T4 - W010 - Professional fees'),
            ('t4_w011', 'T4 - W011 - Commission & brokerage _A_57'),
            ('t4_w012', 'T4 - W012 - Hospitals collecting from doctors'),
            ('t4_w013', 'T4 - W013 - Royalties'),
            ('t4_w014', 'T4 - W014 - Customs clearance'),
            ('t4_w015', 'T4 - W015 - Exemption'),
            ('t4_w016', 'T4 - W016 - advance payments'),
            ('t5_st01', 'T5 - ST01 - Stamping tax (percentage)'),
            ('t6_st02', 'T6 - ST02 - Stamping Tax (amount)'),
            ('t7_ent01', 'T7 - Ent01 - Entertainment tax (rate)'),
            ('t7_ent02', 'T7 - Ent02 - Entertainment tax (amount)'),
            ('t8_rd01', 'T8 - RD01 - Resource development fee (rate)'),
            ('t8_rd02', 'T8 - RD02 - Resource development fee (amount)'),
            ('t9_sc01', 'T9 - SC01 - Service charges (rate)'),
            ('t9_sc02', 'T9 - SC02 - Service charges (amount)'),
            ('t10_mn01', 'T10 - Mn01 - Municipality Fees (rate)'),
            ('t10_mn02', 'T10 - Mn02 - Municipality Fees (amount)'),
            ('t11_mi01', 'T11 - MI01 - Medical insurance fee (rate)'),
            ('t11_mi02', 'T11 - MI02 - Medical insurance fee (amount)'),
            ('t12_of01', 'T12 - OF01 - Other fees (rate)'),
            ('t12_of02', 'T12 - OF02 - Other fees (amount)'),
            ('t13_st03', 'T13 - ST03 - Stamping tax (percentage)'),
            ('t14_st04', 'T14 - ST04 - Stamping Tax (amount)'),
            ('t15_ent03', 'T15 - Ent03 - Entertainment tax (rate)'),
            ('t15_ent04', 'T15 - Ent04 - Entertainment tax (amount)'),
            ('t16_rd03', 'T16 - RD03 - Resource development fee (rate)'),
            ('t16_rd04', 'T16 - RD04 - Resource development fee (amount)'),
            ('t17_sc03', 'T17 - SC03 - Service charges (rate)'),
            ('t17_sc04', 'T17 - SC04 - Service charges (amount)'),
            ('t18_mn03', 'T18 - Mn03 - Municipality Fees (rate)'),
            ('t18_mn04', 'T18 - Mn04 - Municipality Fees (amount)'),
            ('t19_mi03', 'T19 - MI03 - Medical insurance fee (rate)'),
            ('t19_mi04', 'T19 - MI04 - Medical insurance fee (amount)'),
            ('t20_of03', 'T20 - OF03 - Other fees (rate)'),
            ('t20_of04', 'T20 - OF04 - Other fees (amount)')
        ],
        string='ETA Code (Egypt)', default=False)


class AccountTax(models.Model):
    _name = 'account.tax'
    _inherit = ['account.tax', 'l10n_eg.eta.account.tax.mixin']


class AccountTaxTemplate(models.Model):
    _name = 'account.tax.template'
    _inherit = ['account.tax.template', 'l10n_eg.eta.account.tax.mixin']

    def _get_tax_vals(self, company, tax_template_to_tax):
        vals = super(AccountTaxTemplate, self)._get_tax_vals(company, tax_template_to_tax)
        vals.update({
            'l10n_eg_eta_code': self.l10n_eg_eta_code,
        })
        return vals

```

## File: models\__init__.py

```python
from . import account_chart_template
from . import account_tax

```

## File: views\account_tax.xml

```xml
<data>
    <record id="view_account_tax_form" model="ir.ui.view">
        <field name="name">account.tax.form</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <field name="tax_scope" position="after">
                <field name="l10n_eg_eta_code" attrs="{'invisible': [('country_code', '!=', 'EG')]}"/>
            </field>
        </field>
    </record>

    <record id="view_tax_eta_code_tree" model="ir.ui.view">
        <field name="name">account.tax.eta.code.tree</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_tree" />
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="l10n_eg_eta_code" optional="hide"/>
            </field>
        </field>
    </record>
</data>

```

