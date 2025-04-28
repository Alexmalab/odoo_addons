# Odoo Module: l10n_ae

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 Tech Receptives (<http://techreceptives.com>).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 Tech Receptives (<http://techreceptives.com>)

{
    'name': 'U.A.E. - Accounting',
    'author': 'Tech Receptives',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
United Arab Emirates accounting chart and localization.
=======================================================

    """,
    'depends': ['base', 'account'],
    'data': [
             'data/l10n_ae_data.xml',
             'data/account_data.xml',
             'data/l10n_ae_chart_data.xml',
             'data/account.account.template.csv',
             'data/l10n_ae_chart_post_data.xml',
             'data/account_tax_report_data.xml',
             'data/account_tax_template_data.xml',
             'data/fiscal_templates_data.xml',
             'data/account_chart_template_data.xml',
             'views/report_invoice_templates.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"uae_account_3665","Main Safe","101111","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3666","Main Safe - Foreign Currency","101112","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3677","Visa & Master Credit Cards","101311","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3678","Gateway Credit Cards","101312","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3679","Manual Visa & Master Cards","101313","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3680","PayPal Account","101314","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3681","Accounts Receivable","102","account.data_account_type_receivable","l10n_ae.uae_chart_template_standard","True"
"uae_account_3682","Accounts Receivable (PoS)","1021","account.data_account_type_receivable","l10n_ae.uae_chart_template_standard","True"
"uae_account_3684","Trade in Opening Fees","102112","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3685","Post Dated Cheques Received","102113","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3686","Corporate Credit Cards","102114","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3687","Other Receivable","1022","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3688","Other Debtors","10221","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3689","Accrued Rebates Due from Suppliers","102220","account.data_account_type_receivable","l10n_ae.uae_chart_template_standard","True"
"uae_account_3690","Accured Income from Suppliers","102221","account.data_account_type_receivable","l10n_ae.uae_chart_template_standard","True"
"uae_account_3693","Supplier Cost","103110","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3694","Customs Duty","103111","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3695","Freight and Transport","103112","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3696","Shipment Insurance","103113","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3697","Shipments Documentation Charges","103114","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3698","Shipment Other Charges","103115","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3700","Handling Difference in Inventory","103210","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3701","Items Delivered to Customs on temprary Base","103211","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3704","Prepaid Medical Insurance","104110","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3705","Prepaid Life Insurance","104111","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3706","Prepaid Office Rent","104112","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3707","Prepaid Other Insurance","104113","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3708","Prepaid License Fees","104114","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3709","Prepaid Maintenance","104115","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3710","Prepaid Site Hosting Fees","104116","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3711","Prepaid Employees Housing","104117","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3712","Prepaid Schooling Fees","104118","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3713","Prepaid Consultancy Fees","104119","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3714","Prepaid Legal Fees","104120","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3715","Prepaid Sponsorship Fees","104121","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3716","PrePaid Advertisement Expenses","104122","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3717","Prepaid Bank Guarantee","104123","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3718","Other Prepayments","104124","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3719","Prepaid Finance charge for Loans","104125","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3721","Deposit - Office Rent","104201","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3722","Deposits - Customs","104202","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3723","Deposit to Immigration (Visa)","104203","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3724","Deposit Others","104204","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3726","Taxes Paid","104301","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3727","Withholding Tax Receivables","104302","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3734","Leasehold Improvement","152101","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3735","Furniture and Equipment","152102","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3736","Computer Hardware & Software","152103","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3737","Motor Vehicules","152104","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3738","Work In Progrees","152105","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3740","Amortisation on Leasehold Improvement","152201","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3741","Acc.Deprn.of Furniture & Office Equipment","152202","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3742","Acc. Deprn.Computer Hardware & Software","152203","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3743","Acc. Depreciation of Motor Vehicles","152204","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3747","Registration of Trademarks","154001","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3748","Computer Card Renewal","154002","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3749","Dispoal of Outlets","154003","account.data_account_type_current_assets","l10n_ae.uae_chart_template_standard","False"
"uae_account_3753","Payables","2021","account.data_account_type_payable","l10n_ae.uae_chart_template_standard","True"
"uae_account_3763","Reservations","203101","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3764","Credit Notes to Customers","203102","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3768","Accrued - Salaries","204111","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3769","Accrued - Leave Tickets","204112","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3770","Accrued - Leave Salary","204113","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3771","Accrued - Commissions","204114","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3772","Accrued Salaries Increment","204115","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3773","Accrued-Staff Bonus","204116","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3774","Accrued Other Personnel Cost","204120","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3777","Accrued - Utilities","204211","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3778","Accrued - Telephone","204212","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3779","Accrued - Sponsorship","204213","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3780","Accrued - Audit Fees","204214","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3781","Accrued - Office Rent","204215","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3782","Accrued Others","204220","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3785","Taxes Received","204311","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3786","Withholding Tax Payable","204312","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3787","Income Tax Payable","204313","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3789","Deferred income","204410","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3790","Accrued Dubai Customs","204411","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3791","Shipping & Handling","204412","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3798","Share Capital","310001","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3799","Contributed Capital","310002","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3800","Treasury Stocks","310003","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3801","Shareholders Current A/c","310004","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3802","Sub Ordinated Loan","310005","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3804","Previous Years Results","320001","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3805","Current Year Results","320002","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3806","Dividends Paid","320003","account.data_account_type_current_liabilities","l10n_ae.uae_chart_template_standard","False"
"uae_account_3810","Sales Account","401100","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3811","Sales of I/C","401101","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3812","Management Consultancy Fees","401102","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3814","Sales from Other Region","402100","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3818","Advertising Income","411101","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3819","Branding Income","411102","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3820","Space Rental Income","411103","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3821","Early Setmt Margin from Suppliers","411104","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3822","Service Income","411105","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3823","Rebate from Suppliers","411106","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3824","Marketing Rebate from Suppliers","411107","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3827","Interest Revenue","421101","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3828","Interest from FD","421102","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3829","Trade Opening Fees from suppliers","421103","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3830","Products Listing Fees from Suppliers","421104","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3832","Capital Gain","422101","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3833","Gain On Difference Of Exchange","422102","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3834","Excess In Till","422103","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3835","Other Income","422104","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3836","Management Consultancy Fees","422105","account.data_account_type_revenue","l10n_ae.uae_chart_template_standard","False"
"uae_account_3840","Cost of Goods Sold in Trading","5011","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3841","Cost Of Goods Sold I/C Sales","5012","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3843","Share Resource Expenses Account","511001","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3847","Basic Salary","521001","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3848","Housing Allowance","521002","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3849","Transportation Allowance","521003","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3850","Leave Ticket","521004","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3851","Leave Salary","521005","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3852","End Of Service Indemnity","521006","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3853","Medical Insurance","521007","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3854","Life Insurance","521008","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3855","Sales Commission","521009","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3856","Staff School Allowances","521010","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3857","Uniform","521011","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3858","Visa Expenses","521012","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3859","Personnel Cost Others","521013","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3861","Office Rent","521101","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3862","Warehouse Rent","521102","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3864","Water & Electricity","521201","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3865","Other Utility Cahrges","521202","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3867","Telephone","521301","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3868","Courrier","521302","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3869","Web Site Hosting Fees","521303","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3870","Others - Communication","521304","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3872","Air tickets","521401","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3873","Hotel","521402","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3874","Meals","521403","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3875","Per Diem","521404","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3876","Others","521405","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3878","Audit Fees","521601","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3879","Sponsorship Fees","521602","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3880","Legal fees","521603","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3881","Trade License Fees","521604","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3882","Others - Professional Fees","521606","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3884","Other - Advertising Expenses","521701","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3886","Write Off Receivables & Payables","521801","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3887","Write Off Inventory","521802","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3888","Amortisation of Preoperating Expenses","521803","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3889","Cash Shortage","521804","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3890","Others - Provision & Write off","521805","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3892","Insurance","521901","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3893","Training","521902","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3894","Maintenance","521903","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3895","Security & Guard","521904","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3896","Cleaning","521905","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3897","Subscriptions","521906","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3898","Gifts & Donations","521907","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3899","Stationary Out Of Stock","521908","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3900","Stationary From Suppliers","521909","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3901","Kitchen and Buffet Expenses","521910","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3902","Vehicle Expenses","521911","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3903","Convoyance Expenses","521912","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3904","Others - Office Various Expenses","521919","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3906","Other Bank Charges","522001","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3910","Loss On Fixed Assets Disposal","531101","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3911","Loss on Difference on Exchange","531102","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3912","Disposal of Business Branch","531103","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3914","Royalty to Parent Co.","531201","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3916","Income Tax","531301","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3917","Zakat","531302","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3919","Previous Year Adjustments Account","531411","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3921","Other Non Operating Expenses","531511","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3924","Credit Card Charges","541001","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3925","Amex Credit Cards Charges","541002","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3926","Bank Finance & Loan Charges","541003","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3927","Air Miles Card Charges","541004","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3928","Credit Card Swipe Charges","541005","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3929","PayPal Charges","541006","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3932","Amortization on Leasehold Improvement","551001","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3933","Depreciation Of Furniture & Office Equipment","551002","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3934","Depreciation Of Computer Hard & Soft","551003","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3935","Depreciation Of Motor Vehicles","551004","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3937","Consultancy Fees","560001","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3938","Provision for Doubtful Debts","560004","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"
"uae_account_3941","Closing Account","611001","account.data_account_type_expenses","l10n_ae.uae_chart_template_standard","False"

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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <!-- Account Tax Group -->
        <record id="ae_tax_group_5" model="account.tax.group">
            <field name="name">Tax 5%</field>
        </record>
        <record id="ae_tax_group_0" model="account.tax.group">
            <field name="name">Tax 0%</field>
        </record>
        <record id="ae_tax_group_exempted" model="account.tax.group">
            <field name="name">Tax Exempted</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.ae"/>
    </record>

    <record id="tax_report_line_base_all_sales" model="account.tax.report.line">
        <field name="name">VAT on Sales and all other Outputs (Base)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base" model="account.tax.report.line">
        <field name="name">1. Standard Rated supplies (Base)</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_abu_dhabi" model="account.tax.report.line">
        <field name="name">a. Abu Dhabi</field>
        <field name="tag_name">a. Abu Dhabi (Base)</field>
        <field name="code">STD_RATE_SUPP_BASE_AB</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_dubai" model="account.tax.report.line">
        <field name="name">b. Dubai</field>
        <field name="tag_name">b. Dubai (Base)</field>
        <field name="code">STD_RATE_SUPP_BASE_DB</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_sharjah" model="account.tax.report.line">
        <field name="name">c. Sharjah</field>
        <field name="tag_name">c. Sharjah (Base)</field>
        <field name="code">STD_RATE_SUPP_BASE_SJ</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_ajman" model="account.tax.report.line">
        <field name="name">d. Ajman</field>
        <field name="tag_name">d. Ajman (Base)</field>
        <field name="code">STD_RATE_SUPP_BASE_AJ</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_umm_al_quwain" model="account.tax.report.line">
        <field name="name">e. Umm Al Quwain</field>
        <field name="tag_name">e. Umm Al Quwain (Base)</field>
        <field name="code">STD_RATE_SUPP_BASE_UM</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_ras_al_khaima" model="account.tax.report.line">
        <field name="name">f. Ras Al-Khaima</field>
        <field name="tag_name">f. Ras Al-Khaima (Base)</field>
        <field name="code">STD_RATE_SUPP_BASE_RA</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_fujairah" model="account.tax.report.line">
        <field name="name">g. Fujairah</field>
        <field name="tag_name">g. Fujairah (Base)</field>
        <field name="code">STD_RATE_SUPP_BASE_FU</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_base_subtotal" model="account.tax.report.line">
        <field name="name">Sub Total</field>
        <field name="formula">STD_RATE_SUPP_BASE_AB + STD_RATE_SUPP_BASE_DB + STD_RATE_SUPP_BASE_SJ + STD_RATE_SUPP_BASE_AJ + STD_RATE_SUPP_BASE_UM + STD_RATE_SUPP_BASE_RA + STD_RATE_SUPP_BASE_FU</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_base"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
    </record>

    <record id="tax_report_line_tax_refund_tourist_base" model="account.tax.report.line">
        <field name="name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme</field>
        <field name="tag_name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Base)</field>
        <field name="code">TAX_REF_TOUR_SCHEME_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_supplies_reverse_charge_base" model="account.tax.report.line">
        <field name="name">3. Supplies subject to reverse charge provisions</field>
        <field name="tag_name">3. Supplies subject to reverse charge provisions (Base)</field>
        <field name="code">REVERSE_CHARGE_PRO_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_line_zero_rated_supplies_base" model="account.tax.report.line">
        <field name="name">4. Zero rated supplies</field>
        <field name="tag_name">4. Zero rated supplies (Base)</field>
        <field name="code">ZERO_RATE_SUPP_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_line_exempt_supplies_base" model="account.tax.report.line">
        <field name="name">5. Exempt supplies</field>
        <field name="tag_name">5. Exempt supplies (Base)</field>
        <field name="code">EXAMPT_SUPP_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_line_supplies_out_of_scope_base" model="account.tax.report.line">
        <field name="name">6. Out of scope</field>
        <field name="code">OUT_OF_SCOPE_BASE_0</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_line_import_uae_base" model="account.tax.report.line">
        <field name="name">7. Goods imported into the UAE</field>
        <field name="tag_name">7. Goods imported into the UAE (Base)</field>
        <field name="code">GOODS_IMPORT_IN_UAE_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_line_adjustment_import_uae_base" model="account.tax.report.line">
        <field name="name">8. Adjustments to goods imported into the UAE</field>
        <field name="code">ADJUST_GOODS_IMPORT_IN_UAE_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
    </record>

    <record id="tax_report_line_base_all_sales_total" model="account.tax.report.line">
        <field name="name">9. Total</field>
        <field name="formula">ADJUST_GOODS_IMPORT_IN_UAE_BASE + GOODS_IMPORT_IN_UAE_BASE + OUT_OF_SCOPE_BASE_0 + EXAMPT_SUPP_BASE + ZERO_RATE_SUPP_BASE + REVERSE_CHARGE_PRO_BASE + TAX_REF_TOUR_SCHEME_BASE + (STD_RATE_SUPP_BASE_AB + STD_RATE_SUPP_BASE_DB + STD_RATE_SUPP_BASE_SJ + STD_RATE_SUPP_BASE_AJ + STD_RATE_SUPP_BASE_UM + STD_RATE_SUPP_BASE_RA + STD_RATE_SUPP_BASE_FU)</field>
        <field name="parent_id" ref="tax_report_line_base_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
    </record>

    <record id="tax_report_line_base_all_expense" model="account.tax.report.line">
        <field name="name">VAT on Expenses and all other Inputs (Base)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_standard_rated_expense_base" model="account.tax.report.line">
        <field name="name">10. Standard rated expenses</field>
        <field name="tag_name">10. Standard rated expenses (Base)</field>
        <field name="code">STD_RATE_EXPENSES_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_line_expense_supplies_reverse_base" model="account.tax.report.line">
        <field name="name">11. Supplies subject to the reverse charge provisions</field>
        <field name="tag_name">11. Supplies subject to the reverse charge provisions (Base)</field>
        <field name="code">SUPP_REV_CHARGE_PRO_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_expense_out_of_scope" model="account.tax.report.line">
        <field name="name">12. Out of scope</field>
        <field name="code">OUT_OF_SCOPE_1_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_line_base_all_expense_total" model="account.tax.report.line">
        <field name="name">13. Totals</field>
        <field name="formula">OUT_OF_SCOPE_1_BASE + SUPP_REV_CHARGE_PRO_BASE + STD_RATE_EXPENSES_BASE</field>
        <field name="parent_id" ref="tax_report_line_base_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_line_vat_all_sales" model="account.tax.report.line">
        <field name="name">VAT on Sales and all other Outputs (Tax)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat" model="account.tax.report.line">
        <field name="name">1. Standard Rated supplies (Tax)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_abu_dhabi" model="account.tax.report.line">
        <field name="name">a. Abu Dhabi</field>
        <field name="tag_name">a. Abu Dhabi (Tax)</field>
        <field name="code">STD_RATE_SUPP_TAX_AB</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_dubai" model="account.tax.report.line">
        <field name="name">b. Dubai</field>
        <field name="tag_name">b. Dubai (Tax)</field>
        <field name="code">STD_RATE_SUPP_TAX_DB</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_sharjah" model="account.tax.report.line">
        <field name="name">c. Sharjah</field>
        <field name="tag_name">c. Sharjah (Tax)</field>
        <field name="code">STD_RATE_SUPP_TAX_SJ</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_ajman" model="account.tax.report.line">
        <field name="name">d. Ajman</field>
        <field name="tag_name">d. Ajman (Tax)</field>
        <field name="code">STD_RATE_SUPP_TAX_AJ</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_umm_al_quwain" model="account.tax.report.line">
        <field name="name">e. Umm Al Quwain</field>
        <field name="tag_name">e. Umm Al Quwain (Tax)</field>
        <field name="code">STD_RATE_SUPP_TAX_UM</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_ras_al_khaima" model="account.tax.report.line">
        <field name="name">f. Ras Al-Khaima</field>
        <field name="tag_name">f. Ras Al-Khaima (Tax)</field>
        <field name="code">STD_RATE_SUPP_TAX_RA</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_fujairah" model="account.tax.report.line">
        <field name="name">g. Fujairah</field>
        <field name="tag_name">g. Fujairah (Tax)</field>
        <field name="code">STD_RATE_SUPP_TAX_FU</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_line_standard_rated_supplies_vat_subtotal" model="account.tax.report.line">
        <field name="name">Sub Total</field>
        <field name="formula">STD_RATE_SUPP_TAX_AB + STD_RATE_SUPP_TAX_DB + STD_RATE_SUPP_TAX_SJ + STD_RATE_SUPP_TAX_AJ + STD_RATE_SUPP_TAX_UM + STD_RATE_SUPP_TAX_RA + STD_RATE_SUPP_TAX_FU</field>
        <field name="parent_id" ref="tax_report_line_standard_rated_supplies_vat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
    </record>

    <record id="tax_report_line_tax_refund_tourist_vat" model="account.tax.report.line">
        <field name="name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme</field>
        <field name="tag_name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Tax)</field>
        <field name="code">TAX_REF_TOUR_SCHEME_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_supplies_reverse_charge_vat" model="account.tax.report.line">
        <field name="name">3. Supplies subject to reverse charge provisions</field>
        <field name="tag_name">3. Supplies subject to reverse charge provisions (Tax)</field>
        <field name="code">REVERSE_CHARGE_PRO_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_line_zero_rated_supplies_vat" model="account.tax.report.line">
        <field name="name">4. Zero rated supplies</field>
        <field name="tag_name">4. Zero rated supplies (Tax)</field>
        <field name="code">ZERO_RATE_SUPP_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_line_exempt_supplies_vat" model="account.tax.report.line">
        <field name="name">5. Exempt supplies</field>
        <field name="tag_name">5. Exempt supplies (Tax)</field>
        <field name="code">EXAMPT_SUPP_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_line_supplies_out_of_scope_vat" model="account.tax.report.line">
        <field name="name">6. Out of scope</field>
        <field name="code">OUT_OF_SCOPE_TAX_0</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_line_import_uae_vat" model="account.tax.report.line">
        <field name="name">7. Goods imported into the UAE</field>
        <field name="tag_name">7. Goods imported into the UAE (Tax)</field>
        <field name="code">GOODS_IMPORT_IN_UAE_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_line_adjustment_import_uae_vat" model="account.tax.report.line">
        <field name="name">8. Adjustments to goods imported into the UAE</field>
        <field name="code">ADJUST_GOODS_IMPORT_IN_UAE_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
    </record>

    <record id="tax_report_line_vat_all_sales_total" model="account.tax.report.line">
        <field name="name">9. Total</field>
        <field name="formula">(STD_RATE_SUPP_TAX_AB + STD_RATE_SUPP_TAX_DB + STD_RATE_SUPP_TAX_SJ + STD_RATE_SUPP_TAX_AJ + STD_RATE_SUPP_TAX_UM + STD_RATE_SUPP_TAX_RA + STD_RATE_SUPP_TAX_FU) + OUT_OF_SCOPE_TAX_0 + ADJUST_GOODS_IMPORT_IN_UAE_TAX + GOODS_IMPORT_IN_UAE_TAX + EXAMPT_SUPP_TAX + ZERO_RATE_SUPP_TAX + REVERSE_CHARGE_PRO_TAX + TAX_REF_TOUR_SCHEME_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
    </record>

    <record id="tax_report_line_vat_all_expense" model="account.tax.report.line">
        <field name="name">VAT on Expenses and all other Inputs (Tax)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_line_standard_rated_expense_vat" model="account.tax.report.line">
        <field name="name">10. Standard rated expenses</field>
        <field name="tag_name">10. Standard rated expenses (Tax)</field>
        <field name="code">STD_RATE_EXPENSES_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_line_expense_supplies_reverse_vat" model="account.tax.report.line">
        <field name="name">11. Supplies subject to the reverse charge provisions</field>
        <field name="tag_name">11. Supplies subject to the reverse charge provisions (Tax)</field>
        <field name="code">SUPP_REV_CHARGE_PRO_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_expense_out_of_scope_vat" model="account.tax.report.line">
        <field name="name">12. Out of scope</field>
        <field name="code">OUT_OF_SCOPE_1_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_line_vat_all_expense_total" model="account.tax.report.line">
        <field name="name">13. Totals</field>
        <field name="formula">OUT_OF_SCOPE_1_TAX + SUPP_REV_CHARGE_PRO_TAX + STD_RATE_EXPENSES_TAX</field>
        <field name="parent_id" ref="tax_report_line_vat_all_expense"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_line_net_vat_due" model="account.tax.report.line">
        <field name="name">Net VAT Due</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="formula">((STD_RATE_SUPP_TAX_AB + STD_RATE_SUPP_TAX_DB + STD_RATE_SUPP_TAX_SJ + STD_RATE_SUPP_TAX_AJ + STD_RATE_SUPP_TAX_UM + STD_RATE_SUPP_TAX_RA + STD_RATE_SUPP_TAX_FU) + OUT_OF_SCOPE_TAX_0 + ADJUST_GOODS_IMPORT_IN_UAE_TAX + GOODS_IMPORT_IN_UAE_TAX + EXAMPT_SUPP_TAX + ZERO_RATE_SUPP_TAX + REVERSE_CHARGE_PRO_TAX + TAX_REF_TOUR_SCHEME_TAX) - (OUT_OF_SCOPE_1_TAX + SUPP_REV_CHARGE_PRO_TAX + STD_RATE_EXPENSES_TAX)</field>
    </record>

    <record id="tax_report_line_total_value_due_tax_period" model="account.tax.report.line">
        <field name="name">14. Total value of due tax for the period</field>
        <field name="formula">(STD_RATE_SUPP_TAX_AB + STD_RATE_SUPP_TAX_DB + STD_RATE_SUPP_TAX_SJ + STD_RATE_SUPP_TAX_AJ + STD_RATE_SUPP_TAX_UM + STD_RATE_SUPP_TAX_RA + STD_RATE_SUPP_TAX_FU) + OUT_OF_SCOPE_TAX_0 + ADJUST_GOODS_IMPORT_IN_UAE_TAX + GOODS_IMPORT_IN_UAE_TAX + EXAMPT_SUPP_TAX + ZERO_RATE_SUPP_TAX + REVERSE_CHARGE_PRO_TAX + TAX_REF_TOUR_SCHEME_TAX</field>
        <field name="parent_id" ref="tax_report_line_net_vat_due"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_line_total_value_recoverable_tax_period" model="account.tax.report.line">
        <field name="name">15. Total value of recoverable tax for the period</field>
        <field name="formula">OUT_OF_SCOPE_1_TAX + SUPP_REV_CHARGE_PRO_TAX + STD_RATE_EXPENSES_TAX</field>
        <field name="parent_id" ref="tax_report_line_net_vat_due"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_line_net_vat_due_period" model="account.tax.report.line">
        <field name="name">16. Net VAT due (or reclaimed) for the period</field>
        <field name="formula">((STD_RATE_SUPP_TAX_AB + STD_RATE_SUPP_TAX_DB + STD_RATE_SUPP_TAX_SJ + STD_RATE_SUPP_TAX_AJ + STD_RATE_SUPP_TAX_UM + STD_RATE_SUPP_TAX_RA + STD_RATE_SUPP_TAX_FU) + OUT_OF_SCOPE_TAX_0 + ADJUST_GOODS_IMPORT_IN_UAE_TAX + GOODS_IMPORT_IN_UAE_TAX + EXAMPT_SUPP_TAX + ZERO_RATE_SUPP_TAX + REVERSE_CHARGE_PRO_TAX + TAX_REF_TOUR_SCHEME_TAX) - (OUT_OF_SCOPE_1_TAX + SUPP_REV_CHARGE_PRO_TAX + STD_RATE_EXPENSES_TAX)</field>
        <field name="parent_id" ref="tax_report_line_net_vat_due"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_dubai')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_dubai')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_dubai')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_dubai')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_abu_dhabi')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_abu_dhabi')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_abu_dhabi')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_abu_dhabi')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_sharjah')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_sharjah')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_sharjah')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_sharjah')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_ajman')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ajman')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_ajman')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ajman')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_umm_al_quwain')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_umm_al_quwain')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_umm_al_quwain')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_umm_al_quwain')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_ras_al_khaima')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ras_al_khaima')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_ras_al_khaima')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ras_al_khaima')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_fujairah')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_fujairah')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_base_fujairah')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_fujairah')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_exempt_supplies_base')],
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
                'minus_report_line_ids': [ref('tax_report_line_exempt_supplies_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_zero_rated_supplies_base')],
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
                'minus_report_line_ids': [ref('tax_report_line_zero_rated_supplies_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
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
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_dubai')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_dubai')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_dubai')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_dubai')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_abu_dhabi')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_abu_dhabi')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_abu_dhabi')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_abu_dhabi')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_sharjah')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_sharjah')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_sharjah')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_sharjah')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_ajman')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ajman')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_ajman')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ajman')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_umm_al_quwain')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_umm_al_quwain')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_umm_al_quwain')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_umm_al_quwain')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_ras_al_khaima')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ras_al_khaima')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_ras_al_khaima')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_ras_al_khaima')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_fujairah')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_fujairah')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_base'),ref('tax_report_line_standard_rated_supplies_base_fujairah')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_supplies_reverse_charge_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3785'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_supplies_vat_fujairah')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_tax_refund_tourist_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_tax_refund_tourist_vat')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_tax_refund_tourist_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_tax_refund_tourist_vat')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_expense_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_standard_rated_expense_vat')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_expense_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_standard_rated_expense_vat')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
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
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
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
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_import_uae_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_line_import_uae_vat')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_import_uae_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_line_import_uae_vat')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_expense_supplies_reverse_base')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'minus_report_line_ids': [ref('tax_report_line_expense_supplies_reverse_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_expense_supplies_reverse_base')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
                'plus_report_line_ids': [ref('tax_report_line_expense_supplies_reverse_vat')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('uae_account_3726'),
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
        <field name="tax_dest_id" ref="uae_purchase_tax_0"/>
        <field name="position_id" ref="account_fiscal_position_non_uae_countries"/>
    </record>
</odoo>

```

## File: data\l10n_ae_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_ae_statements_menu" name="UAE" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

     <record id="uae_chart_template_standard" model="account.chart.template">
         <field name="name">U.A.E Chart of Accounts - Standard</field>
         <field name="code_digits">6</field>
         <field name="bank_account_code_prefix">101</field>
         <field name="cash_account_code_prefix">105</field>
         <field name="transfer_account_code_prefix">100</field>
         <field name="currency_id" ref="base.AED" />
         <field name="complete_tax_set" eval="True"/>
     </record>
</odoo>

```

## File: data\l10n_ae_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="uae_chart_template_standard" model="account.chart.template">
         <field name="property_account_receivable_id" ref="uae_account_3681" />
         <field name="property_account_payable_id" ref="uae_account_3753" />
         <field name="property_account_expense_categ_id" ref="uae_account_3840" />
         <field name="property_account_income_categ_id" ref="uae_account_3810" />
         <field name="expense_currency_exchange_account_id" ref="uae_account_3911"/>
         <field name="income_currency_exchange_account_id" ref="uae_account_3833"/>
         <field name="default_pos_receivable_account_id" ref="uae_account_3682" />
     </record>
</odoo>

```

## File: data\l10n_ae_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- set VAT label to show on invoice report -->
    <record id="base.ae" model="res.country">
        <field name="vat_label">TRN</field>
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

## File: views\report_invoice_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//p[@name='payment_term']" position="after">
            <p t-if="o.company_id.country_id.code == 'AE' and o.partner_id.country_id.code != 'AE' and o.env.ref('l10n_ae.gcc_countries_group') in o.partner_id.country_id.country_group_ids">
                Supply between <b>United Arad Emirates</b> and <b><span t-field="o.partner_id.country_id.name"/></b>
            </p>
        </xpath>
        <xpath expr="//h2/span" position="before">
            <span t-if="o.company_id.country_id.code == 'AE' and o.move_type in ['out_invoice', 'out_refund']">Tax</span>
        </xpath>

        <xpath expr="//div[hasclass('clearfix')]" position="after">
            <div t-if="o.company_id.country_id.code == 'AE' and o.currency_id != o.company_id.currency_id" id="aed_amounts" class="row clearfix ml-auto my-3 text-nowrap table">
                <t t-set="aed_rate" t-value="o.env['res.currency']._get_conversion_rate(o.currency_id, o.company_id.currency_id, o.company_id, o.invoice_date or datetime.date.today())"/>
                <div name="exchange_rate" class="col-auto">
                    <strong>Exchange Rate</strong>
                    <p class="m-0" t-esc="aed_rate" t-options='{"widget": "float", "precision": 5}'/>
                </div>
                <div name="aed_subtotal" class="col-auto">
                    <strong>Subtotal (AED)</strong>
                    <p class="m-0" t-esc="o.amount_untaxed_signed" t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/>
                </div>
                <div name="aed_vat_amount" class="col-auto">
                    <strong>Tax Amount (AED)</strong>
                    <p class="m-0" t-esc="o.currency_id._convert(o.amount_tax, o.company_id.currency_id, o.company_id, o.invoice_date or datetime.date.today())" t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/>
                </div>
                <div name="aed_total" class="col-auto">
                    <strong>Total (AED)</strong>
                    <p class="m-0" t-esc="o.amount_total_signed" t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/>
                </div>
            </div>
        </xpath>
    </template>
</odoo>

```

