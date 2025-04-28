# Odoo Module: l10n_ae

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'United Arab Emirates - Accounting',
    'author': 'Odoo S.A.',
    'category': 'Accounting/Localizations/Account Charts',
    'website': 'https://www.odoo.com/documentation/16.0/applications/finance/fiscal_localizations/united_arab_emirates.html',
    'description': """
United Arab Emirates Accounting Module
=======================================================
United Arab Emirates accounting basic charts and localization.

Activates:

- Chart of Accounts
- Taxes
- Tax Report
- Fiscal Positions
    """,
    'depends': ['base', 'account'],
    'data': [
        'data/l10n_ae_data.xml',
        'data/l10n_ae_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_group_data.xml',
        'data/l10n_ae_chart_post_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/fiscal_templates_data.xml',
        'data/account_chart_template_data.xml',
        'views/report_invoice_templates.xml',
        'views/account_move.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,name,code,account_type,chart_template_id/id,reconcile
uae_account_100101,Right of use Asset (IFRS 16),100101,asset_fixed,l10n_ae.uae_chart_template_standard,False
uae_account_100102,Accumulated Depreciation right use asset (IFRS 16),100102,asset_fixed,l10n_ae.uae_chart_template_standard,False
uae_account_100103,VAT Receivable,100103,asset_non_current,l10n_ae.uae_chart_template_standard,False
uae_account_101005,Main Safe,101005,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_101006,Main Safe - Foreign Currency,101006,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_101007,Visa & Master Credit Cards,101007,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_101008,Gateway Credit Cards,101008,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_101009,Manual Visa & Master Cards,101009,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_101010,PayPal Account,101010,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_102011,Accounts Receivable,102011,asset_receivable,l10n_ae.uae_chart_template_standard,True
uae_account_102012,Accounts Receivable (PoS),102012,asset_receivable,l10n_ae.uae_chart_template_standard,True
uae_account_102013,Post Dated Cheques Received,102013,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_102014,Other Receivable,102014,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_102015,Other Debtors,102015,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_103016,Shipment Insurance,103016,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_103017,Shipments Documentation Charges,103017,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_103018,Shipment Other Charges,103018,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_103019,Handling Difference in Inventory,103019,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_103020,Items Delivered to Customs on temprary Base,103020,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104021,Prepaid Medical Insurance,104021,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104022,Prepaid Life Insurance,104022,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104023,Prepaid Office Rent,104023,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104024,Prepaid Other Insurance,104024,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104025,Prepaid License Fees,104025,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104026,Prepaid Maintenance,104026,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104027,Prepaid Site Hosting Fees,104027,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104028,Prepaid Employees Housing,104028,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104029,Prepaid Schooling Fees,104029,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104030,Prepaid Consultancy Fees,104030,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104031,Prepaid Legal Fees,104031,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104032,Prepaid Sponsorship Fees,104032,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104033,PrePaid Advertisement Expenses,104033,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104034,Prepaid Bank Guarantee,104034,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104035,Other Prepayments,104035,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104036,Prepaid Finance charge for Loans,104036,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104037,Deposit - Office Rent,104037,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104038,Deposits - Customs,104038,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104039,Deposit to Immigration (Visa),104039,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104040,Deposit Others,104040,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_104041,VAT Input,104041,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106001,Leasehold Improvement,106001,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106002,Furniture and Equipment,106002,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106003,Computer Hardware & Software,106003,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106004,Motor Vehicles,106004,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106005,Work In Progrees,106005,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106006,Amortisation on Leasehold Improvement,106006,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106007,Acc.Deprn.of Furniture & Office Equipment,106007,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106008,Acc. Deprn.Computer Hardware & Software,106008,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106009,Acc. Depreciation of Motor Vehicles,106009,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106010,Registration of Trademarks,106010,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_106011,Computer Card Renewal,106011,asset_current,l10n_ae.uae_chart_template_standard,False
uae_account_201002,Payables,201002,liability_payable,l10n_ae.uae_chart_template_standard,True
uae_account_201003,Credit Notes to Customers,201003,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201004,Accrued - Salaries,201004,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201005,Leave Tickets Provision,201005,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201006,Leave Days Provision,201006,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201007,Accrued - Commissions,201007,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201008,Accrued Salaries Increment,201008,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201009,Accrued-Staff Bonus,201009,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201010,Accrued Other Personnel Cost,201010,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201011,Accrued - Utilities,201011,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201012,Accrued - Telephone,201012,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201013,Accrued - Sponsorship,201013,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201014,Accrued - Audit Fees,201014,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201015,Accrued - Office Rent,201015,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201016,Accrued Others,201016,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201017,VAT Output,201017,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201018,Deferred income,201018,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_201019,Accrued Dubai Customs,201019,liability_current,l10n_ae.uae_chart_template_standard,False
uae_account_202001,End of Service Provision,202001,liability_non_current,l10n_ae.uae_chart_template_standard,False
uae_account_202002,Reservations,202002,liability_non_current,l10n_ae.uae_chart_template_standard,False
uae_account_202003,VAT Payable,202003,liability_non_current,l10n_ae.uae_chart_template_standard,False
uae_account_400001,Cost of Goods Sold in Trading,400001,expense_direct_cost,l10n_ae.uae_chart_template_standard,False
uae_account_400002,Cost Of Goods Sold I/C Sales,400002,expense_direct_cost,l10n_ae.uae_chart_template_standard,False
uae_account_400003,Basic Salary,400003,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400004,Housing Allowance,400004,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400005,Transportation Allowance,400005,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400006,Leave Ticket,400006,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400007,Leave Salary,400007,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400008,End Of Service Indemnity,400008,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400009,Medical Insurance,400009,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400010,Life Insurance,400010,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400011,Sales Commission,400011,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400012,Staff Other Allowances,400012,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400013,Uniform,400013,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400014,Visa Expenses,400014,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400015,Personnel Cost Others,400015,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400016,Office Rent,400016,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400017,Warehouse Rent,400017,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400018,Water & Electricity,400018,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400019,Other Utility Cahrges,400019,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400020,Telephone,400020,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400021,Courrier,400021,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400022,Web Site Hosting Fees,400022,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400023,Others - Communication,400023,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400024,Air tickets,400024,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400025,Hotel,400025,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400026,Meals,400026,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400027,Per Diem,400027,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400028,Others,400028,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400029,Audit Fees,400029,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400030,Sponsorship Fees,400030,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400031,Legal fees,400031,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400032,Trade License Fees,400032,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400033,Others - Professional Fees,400033,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400034,Other - Advertising Expenses,400034,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400035,Write Off Receivables & Payables,400035,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400036,Write Off Inventory,400036,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400037,Amortisation of Preoperating Expenses,400037,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400038,Cash Shortage,400038,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400039,Others - Provision & Write off,400039,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400040,Insurance,400040,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400041,Training,400041,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400042,Maintenance,400042,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400043,Security & Guard,400043,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400044,Cleaning,400044,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400045,Subscriptions,400045,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400046,Gifts & Donations,400046,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400047,Kitchen and Buffet Expenses,400047,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400048,Vehicle Expenses,400048,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400049,Convoyance Expenses,400049,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400050,Others - Office Various Expenses,400050,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400051,Other Bank Charges,400051,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400052,Loss On Fixed Assets Disposal,400052,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400053,Loss on Difference on Exchange,400053,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400054,Disposal of Business Branch,400054,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400055,Income Tax,400055,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400056,Previous Year Adjustments Account,400056,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400057,Other Non Operating Expenses,400057,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400058,Credit Card Charges,400058,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400059,Bank Finance & Loan Charges,400059,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400060,Air Miles Card Charges,400060,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400061,Credit Card Swipe Charges,400061,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400062,PayPal Charges,400062,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400063,Amortization on Leasehold Improvement,400063,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400064,Depreciation Of Furniture & Office Equipment,400064,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400065,Depreciation Of Computer Hard & Soft,400065,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400066,Depreciation Of Motor Vehicles,400066,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400067,Consultancy Fees,400067,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400068,Provision for Doubtful Debts,400068,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400069,Closing Account,400069,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400070,Depreciation on right of use asset (IFRS 16),400070,expense,l10n_ae.uae_chart_template_standard,False
uae_account_400071,Cash Discount Loss,400071,expense,l10n_ae.uae_chart_template_standard,False
uae_account_500001,Sales Account,500001,income,l10n_ae.uae_chart_template_standard,False
uae_account_500002,Sales of I/C,500002,income,l10n_ae.uae_chart_template_standard,False
uae_account_500003,Management Consultancy Fees,500003,income,l10n_ae.uae_chart_template_standard,False
uae_account_500004,Sales from Other Region,500004,income,l10n_ae.uae_chart_template_standard,False
uae_account_500005,Advertising Income,500005,income,l10n_ae.uae_chart_template_standard,False
uae_account_500006,Branding Income,500006,income,l10n_ae.uae_chart_template_standard,False
uae_account_500007,Space Rental Income,500007,income,l10n_ae.uae_chart_template_standard,False
uae_account_500008,Service Income,500008,income,l10n_ae.uae_chart_template_standard,False
uae_account_500009,Interest Revenue,500009,income,l10n_ae.uae_chart_template_standard,False
uae_account_500010,Capital Gain,500010,income,l10n_ae.uae_chart_template_standard,False
uae_account_500011,Gain On Difference Of Exchange,500011,income,l10n_ae.uae_chart_template_standard,False
uae_account_500012,Excess In Till,500012,income,l10n_ae.uae_chart_template_standard,False
uae_account_500013,Other Income,500013,income,l10n_ae.uae_chart_template_standard,False
uae_account_500014,Cash Discount Gain,500014,income_other,l10n_ae.uae_chart_template_standard,False
uae_account_999999,Undistributed Profits/Losses,999999,equity_unaffected,l10n_ae.uae_chart_template_standard,False

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ae.uae_chart_template_standard')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <!-- Account Tax Group -->
        <record id="ae_tax_group_5" model="account.tax.group">
            <field name="country_id" ref="base.ae"/>
            <field name="name">VAT 5%</field>
        </record>
        <record id="ae_tax_group_0" model="account.tax.group">
            <field name="country_id" ref="base.ae"/>
            <field name="name">VAT 0%</field>
        </record>
        <record id="ae_tax_group_exempted" model="account.tax.group">
            <field name="country_id" ref="base.ae"/>
            <field name="name">VAT Exempted</field>
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
        <field name="country_id" ref="base.ae"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_base_all_sales" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Base)</field>
                <field name="aggregation_formula">1_STANDARD_RATED_SUPPLIES_BASE.balance + TAX_REF_TOUR_SCHEME_BASE.balance + REVERSE_CHARGE_PRO_BASE.balance + ZERO_RATE_SUPP_BASE.balance + EXAMPT_SUPP_BASE.balance + OUT_OF_SCOPE_BASE_0.balance + GOODS_IMPORT_IN_UAE_BASE.balance + ADJUST_GOODS_IMPORT_IN_UAE_BASE.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_supplies_base" model="account.report.line">
                        <field name="name">1. Standard Rated supplies (Base)</field>
                        <field name="code">1_STANDARD_RATED_SUPPLIES_BASE</field>
                        <field name="aggregation_formula">STD_RATE_SUPP_BASE_AB.balance + STD_RATE_SUPP_BASE_DB.balance + STD_RATE_SUPP_BASE_SJ.balance + STD_RATE_SUPP_BASE_AJ.balance + STD_RATE_SUPP_BASE_UM.balance + STD_RATE_SUPP_BASE_RA.balance + STD_RATE_SUPP_BASE_FU.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_standard_rated_supplies_base_abu_dhabi" model="account.report.line">
                                <field name="name">a. Abu Dhabi</field>
                                <field name="code">STD_RATE_SUPP_BASE_AB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_abu_dhabi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">a. Abu Dhabi (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_dubai" model="account.report.line">
                                <field name="name">b. Dubai</field>
                                <field name="code">STD_RATE_SUPP_BASE_DB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_dubai_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">b. Dubai (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_sharjah" model="account.report.line">
                                <field name="name">c. Sharjah</field>
                                <field name="code">STD_RATE_SUPP_BASE_SJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_sharjah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">c. Sharjah (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_ajman" model="account.report.line">
                                <field name="name">d. Ajman</field>
                                <field name="code">STD_RATE_SUPP_BASE_AJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_ajman_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">d. Ajman (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_umm_al_quwain" model="account.report.line">
                                <field name="name">e. Umm Al Quwain</field>
                                <field name="code">STD_RATE_SUPP_BASE_UM</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_umm_al_quwain_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">e. Umm Al Quwain (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_ras_al_khaima" model="account.report.line">
                                <field name="name">f. Ras Al-Khaima</field>
                                <field name="code">STD_RATE_SUPP_BASE_RA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_ras_al_khaima_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">f. Ras Al-Khaima (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_fujairah" model="account.report.line">
                                <field name="name">g. Fujairah</field>
                                <field name="code">STD_RATE_SUPP_BASE_FU</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_fujairah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">g. Fujairah (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_subtotal" model="account.report.line">
                                <field name="name">Sub Total</field>
                                <field name="aggregation_formula">STD_RATE_SUPP_BASE_AB.balance + STD_RATE_SUPP_BASE_DB.balance + STD_RATE_SUPP_BASE_SJ.balance + STD_RATE_SUPP_BASE_AJ.balance + STD_RATE_SUPP_BASE_UM.balance + STD_RATE_SUPP_BASE_RA.balance + STD_RATE_SUPP_BASE_FU.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_tax_refund_tourist_base" model="account.report.line">
                        <field name="name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme</field>
                        <field name="code">TAX_REF_TOUR_SCHEME_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_tax_refund_tourist_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_reverse_charge_base" model="account.report.line">
                        <field name="name">3. Supplies subject to reverse charge provisions</field>
                        <field name="code">REVERSE_CHARGE_PRO_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_reverse_charge_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Supplies subject to reverse charge provisions (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_zero_rated_supplies_base" model="account.report.line">
                        <field name="name">4. Zero rated supplies</field>
                        <field name="code">ZERO_RATE_SUPP_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_zero_rated_supplies_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4. Zero rated supplies (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_supplies_base" model="account.report.line">
                        <field name="name">5. Exempt supplies</field>
                        <field name="code">EXAMPT_SUPP_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_supplies_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. Exempt supplies (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_out_of_scope_base" model="account.report.line">
                        <field name="name">6. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_BASE_0</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_out_of_scope_base_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_import_uae_base" model="account.report.line">
                        <field name="name">7. Goods imported into the UAE</field>
                        <field name="code">GOODS_IMPORT_IN_UAE_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_import_uae_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Goods imported into the UAE (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_adjustment_import_uae_base" model="account.report.line">
                        <field name="name">8. Adjustments to goods imported into the UAE</field>
                        <field name="code">ADJUST_GOODS_IMPORT_IN_UAE_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_adjustment_import_uae_base_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_base_all_sales_total" model="account.report.line">
                        <field name="name">9. Total</field>
                        <field name="aggregation_formula">ADJUST_GOODS_IMPORT_IN_UAE_BASE.balance + GOODS_IMPORT_IN_UAE_BASE.balance + OUT_OF_SCOPE_BASE_0.balance + EXAMPT_SUPP_BASE.balance + ZERO_RATE_SUPP_BASE.balance + REVERSE_CHARGE_PRO_BASE.balance + TAX_REF_TOUR_SCHEME_BASE.balance + (STD_RATE_SUPP_BASE_AB.balance + STD_RATE_SUPP_BASE_DB.balance + STD_RATE_SUPP_BASE_SJ.balance + STD_RATE_SUPP_BASE_AJ.balance + STD_RATE_SUPP_BASE_UM.balance + STD_RATE_SUPP_BASE_RA.balance + STD_RATE_SUPP_BASE_FU.balance)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_base_all_expense" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Base)</field>
                <field name="aggregation_formula">STD_RATE_EXPENSES_BASE.balance + SUPP_REV_CHARGE_PRO_BASE.balance + OUT_OF_SCOPE_1_BASE.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_expense_base" model="account.report.line">
                        <field name="name">10. Standard rated expenses</field>
                        <field name="code">STD_RATE_EXPENSES_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_expense_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10. Standard rated expenses (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_supplies_reverse_base" model="account.report.line">
                        <field name="name">11. Supplies subject to the reverse charge provisions</field>
                        <field name="code">SUPP_REV_CHARGE_PRO_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_supplies_reverse_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11. Supplies subject to the reverse charge provisions (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_out_of_scope" model="account.report.line">
                        <field name="name">12. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_1_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_out_of_scope_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_base_all_expense_total" model="account.report.line">
                        <field name="name">13. Totals</field>
                        <field name="aggregation_formula">OUT_OF_SCOPE_1_BASE.balance + SUPP_REV_CHARGE_PRO_BASE.balance + STD_RATE_EXPENSES_BASE.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_all_sales" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Tax)</field>
                <field name="aggregation_formula">1_STANDARD_RATED_SUPPLIES_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + ZERO_RATE_SUPP_TAX.balance + EXAMPT_SUPP_TAX.balance + OUT_OF_SCOPE_TAX_0.balance + GOODS_IMPORT_IN_UAE_TAX.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_supplies_vat" model="account.report.line">
                        <field name="name">1. Standard Rated supplies (Tax)</field>
                        <field name="code">1_STANDARD_RATED_SUPPLIES_TAX</field>
                        <field name="aggregation_formula">STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_standard_rated_supplies_vat_abu_dhabi" model="account.report.line">
                                <field name="name">a. Abu Dhabi</field>
                                <field name="code">STD_RATE_SUPP_TAX_AB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_abu_dhabi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">a. Abu Dhabi (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_dubai" model="account.report.line">
                                <field name="name">b. Dubai</field>
                                <field name="code">STD_RATE_SUPP_TAX_DB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_dubai_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">b. Dubai (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_sharjah" model="account.report.line">
                                <field name="name">c. Sharjah</field>
                                <field name="code">STD_RATE_SUPP_TAX_SJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_sharjah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">c. Sharjah (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_ajman" model="account.report.line">
                                <field name="name">d. Ajman</field>
                                <field name="code">STD_RATE_SUPP_TAX_AJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_ajman_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">d. Ajman (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_umm_al_quwain" model="account.report.line">
                                <field name="name">e. Umm Al Quwain</field>
                                <field name="code">STD_RATE_SUPP_TAX_UM</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_umm_al_quwain_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">e. Umm Al Quwain (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_ras_al_khaima" model="account.report.line">
                                <field name="name">f. Ras Al-Khaima</field>
                                <field name="code">STD_RATE_SUPP_TAX_RA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_ras_al_khaima_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">f. Ras Al-Khaima (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_fujairah" model="account.report.line">
                                <field name="name">g. Fujairah</field>
                                <field name="code">STD_RATE_SUPP_TAX_FU</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_fujairah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">g. Fujairah (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_subtotal" model="account.report.line">
                                <field name="name">Sub Total</field>
                                <field name="aggregation_formula">STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_tax_refund_tourist_vat" model="account.report.line">
                        <field name="name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme</field>
                        <field name="code">TAX_REF_TOUR_SCHEME_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_tax_refund_tourist_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_reverse_charge_vat" model="account.report.line">
                        <field name="name">3. Supplies subject to reverse charge provisions</field>
                        <field name="code">REVERSE_CHARGE_PRO_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_reverse_charge_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Supplies subject to reverse charge provisions (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_zero_rated_supplies_vat" model="account.report.line">
                        <field name="name">4. Zero rated supplies</field>
                        <field name="code">ZERO_RATE_SUPP_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_zero_rated_supplies_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4. Zero rated supplies (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_supplies_vat" model="account.report.line">
                        <field name="name">5. Exempt supplies</field>
                        <field name="code">EXAMPT_SUPP_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_supplies_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. Exempt supplies (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_out_of_scope_vat" model="account.report.line">
                        <field name="name">6. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_TAX_0</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_out_of_scope_vat_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_import_uae_vat" model="account.report.line">
                        <field name="name">7. Goods imported into the UAE</field>
                        <field name="code">GOODS_IMPORT_IN_UAE_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_import_uae_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Goods imported into the UAE (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_adjustment_import_uae_vat" model="account.report.line">
                        <field name="name">8. Adjustments to goods imported into the UAE</field>
                        <field name="code">ADJUST_GOODS_IMPORT_IN_UAE_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_adjustment_import_uae_vat_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vat_all_sales_total" model="account.report.line">
                        <field name="name">9. Total</field>
                        <field name="aggregation_formula">(STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_all_expense" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Tax)</field>
                <field name="aggregation_formula">STD_RATE_EXPENSES_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + OUT_OF_SCOPE_1_TAX.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_expense_vat" model="account.report.line">
                        <field name="name">10. Standard rated expenses</field>
                        <field name="code">STD_RATE_EXPENSES_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_expense_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10. Standard rated expenses (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_supplies_reverse_vat" model="account.report.line">
                        <field name="name">11. Supplies subject to the reverse charge provisions</field>
                        <field name="code">SUPP_REV_CHARGE_PRO_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_supplies_reverse_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11. Supplies subject to the reverse charge provisions (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_out_of_scope_vat" model="account.report.line">
                        <field name="name">12. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_1_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_out_of_scope_vat_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vat_all_expense_total" model="account.report.line">
                        <field name="name">13. Totals</field>
                        <field name="aggregation_formula">OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_net_vat_due" model="account.report.line">
                <field name="name">Net VAT Due</field>
                <field name="aggregation_formula">((STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance) - (OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance)</field>
                <field name="children_ids">
                    <record id="tax_report_line_total_value_due_tax_period" model="account.report.line">
                        <field name="name">14. Total value of due tax for the period</field>
                        <field name="aggregation_formula">(STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance</field>
                    </record>
                    <record id="tax_report_line_total_value_recoverable_tax_period" model="account.report.line">
                        <field name="name">15. Total value of recoverable tax for the period</field>
                        <field name="aggregation_formula">OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance</field>
                    </record>
                    <record id="tax_report_line_net_vat_due_period" model="account.report.line">
                        <field name="name">16. Net VAT due (or reclaimed) for the period</field>
                        <field name="aggregation_formula">((STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance) - (OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance)</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="uae_sale_tax_5_dubai" model="account.tax.template">
        <field name="name">VAT 5% (Dubai)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_dubai_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_dubai_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_dubai_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_dubai_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_5_abu_dhabi" model="account.tax.template">
        <field name="name">VAT 5% (Abu Dhabi)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_abu_dhabi_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_abu_dhabi_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_abu_dhabi_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_abu_dhabi_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_5_sharjah" model="account.tax.template">
        <field name="name">VAT 5% (Sharjah)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_sharjah_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_sharjah_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_sharjah_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_sharjah_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_5_ajman" model="account.tax.template">
        <field name="name">VAT 5% (Ajman)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_ajman_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_ajman_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_ajman_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_ajman_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_5_umm_al_quwain" model="account.tax.template">
        <field name="name">VAT 5% (Umm Al Quwain)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_umm_al_quwain_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_umm_al_quwain_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_umm_al_quwain_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_umm_al_quwain_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_5_ras_al_khaima" model="account.tax.template">
        <field name="name">VAT 5% (Ras Al-Khaima)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_ras_al_khaima_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_ras_al_khaima_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_ras_al_khaima_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_ras_al_khaima_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_5_fujairah" model="account.tax.template">
        <field name="name">VAT 5% (Fujairah)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_fujairah_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_fujairah_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_base_fujairah_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_supplies_vat_fujairah_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_exempted" model="account.tax.template">
        <field name="name">Exempted Tax</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Exempted</field>
        <field name="tax_group_id" ref="ae_tax_group_exempted"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_exempt_supplies_base_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_exempt_supplies_base_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="uae_sale_tax_0" model="account.tax.template">
        <field name="name">VAT 0%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 0%</field>
        <field name="tax_group_id" ref="ae_tax_group_0"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_zero_rated_supplies_base_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_zero_rated_supplies_base_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="uae_export_tax" model="account.tax.template">
        <field name="name">Export Tax 0%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Export Tax</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="uae_sale_tax_reverse_charge_dubai" model="account.tax.template">
        <field name="name">Reverse Charge Provision (Dubai)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_reverse_charge_abu_dhabi" model="account.tax.template">
        <field name="name">Reverse Charge Provision (Abu Dhabi)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_reverse_charge_sharjah" model="account.tax.template">
        <field name="name">Reverse Charge Provision (Sharjah)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_reverse_charge_ajman" model="account.tax.template">
        <field name="name">Reverse Charge Provision (Ajman)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_reverse_charge_umm_al_quwain" model="account.tax.template">
        <field name="name">Reverse Charge Provision (Umm Al Quwain)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_reverse_charge_ras_al_khaima" model="account.tax.template">
        <field name="name">Reverse Charge Provision (Ras Al-Khaima)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_reverse_charge_fujairah" model="account.tax.template">
        <field name="name">Reverse Charge Provision (Fujairah)</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_sale_tax_tourist_refund" model="account.tax.template">
        <field name="name">Tourist Refund scheme 5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5.0</field>
        <field name="amount_type">percent</field>
        <field name="description">Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_tax_refund_tourist_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_tax_refund_tourist_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_tax_refund_tourist_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_tax_refund_tourist_vat_tag')],
            }),
        ]"/>
    </record>
    <!-- purchase taxes -->
    <record id="uae_purchase_tax_5" model="account.tax.template">
        <field name="name">VAT 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 5%</field>
        <field name="tax_group_id" ref="ae_tax_group_5"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_expense_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_standard_rated_expense_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_expense_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_standard_rated_expense_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_purchase_tax_exempted" model="account.tax.template">
        <field name="name">Exempted Tax</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Exempted Tax</field>
        <field name="tax_group_id" ref="ae_tax_group_exempted"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="uae_purchase_tax_0" model="account.tax.template">
        <field name="name">VAT 0%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">VAT 0%</field>
        <field name="tax_group_id" ref="ae_tax_group_0"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="uae_import_tax" model="account.tax.template">
        <field name="name">Import Tax 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Import Tax</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_import_uae_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_import_uae_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_import_uae_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_import_uae_vat_tag')],
            }),
        ]"/>
    </record>
    <record id="uae_purchase_tax_reverse_charge" model="account.tax.template">
        <field name="name">Reverse Charge Provision</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">Supplies subject to reverse charge provisions</field>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'plus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'minus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_base_tag'), ref('tax_report_line_supplies_reverse_charge_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('uae_account_104041'),
                'minus_report_expression_ids': [ref('tax_report_line_expense_supplies_reverse_vat_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_201017'),
                'plus_report_expression_ids': [ref('tax_report_line_supplies_reverse_charge_vat_tag')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\fiscal_templates_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_fiscal_position_dubai" model="account.fiscal.position.template">
        <field name="name">Dubai</field>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">16</field>
        <field name="country_id" ref="base.ae"/>
        <field name="state_ids" eval="[(6,0,[ref('base.state_ae_du')])]"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>

    <record id="account_fiscal_position_abu_dhabi" model="account.fiscal.position.template">
        <field name="name">Abu Dhabi</field>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">16</field>
        <field name="country_id" ref="base.ae"/>
        <field name="state_ids" eval="[(6,0,[ref('base.state_ae_az')])]"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>
    <record id="account_fiscal_position_abu_dhabi_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_sale_tax_5_dubai"/>
        <field name="tax_dest_id" ref="uae_sale_tax_5_abu_dhabi"/>
        <field name="position_id" ref="account_fiscal_position_abu_dhabi"/>
    </record>

    <record id="account_fiscal_position_sharjah" model="account.fiscal.position.template">
        <field name="name">Sharjah</field>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">16</field>
        <field name="country_id" ref="base.ae"/>
        <field name="state_ids" eval="[(6,0,[ref('base.state_ae_sh')])]"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>
    <record id="account_fiscal_position_sharjah_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_sale_tax_5_dubai"/>
        <field name="tax_dest_id" ref="uae_sale_tax_5_sharjah"/>
        <field name="position_id" ref="account_fiscal_position_sharjah"/>
    </record>

    <record id="account_fiscal_position_ajman" model="account.fiscal.position.template">
        <field name="name">Ajman</field>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">16</field>
        <field name="country_id" ref="base.ae"/>
        <field name="state_ids" eval="[(6,0,[ref('base.state_ae_aj')])]"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>
    <record id="account_fiscal_position_ajman_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_sale_tax_5_dubai"/>
        <field name="tax_dest_id" ref="uae_sale_tax_5_ajman"/>
        <field name="position_id" ref="account_fiscal_position_ajman"/>
    </record>

    <record id="account_fiscal_position_umm_al_quwain" model="account.fiscal.position.template">
        <field name="name">Umm Al Quwain</field>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">16</field>
        <field name="country_id" ref="base.ae"/>
        <field name="state_ids" eval="[(6,0,[ref('base.state_ae_uq')])]"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>
    <record id="account_fiscal_position_umm_al_quwain_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_sale_tax_5_dubai"/>
        <field name="tax_dest_id" ref="uae_sale_tax_5_umm_al_quwain"/>
        <field name="position_id" ref="account_fiscal_position_umm_al_quwain"/>
    </record>

    <record id="account_fiscal_position_ras_al_khaima" model="account.fiscal.position.template">
        <field name="name">Ras Al-Khaima</field>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">16</field>
        <field name="country_id" ref="base.ae"/>
        <field name="state_ids" eval="[(6,0,[ref('base.state_ae_rk')])]"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>
    <record id="account_fiscal_position_ras_al_khaima_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_sale_tax_5_dubai"/>
        <field name="tax_dest_id" ref="uae_sale_tax_5_ras_al_khaima"/>
        <field name="position_id" ref="account_fiscal_position_ras_al_khaima"/>
    </record>

    <record id="account_fiscal_position_fujairah" model="account.fiscal.position.template">
        <field name="name">Fujairah</field>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">16</field>
        <field name="country_id" ref="base.ae"/>
        <field name="state_ids" eval="[(6,0,[ref('base.state_ae_fu')])]"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>
    <record id="account_fiscal_position_fujairah_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_sale_tax_5_dubai"/>
        <field name="tax_dest_id" ref="uae_sale_tax_5_fujairah"/>
        <field name="position_id" ref="account_fiscal_position_fujairah"/>
    </record>

    <record id="account_fiscal_position_non_uae_countries" model="account.fiscal.position.template">
        <field name="name">Non-UAE</field>
        <field name="sequence">20</field>
        <field name="auto_apply" eval="True"/>
        <field name="chart_template_id" ref="uae_chart_template_standard"/>
    </record>
    <record id="acccount_fiscal_position_tax_non_uae_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_sale_tax_5_dubai"/>
        <field name="tax_dest_id" ref="uae_sale_tax_0"/>
        <field name="position_id" ref="account_fiscal_position_non_uae_countries"/>
    </record>
    <record id="acccount_fiscal_position_tax_non_uae_02" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="uae_purchase_tax_5"/>
        <field name="tax_dest_id" ref="uae_purchase_tax_reverse_charge"/>
        <field name="position_id" ref="account_fiscal_position_non_uae_countries"/>
    </record>
</odoo>

```

## File: data\l10n_ae_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <record id="uae_chart_template_standard" model="account.chart.template">
         <field name="name">U.A.E Chart of Accounts - Standard</field>
         <field name="code_digits">6</field>
         <field name="bank_account_code_prefix">101</field>
         <field name="cash_account_code_prefix">105</field>
         <field name="transfer_account_code_prefix">100</field>
         <field name="currency_id" ref="base.AED" />
         <field name="country_id" ref="base.ae"/>
     </record>
</odoo>

```

## File: data\l10n_ae_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="uae_chart_template_standard" model="account.chart.template">
        <field name="property_account_receivable_id" ref="uae_account_102011"/>
        <field name="property_account_payable_id" ref="uae_account_201002"/>
        <field name="property_account_expense_categ_id" ref="uae_account_400001"/>
        <field name="property_account_income_categ_id" ref="uae_account_500001"/>
        <field name="expense_currency_exchange_account_id" ref="uae_account_400053"/>
        <field name="income_currency_exchange_account_id" ref="uae_account_500011"/>
        <field name="default_pos_receivable_account_id" ref="uae_account_102012"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="uae_account_400071"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="uae_account_500014"/>
        <field name="property_tax_payable_account_id" ref="uae_account_202003"/>
        <field name="property_tax_receivable_account_id" ref="uae_account_100103"/>
    </record>
</odoo>

```

## File: data\l10n_ae_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- set VAT label to show on invoice report -->
    <record id="base.ae" model="res.country">
        <field name="vat_label">VAT</field>
    </record>
    <record id="base.AED" model="res.currency">
        <field name="symbol">AED</field>
    </record>
    <record id="gcc_countries_group" model="res.country.group">
        <field name="name">GCC VAT implementing States</field>
        <field name="country_ids" eval="[(6,0,[ref('base.ae'),ref('base.sa'),ref('base.bh')])]"/>
    </record>
</odoo>

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        """ If UAE chart, we add 2 new journals TA and IFRS"""
        if self == self.env.ref('l10n_ae.uae_chart_template_standard'):
            if not journals_dict:
                journals_dict = []
            journals_dict.extend(
                [{"name": "Tax Adjustments", "company_id": company.id, "code": "TA", "type": "general", "sequence": 1,
                  "favorite": True},
                 {"name": "IFRS 16", "company_id": company.id, "code": "IFRS", "type": "general", "favorite": True,
                  "sequence": 10}])
        return super()._prepare_all_journals(acc_template_ref, company, journals_dict=journals_dict)

    def _load_template(self, company, code_digits=None, account_ref=None, taxes_ref=None):
        account_ref, taxes_ref = super(AccountChartTemplate, self)._load_template(company=company,
                                                                                  code_digits=code_digits,
                                                                                  account_ref=account_ref,
                                                                                  taxes_ref=taxes_ref)
        if self == self.env.ref('l10n_ae.uae_chart_template_standard'):
            ifrs_journal = self.env['account.journal'].search(
                [('company_id', '=', company.id), ('code', '=', 'IFRS')]).id
            if ifrs_journal:
                ifrs_account_ids = [self.env.ref('l10n_ae.uae_account_100101').id,
                                    self.env.ref('l10n_ae.uae_account_100102').id,
                                    self.env.ref('l10n_ae.uae_account_400070').id]
                ifrs_accounts = self.env['account.account'].browse([account_ref.get(id) for id in ifrs_account_ids])
                for account in ifrs_accounts:
                    account.allowed_journal_ids = [(4, ifrs_journal, 0)]
            self.env.ref('l10n_ae.ae_tax_group_5').write(
                {'property_tax_payable_account_id': account_ref.get(self.env.ref('l10n_ae.uae_account_202003').id),
                 'property_tax_receivable_account_id': account_ref.get(self.env.ref('l10n_ae.uae_account_100103').id)})
        return account_ref, taxes_ref

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    l10n_ae_vat_amount = fields.Monetary(compute='_compute_vat_amount', string='VAT Amount')

    @api.depends('price_subtotal', 'price_total')
    def _compute_vat_amount(self):
        for record in self:
            record.l10n_ae_vat_amount = record.price_total - record.price_subtotal

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move_line
from . import account_chart_template

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="-3.03" y="4.82" width="67.93" height="36.8" maskUnits="userSpaceOnUse">
      <rect x="4.08" y="7.61" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
    <use width="106" height="106" xlink:href="#c"/>
    <rect x="4" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="1024" height="512" transform="translate(-3.03 4.82) scale(0.07 0.07)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABAAAAAIrCAYAAABxputMAAAACXBIWXMAAKbcAACm3AFbmodlAAAN6ElEQVR4Xu3bMXEEQRAEwTnF83mREIRDIoTH4BmtIKytrUy73XEqYq41swZIun7fuwkAAHCG52u3AAAAAP4/AQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIeO0GwLnu989uAgAAHOJaM2s3Ag61nD8AAEQ8XgAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAl67AXCu+753EwAA4BDXmlm7EXCmazcAAABO8XgBAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAgQAAAAACBAAAAAAIAAAQAAAAACBAAAAAAIEAAAAAAgQAAAAACAAAEAAAAAAgQAAAAACBAAAAAAIEAAAAAAgAABAAAAAAIEAAAAAAh4zcyzGwEAR/nMzPduBAAc5fMHkZATkvPI2gwAAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

## File: views\account_move.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record id="view_move_form" model="ir.ui.view">
            <field name="name">l10n_ae.account.move.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//tree//field[@name='tax_ids']" position="after">
                    <field name="l10n_ae_vat_amount" optional="show"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\report_invoice_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@name='payment_term']" position="after">
            <p t-if="o.company_id.country_id.code == 'AE' and o.partner_id.country_id.code != 'AE' and o.env.ref('l10n_ae.gcc_countries_group') in o.partner_id.country_id.country_group_ids">
                Supply between <b>United Arab Emirates</b> and
                <b>
                    <span t-field="o.partner_id.country_id.name"/>
                </b>
            </p>
        </xpath>
        <xpath expr="//h2/span" position="before">
            <span t-if="o.company_id.country_id.code == 'AE' and o.move_type in ['out_invoice', 'out_refund']">TAX
            </span>
        </xpath>

        <xpath expr="//thead//th[@name='th_taxes']" position="replace">
            <th name="th_taxes"
                t-attf-class="text-start {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                <span t-if="o.company_id.country_id.code == 'AE'">VAT</span>
                <span t-else="">Taxes</span>
            </th>
            <th t-if="o.company_id.country_id.code == 'AE'" name="tax_amount"
                t-attf-class="text-start {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                <span>VAT Amount</span>
            </th>
        </xpath>

        <xpath expr="//span[@id='line_tax_ids']/.." position="after">
            <td t-if="o.company_id.country_id.code == 'AE'">
                <span t-field="line.l10n_ae_vat_amount" id="line_tax_amount"/>
            </td>
        </xpath>

        <xpath expr="//div[hasclass('clearfix')]" position="after">
            <div t-if="o.company_id.country_id.code == 'AE' and o.currency_id != o.company_id.currency_id"
                 id="aed_amounts" class="row clearfix ms-auto my-3 text-nowrap table">
                <t t-set="aed_rate"
                   t-value="o.env['res.currency']._get_conversion_rate(o.currency_id, o.company_id.currency_id, o.company_id, o.invoice_date or datetime.date.today())"/>
                <div name="exchange_rate" class="col-auto">
                    <strong>Exchange Rate</strong>
                    <p class="m-0" t-esc="aed_rate" t-options='{"widget": "float", "precision": 5}'/>
                </div>
                <div name="aed_subtotal" class="col-auto">
                    <strong>Subtotal (AED)</strong>
                    <p class="m-0" t-esc="o.amount_untaxed_signed"
                       t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/>
                </div>
                <div name="aed_vat_amount" class="col-auto">
                    <strong>VAT Amount (AED)</strong>
                    <p class="m-0"
                       t-esc="o.currency_id._convert(o.amount_tax, o.company_id.currency_id, o.company_id, o.invoice_date or datetime.date.today())"
                       t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/>
                </div>
                <div name="aed_total" class="col-auto">
                    <strong>Total (AED)</strong>
                    <p class="m-0" t-esc="o.amount_total_signed"
                       t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/>
                </div>
            </div>
        </xpath>
    </template>
</odoo>

```

