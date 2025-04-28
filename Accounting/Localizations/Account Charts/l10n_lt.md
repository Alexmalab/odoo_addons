# Odoo Module: l10n_lt

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from odoo import api, SUPERUSER_ID


def load_translations(cr, registry):
    """Load template translations."""
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref(
        'l10n_lt.account_chart_template_lithuania').process_coa_translations()

```

## File: __manifest__.py

```python
# Author: Silvija Butko. Copyright: JSC Focusate.
# Co-Authors: Eimantas Nėjus, Andrius Laukavičius. Copyright: JSC Focusate
# See LICENSE file for full copyright and licensing details.
{
    'name': "Lithuania - Accounting",
    'version': '1.0.0',
    'description': """
        Chart of Accounts (COA) Template for Lithuania's Accounting.

        This module also includes:

        * List of available banks in Lithuania.
        * Tax groups.
        * Most common Lithuanian Taxes.
        * Fiscal positions.
        * Account Tags.
    """,
    'license': 'LGPL-3',
    'author': "Focusate",
    'website': "http://www.focusate.eu",
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'l10n_multilang',
    ],
    'data': [
        'data/account_account_tag_data.xml',
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_setup_data.xml',
        'data/res_bank_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template_data.xml',
        # Try Loading COA for Current Company
        'data/account_chart_template_load.xml',
        'data/menuitem_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': 'load_translations',
    'installable': True,
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","tag_ids/id","reconcile"
"account_account_template_1130","Software Acquisition Cost",1130,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_1_3","False"
"account_account_template_1138","Amortization of Software Value (-)",1138,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_1_3","False"
"account_account_template_1200","Land Acquisition Cost",1200,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_1","False"
"account_account_template_1201","Change of Land Value after Revaluation",1201,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_1","False"
"account_account_template_1210","Acquisition Cost of Buildings",1210,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_2","False"
"account_account_template_1212","Buildings being Prepared for Use",1212,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_2","False"
"account_account_template_1217","Depreciation of the Acquisition Cost of Buildings (-)",1217,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_2","False"
"account_account_template_1220","Acquisition Cost of Machinery and Equipment",1220,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_3","False"
"account_account_template_1222","Machinery and Equipment being Prepared for Use",1222,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_3","False"
"account_account_template_1227","Depreciation of the Acquisition Cost of Machinery and Equipment (-)",1227,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_3","False"
"account_account_template_1230","Acquisition Cost of Vehicles",1230,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_4","False"
"account_account_template_1232","Vehicles being Prepared for Use",1232,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_4","False"
"account_account_template_1237","Depreciation of the Acquisition Cost of Vehicles (-)",1237,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_4","False"
"account_account_template_1240","Acquisition Cost of Other Equipment, Appliances and Tools",1240,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_5","False"
"account_account_template_1242","Other Equipment, Appliances and Tools being Prepared for Use",1242,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_5","False"
"account_account_template_1247","Depreciation of the Acquisition Cost of Other Equipment, Appliances and Tools (-)",1247,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_5","False"
"account_account_template_12500","Cost of the Acquisition of Land as Investment Property",12500,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_6_1","False"
"account_account_template_12509","Value Loss of the Land as Investment Property (-)",12509,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_6_1","False"
"account_account_template_12510","Cost of the Acquisition of Buildings as Investment Property",12510,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_6_2","False"
"account_account_template_12517","Depreciation of the Acquisition Cost of Buildings as Investment Property (-)",12517,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_6_2","False"
"account_account_template_12519","Value Loss of Buildings as Investment Property (-)",12519,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_6_2","False"
"account_account_template_1260","Advance Payments for Long-Term Tangible Assets",1260,"account.data_account_type_fixed_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_2_7","False"
"account_account_template_166110","Acquisition Cost of Other Non-Equity Securities",166110,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_7","False"
"account_account_template_166119","Value Loss of Other Non-Equity Securities (-)",166119,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_7","False"
"account_account_template_1665","Term Deposits",1665,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_7","False"
"account_account_template_16700","Value of Receivable Debts from Customers",16700,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_8","False"
"account_account_template_16710","Value of Loans Granted",16710,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_8","False"
"account_account_template_1672","Receivable Amount of Leasing (Financial Rent) after One Year",1672,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_8","False"
"account_account_template_16740","Value of Receivables",16740,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_8","False"
"account_account_template_1680","Advance Payments for Financial Assets",1680,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_9","False"
"account_account_template_1681","Derivative Financial Assets",1681,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_3_9","False"
"account_account_template_171","Deferred Profit Tax Asset",171,"account.data_account_type_non_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_a_4_1","False"
"account_account_template_2010","Acquisition Cost of Raw Materials, Materials and Assembly Parts",2010,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_1","False"
"account_account_template_2011","Raw Materials, Materials and Assembly Parts in Transit",2011,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_1","True"
"account_account_template_2012","Raw Materials, Materials and Assembly Parts at Third Parties",2012,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_1","False"
"account_account_template_2019","Value Loss of Raw Materials, Materials and Assembly Parts (-)",2019,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_1","False"
"account_account_template_20200","Cost of Incomplete Production",20200,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_2","False"
"account_account_template_20210","Cost of Executive Works",20210,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_2","False"
"account_account_template_2030","Cost of Production",2030,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_3","False"
"account_account_template_2035","Production in Transit",2035,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_3","True"
"account_account_template_2036","Production at Third Parties",2036,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_3","False"
"account_account_template_2039","Value Loss of Production (-)",2039,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_3","False"
"account_account_template_2040","Cost of Purchased Goods for Resale",2040,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_4","False"
"account_account_template_2045","Purchased Goods for Resale in Transit",2045,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_4","True"
"account_account_template_2046","Purchased Goods for Resale at Third Parties",2046,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_4","False"
"account_account_template_2049","Value Loss of Purchased Goods for Resale (-)",2049,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_4","False"
"account_account_template_2060","Cost of Tangible Assets",2060,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_6","False"
"account_account_template_2080","Prepayments to Suppliers",2080,"account.data_account_type_prepayments","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_7","False"
"account_account_template_2084","Deposit",2084,"account.data_account_type_prepayments","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_1_7","False"
"account_account_template_2410","Value of Debts from Customers",2410,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_1","True"
"account_account_template_2411","Value of Debts from POS Customers",2411,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_1","True"
"account_account_template_24400","Value of Loans Granted",24400,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_2441","Receivable Value-Added Tax",2441,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","False"
"account_account_template_2442","Prepaid Profit Tax",2442,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_2443","Tax Overpayments",2443,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_2444","VSDF (Lithuanian State Social Insurance Fund) Debt to the Company",2444,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_24450","Value of Receivables from Accountable Persons",24450,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_24460","Value of Other Receivable Debts",24460,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_24471","Profit paid in Advance to the Owners of the Company",24471,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_24472","Payed Funds for Personal Use to the Owners of the Company",24472,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_2449","Other Questionable Debts (-)",2449,"account.data_account_type_receivable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_2_4","True"
"account_account_template_26200","Acquisition Cost of Equity Securities of Other Companies",26200,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_3_2","False"
"account_account_template_26209","Value Loss of Equity Securities of Other Companies (-)",26209,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_3_2","False"
"account_account_template_262110","Acquisition Cost of Other Non-Equity Securities",262110,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_3_2","False"
"account_account_template_262119","Value Loss of Other Non-Equity Securities (-)",262119,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_3_2","False"
"account_account_template_2623","Term Deposits",2623,"account.data_account_type_current_assets","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_3_2","False"
"account_chart_template_lithuania_liquidity_transfer","Cash in Transit",273,"account.data_account_type_current_assets","account_chart_template_lithuania","l10n_lt.account_account_tag_b_4","True"
"account_account_template_274","Cash Equivalents",274,"account.data_account_type_liquidity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_4","False"
"account_account_template_279","Frozen Funds (-)",279,"account.data_account_type_liquidity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_b_4","False"
"account_account_template_291","Future Expenses",291,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_c_prepayments_accrued_income","False"
"account_account_template_292","Accumulative Income",292,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_c_prepayments_accrued_income","False"
"account_account_template_3011","Ordinary Shares",3011,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_1_1","False"
"account_account_template_3012","Preference Shares",3012,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_1_1","False"
"account_account_template_302","Subscribed Capital Unpaid (-)",302,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_1_2","False"
"account_account_template_303","Own Shares (Yawn) (-)",303,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_1_3","False"
"account_account_template_305","Company Owner’s Capital",305,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_1_3","False"
"account_account_template_308","Owners’ Contributions",308,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_1_3","False"
"account_account_template_331","Mandatory or Reserve Capital",331,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_4_1","False"
"account_account_template_3411","Acknowledged Net Profit (Loss) of The Accounting Year in The Profit (Loss) Report",3411,"account.data_unaffected_earnings","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_5_1","False"
"account_account_template_3412","Unacknowledged Net Profit (Loss) of The Accounting Year in The Profit (Loss) Report",3412,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_5_1","False"
"account_account_template_3421","Acknowledged Net Profit (Loss) of The Previous Accounting Year in The Profit (Loss) Report",3421,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_5_2","False"
"account_account_template_3422","Unacknowledged Net Profit (Loss) of The Previous Accounting Year in The Profit (Loss) Report",3422,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_5_2","False"
"account_account_template_3424","Profit (Loss) of Material Corrections of The Previous Years",3424,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_d_5_2","False"
"account_account_template_390","Common Summary of Accounts",390,"account.data_account_type_equity","l10n_lt.account_chart_template_lithuania",,"False"
"account_account_template_411","Provisions for Pensions and Similar Liabilities",411,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_f_1","False"
"account_account_template_412","Tax Provisions",412,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_f_2","False"
"account_account_template_413","Other Provisions",413,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_f_3","False"
"account_account_template_4211","Leasing (Financial Rent) or Other Obligations",4211,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_1","False"
"account_account_template_4214","Other Debt Obligations",4214,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_1","False"
"account_account_template_4220","Long-Term Liabilities under Loan Agreements",4220,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_2","False"
"account_account_template_4221","Other Obligations",4221,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_2","False"
"account_account_template_4230","Prepayments from Customers",4230,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_3","False"
"account_account_template_4231","Prepayments from Service Receivers",4231,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_3","False"
"account_account_template_424","Debts to Suppliers",424,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_4","False"
"account_account_template_428","Other Payables and Long-Term Liabilities",428,"account.data_account_type_non_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_1_8","False"
"account_account_template_4401","Part of Leasing (Financial Rent) or Other Obligations in The Current Year",4401,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_1","False"
"account_account_template_4403","Part of Other Long-Term Debts in The Current Year",4403,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_1","False"
"account_account_template_4410","Liabilities under Short-Term Loan Agreements",4410,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_2","False"
"account_account_template_4411","Part of Debts to Credit Institutions in The Current Year",4411,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_2","False"
"account_account_template_4413","Other Obligations (Executables and etc.)",4413,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_2","False"
"account_account_template_4420","Prepayments from Customers",4420,"account.data_account_type_prepayments","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_3","False"
"account_account_template_4421","Prepayments from Service Receivers",4421,"account.data_account_type_prepayments","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_3","False"
"account_account_template_4430","Debts to Suppliers for Goods and Services",4430,"account.data_account_type_payable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_4","True"
"account_account_template_4431","Other Debts to Suppliers",4431,"account.data_account_type_payable","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_4","True"
"account_account_template_4470","Obligations of Profit Tax",4470,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_8","False"
"account_account_template_4471","Other Obligations",4471,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_8","False"
"account_account_template_4480","Payable Salary",4480,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_9","False"
"account_account_template_4481","Payable Personal Income Tax",4481,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_9","False"
"account_account_template_4482","Contributions of Payable Social Security",4482,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_9","False"
"account_account_template_4483","Contributions of Payable Guarantee Fund",4483,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_9","False"
"account_account_template_4485","Leave Liabilities",4485,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_9","False"
"account_account_template_4486","Contributions of Payable Compulsory Health Insurance",4486,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_9","False"
"account_account_template_4490","Payable Dividends",4490,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_10","False"
"account_account_template_4491","Payable Bonuses",4491,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_10","False"
"account_account_template_4492","Payable Value-Added Tax",4492,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_10","False"
"account_account_template_4493","Other Payable Taxes to Budget",4493,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_10","False"
"account_account_template_4494","Other Payables",4494,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_10","False"
"account_account_template_4495","Payables to the Owners of the Company",4495,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_g_2_10","False"
"account_account_template_491","Accumulated Expenses",491,"account.data_account_type_revenue","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_h_accruals_deferred_income","False"
"account_account_template_492","Income of Future Periods",492,"account.data_account_type_revenue","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_h_accruals_deferred_income","False"
"account_account_template_5000","Income from Goods Sold",5000,"account.data_account_type_revenue","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_1_net_turnover","False"
"account_account_template_5001","Income from Services Sold",5001,"account.data_account_type_revenue","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_1_net_turnover","False"
"account_account_template_5003","Income from Production Sold",5003,"account.data_account_type_revenue","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_1_net_turnover","False"
"account_account_template_509","Discounts, Returns (-)",509,"account.data_account_type_revenue","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_1_net_turnover","False"
"account_account_template_5400","Profit from Asset Disposition",5400,"account.data_account_type_other_income","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_6_other_operating_results","False"
"account_account_template_5401","Other Income",5401,"account.data_account_type_other_income","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_6_other_operating_results","False"
"account_account_template_5600","Income from Other Long-Term Investments and Loan Interests",5600,"account.data_account_type_other_income","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_8_income_other_longterm_investments_loans","False"
"account_account_template_5802","Income from Other Loan Interests",5802,"account.data_account_type_other_income","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_9_other_interest_similar_income","False"
"account_account_template_5803","Positive Currency Exchange Difference",5803,"account.data_account_type_revenue","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_9_other_interest_similar_income,account.account_tag_financing","False"
"account_account_template_5804","Income from Fines",5804,"account.data_account_type_other_income","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_9_other_interest_similar_income","False"
"account_account_template_5809","Profit from Disposition of Investments",5809,"account.data_account_type_other_income","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_9_other_interest_similar_income","False"
"account_account_template_6000","Cost of Goods Sold",6000,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_6001","Cost of Services Sold",6001,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60030","Cost of Production Sold",60030,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60031","Short-Term Production Inventory",60031,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60032","Direct Transport",60032,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60033","Direct Commissions",60033,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60034","Direct Payroll of Employees Working in Production",60034,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60040","Working Tools for Production",60040,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60041","Maintenance of Working Tools for Production",60041,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60042","Maintenance of Production Transport",60042,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60043","Energy Resources",60043,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_60044","Fuel",60044,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_6005","Increase (Decrease) in Inventories",6005,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_609","Discounts, Returns (-)",609,"account.data_account_type_direct_costs","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_2_cost_of_sales","False"
"account_account_template_6202","Cost of Services and Goods Advertising",6202,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_4_selling_expenses","False"
"account_account_template_6203","Payroll and Other Costs of Employees Working in Sales",6203,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_4_selling_expenses","False"
"account_account_template_6208","Other Costs of Sales",6208,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_4_selling_expenses","False"
"account_account_template_6209","Discounts Received (-)",6209,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_4_selling_expenses","False"
"account_account_template_6300","Rental Costs",6300,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6301","Costs of Maintenance and Exploitation",6301,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6302","Fuel Costs",6302,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6303","Insurance Costs",6303,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6304","Payroll and Other Costs of Employees Working in Administration",6304,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6306","Depreciation Costs of Long-Term Tangible Assets",6306,"account.data_account_type_depreciation","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6307","Amortization Costs of Intangible Assets Value",6307,"account.data_account_type_depreciation","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63080","Costs of Real Estate Tax",63080,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63081","Costs of not Deductible Value-Added Tax",63081,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63082","Pollution Tax Costs",63082,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63083","Costs of Other Taxes",63083,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63090","Costs of Value Loss of Debts from Customers",63090,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63091","Costs of Inventory Value Loss",63091,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63092","Costs of Intangible Assets Value Loss",63092,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63093","Costs of Value Loss of Long-Term Tangible Assets",63093,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63094","Costs of Value Loss of Other Assets",63094,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6310","Provisions Costs",6310,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6311","Costs of Fines",6311,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63120","Other Costs and Bank Taxes",63120,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63121","Representative Costs",63121,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63122","Daily Allowance",63122,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_63123","Support",63123,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_5_general_administrative_expenses","False"
"account_account_template_6400","Asset Disposition Losses",6400,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_6_other_operating_results","False"
"account_account_template_6401","Other Costs",6401,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_6_other_operating_results","False"
"account_account_template_6701","Costs of Value Loss of Long-Term Financial Assets",6701,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_10_impaired_fin_assets_short_investments","False"
"account_account_template_6702","Costs of Value Loss of Short-Term Investments",6702,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_10_impaired_fin_assets_short_investments","False"
"account_account_template_6802","Costs of Other Loan Interests",6802,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False"
"account_account_template_6803","Negative Currency Exchange Difference",6803,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False"
"account_account_template_6804","Costs of Fines",6804,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False"
"account_account_template_6805","Representation and other disallowed deductions",6805,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False"
"account_account_template_6806","Costs of Interests for Financial Rent",6806,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False"
"account_account_template_6809","Investment Disposition Losses",6809,"account.data_account_type_expenses","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False"
"account_account_template_6900","Profit and Other Similar Taxes of The Current Year",6900,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_12_tax_on_profit","False"
"account_account_template_6901","Costs (Income) of Deferred Value-Added Tax",6901,"account.data_account_type_current_liabilities","l10n_lt.account_chart_template_lithuania","l10n_lt.account_account_tag_12_tax_on_profit","False"
"account_account_template_999","Interim Account",999,"account.data_account_type_liquidity","account_chart_template_lithuania",,"False"

```

## File: data\account_account_tag_data.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo noupdate="1">
  <record id="account_account_tag_a_1_1" model="account.account.tag">
    <field name="name">A.1.1. Assets arising from development</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_2" model="account.account.tag">
    <field name="name">A.1.2. Goodwill</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_3" model="account.account.tag">
    <field name="name">A.1.3. Software</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_4" model="account.account.tag">
    <field name="name">A.1.4. Concessions, patents, licenses, trade marks and similar rights</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_5" model="account.account.tag">
    <field name="name">A.1.5. Other intangible assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_6" model="account.account.tag">
    <field name="name">A.1.6. Advance payments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_1" model="account.account.tag">
    <field name="name">A.2.1. Land</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_2" model="account.account.tag">
    <field name="name">A.2.2. Buildings and structures</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_3" model="account.account.tag">
    <field name="name">A.2.3. Machinery and plant</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_4" model="account.account.tag">
    <field name="name">A.2.4. Vehicles</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_5" model="account.account.tag">
    <field name="name">A.2.5. Other equipment, fittings and tools</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_6_1" model="account.account.tag">
    <field name="name">A.2.6.1. Land</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_6_2" model="account.account.tag">
    <field name="name">A.2.6.2. Buildings</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_7" model="account.account.tag">
    <field name="name">A.2.7. Advance payments and tangible assets under construction (production)</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_1" model="account.account.tag">
    <field name="name">A.3.1. Shares in entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_2" model="account.account.tag">
    <field name="name">A.3.2. Loans to entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_3" model="account.account.tag">
    <field name="name">A.3.3. Amounts receivable from entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_4" model="account.account.tag">
    <field name="name">A.3.4. Shares in associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_5" model="account.account.tag">
    <field name="name">A.3.5. Loans to associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_6" model="account.account.tag">
    <field name="name">A.3.6. Amounts receivable from the associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_7" model="account.account.tag">
    <field name="name">A.3.7. Long-term investments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_8" model="account.account.tag">
    <field name="name">A.3.8. Amounts receivable after one year</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_9" model="account.account.tag">
    <field name="name">A.3.9. Other financial assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_4_1" model="account.account.tag">
    <field name="name">A.4.1. Assets of the deferred tax on profit</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_4_2" model="account.account.tag">
    <field name="name">A.4.2. Biological assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_4_3" model="account.account.tag">
    <field name="name">A.4.3. Other assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_1" model="account.account.tag">
    <field name="name">B.1.1. Raw materials, materials and consumables</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_2" model="account.account.tag">
    <field name="name">B.1.2. Production and work in progress</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_3" model="account.account.tag">
    <field name="name">B.1.3. Finished goods</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_4" model="account.account.tag">
    <field name="name">B.1.4. Goods for resale</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_5" model="account.account.tag">
    <field name="name">B.1.5. Biological assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_6" model="account.account.tag">
    <field name="name">B.1.6. Fixed tangible assets held for sale</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_7" model="account.account.tag">
    <field name="name">B.1.7. Advance payments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_1" model="account.account.tag">
    <field name="name">B.2.1. Trade debtors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_2" model="account.account.tag">
    <field name="name">B.2.2. Amounts owed by entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_3" model="account.account.tag">
    <field name="name">B.2.3. Amounts owed by associates entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_4" model="account.account.tag">
    <field name="name">B.2.4. Other debtors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_3_1" model="account.account.tag">
    <field name="name">B.3.1. Shares in entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_3_2" model="account.account.tag">
    <field name="name">B.3.2. Other investments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_4" model="account.account.tag">
    <field name="name">B.4. CASH AND CASH EQUIVALENTS</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_c_prepayments_accrued_income" model="account.account.tag">
    <field name="name">C. PREPAYMENTS AND ACCRUED INCOME</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_1_1" model="account.account.tag">
    <field name="name">D.1.1. Authorized (subscribed) or primary capital</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_1_2" model="account.account.tag">
    <field name="name">D.1.2. Subscribed capital unpaid (–)</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_1_3" model="account.account.tag">
    <field name="name">D.1.3. Own shares (–)</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_2_share_premium" model="account.account.tag">
    <field name="name">D.2. SHARE PREMIUM ACCOUNT</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_3_revaluation_reserve" model="account.account.tag">
    <field name="name">D.3. REVALUATION RESERVE</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_4_1" model="account.account.tag">
    <field name="name">D.4.1. Compulsory reserve or emergency (reserve) capital</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_4_2" model="account.account.tag">
    <field name="name">D.4.2. Reserve for acquiring own shares</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_4_3" model="account.account.tag">
    <field name="name">D.4.3. Other reserves</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_5_1" model="account.account.tag">
    <field name="name">D.5.1. Profit (loss) for the reporting year </field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_5_2" model="account.account.tag">
    <field name="name">D.5.2. Profit (loss) brought forward</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_e_grants_subsidies" model="account.account.tag">
    <field name="name">E. GRANTS, SUBSIDIES</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_f_1" model="account.account.tag">
    <field name="name">F.1. Provisions for pensions and similar obligations</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_f_2" model="account.account.tag">
    <field name="name">F.2. Provisions for taxation</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_f_3" model="account.account.tag">
    <field name="name">F.3. Other provisions</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_1" model="account.account.tag">
    <field name="name">G.1.1. Debenture loans</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_2" model="account.account.tag">
    <field name="name">G.1.2. Amounts owed to credit institutions</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_3" model="account.account.tag">
    <field name="name">G.1.3. Payments received on account</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_4" model="account.account.tag">
    <field name="name">G.1.4. Trade creditors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_5" model="account.account.tag">
    <field name="name">G.1.5. Amounts payable under the bills and checks </field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_6" model="account.account.tag">
    <field name="name">G.1.6. Amounts payable to the entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_7" model="account.account.tag">
    <field name="name">G.1.7. Amounts payable to the associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_8" model="account.account.tag">
    <field name="name">G.1.8. Other amounts payable and long-term liabilities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_1" model="account.account.tag">
    <field name="name">G.2.1. Debenture loans</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_2" model="account.account.tag">
    <field name="name">G.2.2. Amounts owed to credit institutions</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_3" model="account.account.tag">
    <field name="name">G.2.3. Payments received on account</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_4" model="account.account.tag">
    <field name="name">G.2.4. Trade creditors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_5" model="account.account.tag">
    <field name="name">G.2.5. Amounts payable under the bills and checks </field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_6" model="account.account.tag">
    <field name="name">G.2.6. Amounts payable to the entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_7" model="account.account.tag">
    <field name="name">G.2.7. Amounts payable to the associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_8" model="account.account.tag">
    <field name="name">G.2.8. Liabilities of tax on profit</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_9" model="account.account.tag">
    <field name="name">G.2.9. Liabilities related to employment relations</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_10" model="account.account.tag">
    <field name="name">G.2.10. Other amounts payable and short-term liabilities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_h_accruals_deferred_income" model="account.account.tag">
    <field name="name">H. ACCRUALS AND DEFERRED INCOME</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_1_net_turnover" model="account.account.tag">
    <field name="name">1. Net turnover</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_2_cost_of_sales" model="account.account.tag">
    <field name="name">2. Cost of sales</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_3_adjustments_of_biological_assets" model="account.account.tag">
    <field name="name">3. Fair value adjustments of the biological assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_4_selling_expenses" model="account.account.tag">
    <field name="name">4. Selling expenses</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_5_general_administrative_expenses" model="account.account.tag">
    <field name="name">5. General and administrative expenses</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_6_other_operating_results" model="account.account.tag">
    <field name="name">6. Other operating results</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_7_income_investments_parent" model="account.account.tag">
    <field name="name">7. Income from investments in the shares of parent, subsidiaries and associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_8_income_other_longterm_investments_loans" model="account.account.tag">
    <field name="name">8. Income from other long-term investments and loans</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_9_other_interest_similar_income" model="account.account.tag">
    <field name="name">9. Other interest and similar income</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_10_impaired_fin_assets_short_investments" model="account.account.tag">
    <field name="name">10. The impairment of the financial assets and short-term investments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_11_interest_other_similar_expenses" model="account.account.tag">
    <field name="name">11. Interest and other similar expenses</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_12_tax_on_profit" model="account.account.tag">
    <field name="name">12. Tax on profit</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Chart of Accounts Template -->
    <record id="account_chart_template_lithuania" model="account.chart.template">
        <field name="name">Lithuania - Accounting</field>
        <field name="code_digits">1</field>
        <field name="currency_id" ref="base.EUR"/>
        <!-- Anglo-Saxon Accounting is used for reporting cost of good sold
            when products are sold/delivered.-->
        <field name="use_anglo_saxon" eval="True"/>
        <!-- Languages for which the translations of templates could be
        copied in the final object when generating them from templates -->
        <field name="spoken_languages" eval="'lt_LT'"/>
        <field name="bank_account_code_prefix">271</field>
        <field name="cash_account_code_prefix">272</field>
        <field name="transfer_account_code_prefix">273</field>
    </record>
</odoo>

```

## File: data\account_chart_template_load.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_lt.account_chart_template_lithuania')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_setup_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Setup Chart of Accounts Template -->
    <record id="account_chart_template_lithuania" model="account.chart.template">
        <field name="property_account_receivable_id" ref="account_account_template_2410"/>
        <field name="property_account_payable_id" ref="account_account_template_4430"/>
        <field name="property_account_expense_categ_id" ref="account_account_template_6000"/>
        <field name="property_account_income_categ_id" ref="account_account_template_5000"/>
        <field name="income_currency_exchange_account_id" ref="account_account_template_5803"/>
        <field name="expense_currency_exchange_account_id" ref="account_account_template_6803"/>
        <field name="property_stock_account_input_categ_id" ref="account_account_template_2045"/>
        <field name="property_stock_account_output_categ_id" ref="account_account_template_2045"/>
        <field name="property_stock_valuation_account_id" ref="account_account_template_2040"/>
        <field name="default_pos_receivable_account_id" ref="account_account_template_2411"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Account Fiscal Position Template -->
    <record id="account_fiscal_position_template_lt" model="account.fiscal.position.template">
        <field name="name">LT</field>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.lt"/>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
    </record>
    <record id="account_fiscal_position_template_eu" model="account.fiscal.position.template">
        <field name="name">EU</field>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_group_id" ref="base.europe"/>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="20"/>
    </record>
    <record id="account_fiscal_position_template_b2c_eu" model="account.fiscal.position.template">
        <field name="name">B2C EU</field>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="False"/>
        <field name="country_group_id" ref="base.europe"/>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="25"/>
    </record>
    <record id="account_fiscal_position_template_out_eu" model="account.fiscal.position.template">
        <field name="name">Not EU</field>
        <field name="auto_apply" eval="True"/>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="30"/>
    </record>
    <!-- Account Fiscal Position Tax Template -->
    <record id="account_fiscal_position_tax_template_eu_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="account_fiscal_position_template_eu"/>
        <field name="tax_src_id" ref="account_tax_template_sales_21"/>
        <field name="tax_dest_id" ref="account_tax_template_sales_0_vat13"/>
    </record>
    <record id="account_fiscal_position_tax_template_eu_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="account_fiscal_position_template_eu"/>
        <field name="tax_src_id" ref="account_tax_template_purchase_21"/>
        <field name="tax_dest_id" ref="account_tax_template_purchase_assumed_21"/>
    </record>
    <record id="account_fiscal_position_tax_template_out_eu_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="account_fiscal_position_template_out_eu"/>
        <field name="tax_src_id" ref="account_tax_template_sales_21"/>
        <field name="tax_dest_id" ref="account_tax_template_sales_0_vat12"/>
    </record>
    <record id="account_fiscal_position_tax_template_out_eu_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="account_fiscal_position_template_out_eu"/>
        <field name="tax_src_id" ref="account_tax_template_purchase_21"/>
        <field name="tax_dest_id" ref="account_tax_template_purchase_0"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">
    <record id="tax_group_vat_0" model="account.tax.group">
      <field name="name">VAT 0%</field>
    </record>
    <record id="tax_group_vat_5" model="account.tax.group">
      <field name="name">VAT 5%</field>
    </record>
    <record id="tax_group_vat_9" model="account.tax.group">
      <field name="name">VAT 9%</field>
    </record>
    <record id="tax_group_vat_21" model="account.tax.group">
      <field name="name">VAT 21%</field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Sales -->
    <!-- 0% VAT -->
    <record id="account_tax_template_sales_0_vat5" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Sale 0% (VAT5) Tax Exempt in LT</field>
        <field name="type_tax_use">sale</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_sales_0_vat12" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Sale 0% (VAT12) export</field>
        <field name="description">0% VAT</field>
        <field name="type_tax_use">sale</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_sales_0_vat13" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Sale 0% (VAT13)</field>
        <field name="description">0% VAT</field>
        <field name="type_tax_use">sale</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_sales_0_vat15" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Sale 0% (VAT15) Service not in EU</field>
        <field name="type_tax_use">sale</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <!-- 5% VAT -->
    <record id="account_tax_template_sales_5" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="name">Sale 5% (VAT3)</field>
        <field name="description">5% VAT</field>
        <field name="type_tax_use">sale</field>
        <field name="amount" eval="5.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <!-- 21% VAT -->
    <record id="account_tax_template_sales_21" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Sale 21% (VAT1)</field>
        <field name="description">21% VAT</field>
        <field name="type_tax_use">sale</field>
        <field name="amount" eval="21.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <!-- Reversed Taxes -->
    <record id="account_tax_template_sales_reversed_21" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Reversed Sale 21% (VAT25)</field>
        <field name="description">21% VAT</field>
        <field name="type_tax_use">sale</field>
        <field name="amount" eval="21.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <!-- Purchases -->
    <!-- 0% VAT -->
    <record id="account_tax_template_purchase_0_vat5" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Purchase 0% (VAT5) Tax Exempt LT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_purchase_0_vat14" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Purchase 0% (VAT14) export, transportation</field>
        <field name="description">0% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_purchase_0" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Purchase 0% (VAT15)</field>
        <field name="description">0% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_purchase_0_vat42" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Purchase 0% (VAT42)</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_purchase_0_vat100" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="name">Purchase 0% (VAT100) other cases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="0.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <!-- 5% VAT -->
    <record id="account_tax_template_purchase_5" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="name">Purchase 5% (VAT3)</field>
        <field name="description">5% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="5.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <!-- 9% VAT -->
    <record id="account_tax_template_purchase_not_deductible_9" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_9"/>
        <field name="name">Purchase 9% (VAT2) not deductible</field>
        <field name="description">9% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="9.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_63081'),
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
                'account_id': ref('account_account_template_63081'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_purchase_9" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_9"/>
        <field name="name">Purchase 9% (VAT2)</field>
        <field name="description">9% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="9.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <!-- 21% VAT -->
    <record id="account_tax_template_purchase_not_deductible_21" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Purchase 21% (VAT1) not deductible</field>
        <field name="description">21% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="21.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_63081'),
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
                'account_id': ref('account_account_template_63081'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_purchase_21" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Purchase 21% (VAT1)</field>
        <field name="description">21% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="21.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
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
                'account_id': ref('account_account_template_2441'),
            }),
        ]"/>
    </record>
    <!-- Reversed Taxes -->
    <record id="account_tax_template_purchase_reversed_21" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Reversed Purchase 21% (VAT25)</field>
        <field name="description">21% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="21.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="90"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_4492'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <!-- Assumed Taxes -->
    <!-- 21% (VAT16) -->
    <record id="account_tax_template_purchase_assumed_21_vat16" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Assumed Purchase 21% (VAT16) from EU</field>
        <field name="description">Assumed 21% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="100"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_2441'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <!-- 21% (VAT20) -->
    <record id="account_tax_template_purchase_assumed_21_vat20" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Assumed Purchase 21% (VAT20)</field>
        <field name="description">Assumed 21% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="21.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="110"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_2441'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
    <!-- 21% (VAT21) -->
    <record id="account_tax_template_purchase_assumed_21" model="account.tax.template">
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="name">Assumed Purchase 21% (VAT21) Goods/Services</field>
        <field name="description">Assumed 21% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="21.0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="account_chart_template_lithuania"/>
        <field name="sequence" eval="120"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_2441'),
            }),
             (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
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
                'account_id': ref('account_account_template_2441'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('account_account_template_4492'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- LT BUSINESS ACCOUNTING STANDARD (BAS) REPORTS MENU ITEM -->
    <menuitem id="account_reports_lt_statements_menu"
        name="Lithuania"
        parent="account.menu_finance_reports"
        sequence="1"/>
</odoo>

```

## File: data\res_bank_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">
    <record id="res_bank_siauliu" model="res.bank">
        <field name="name">AB Šiaulių bankas</field>
        <field name="bic">CBSBLT26</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Šiauliai</field>
        <field name="street">Tilžės g.149</field>
        <field name="zip">LT-76348</field>
        <field name="phone">+370 4 1595607</field>
        <field name="email">info@sb.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_citadele" model="res.bank">
        <field name="name">AB "Citadele" bankas</field>
        <field name="bic">INDULT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">K. Kalinausko g. 13</field>
        <field name="zip">LT-03107</field>
        <field name="phone">+370 5 2664600</field>
        <field name="email">info@citadele.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_danske" model="res.bank">
        <field name="name">Danske bank A/S Lietuvos filialas</field>
        <field name="bic">SMPOLT22</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Saltoniškių g. 2</field>
        <field name="zip">LT-08500</field>
        <field name="phone">+370 5 215666</field>
        <field name="email">info@danskebank.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_luminor" model="res.bank">
        <field name="name">Luminor Bank AB</field>
        <field name="bic">AGBLLT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Konstitucijos pr. 21A</field>
        <field name="zip">LT-08130</field>
        <field name="phone">+370 5 2393444</field>
        <field name="email">info@luminor.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_paysera" model="res.bank">
        <field name="name">UAB "Paysera LT"</field>
        <field name="bic">EVIULT21</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Mėnulio g. 7</field>
        <field name="zip">LT-04326</field>
        <field name="phone">+370 5 2071558</field>
        <field name="email">info@paysera.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_seb" model="res.bank">
        <field name="name">AB SEB bankas</field>
        <field name="bic">CBVILT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Gedimino pr. 12,</field>
        <field name="zip">LT-01103</field>
        <field name="phone">+370 5 2682800</field>
        <field name="email">info@seb.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_swedbank" model="res.bank">
        <field name="name">"Swedbank", AB</field>
        <field name="bic">HABALT22</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Konstitucijos pr. 20A</field>
        <field name="zip">LT-03502</field>
        <field name="phone">+370 5 2684444</field>
        <field name="email">info@swedbak.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_medicinos" model="res.bank">
        <field name="name">UAB Medicinos bankas</field>
        <field name="bic">MDBALT22</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Pamėnkalnio g. 40</field>
        <field name="zip">LT-01114</field>
        <field name="phone">+370 800 60700</field>
        <field name="email">info@medbank.lt</field>
    </record>
    <record id="res_bank_nordea" model="res.bank">
        <field name="name">Nordea Bank AB</field>
        <field name="bic">NDEALT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Didžioji g. 18</field>
        <field name="zip">LT-01128</field>
        <field name="phone">+370 5 2361361</field>
        <field name="email">info@nordea.lt</field>
        <field name="active" eval="1"/>
    </record>
</odoo>

```

## File: i18n_extra\l10n_lt.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
#   * l10n_lt
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 12.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2019-09-05 09:02+0000\n"
"PO-Revision-Date: 2019-09-05 09:02+0000\n"
"Last-Translator: <>\n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_lt
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_0
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_0_vat14
#: model:account.tax.template,description:l10n_lt.account_tax_template_sales_0_vat12
#: model:account.tax.template,description:l10n_lt.account_tax_template_sales_0_vat13
msgid "0% VAT"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_1_net_turnover
msgid "1. Net turnover"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_9_other_interest_similar_income
msgid "9. Other interest and similar income"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_10_impaired_fin_assets_short_investments
msgid "10. The impairment of the financial assets and short-term investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_11_interest_other_similar_expenses
msgid "11. Interest and other similar expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_12_tax_on_profit
msgid "12. Tax on profit"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_2_cost_of_sales
msgid "2. Cost of sales"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_not_deductible_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_reversed_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_reversed_negative_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_reversed_positive_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_sales_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_sales_reversed_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_sales_reversed_negative_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_sales_reversed_positive_21
msgid "21% VAT"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_3_adjustments_of_biological_assets
msgid "3. Fair value adjustments of the biological assets"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_5
#: model:account.tax.template,description:l10n_lt.account_tax_template_sales_5
msgid "5% VAT"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_4_selling_expenses
msgid "4. Selling expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_5_general_administrative_expenses
msgid "5. General and administrative expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_6_other_operating_results
msgid "6. Other operating results"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_7_income_investments_parent
msgid "7. Income from investments in the shares of parent, subsidiaries and associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_9
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_not_deductible_9
msgid "9% VAT"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_8_income_other_longterm_investments_loans
msgid "8. Income from other long-term investments and loans"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_1
msgid "A.1.1. Assets arising from development"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_2
msgid "A.1.2. Goodwill"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_3
msgid "A.1.3. Software"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_4
msgid "A.1.4. Concessions, patents, licenses, trade marks and similar rights"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_5
msgid "A.1.5. Other intangible assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_6
msgid "A.1.6. Advance payments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_1
msgid "A.2.1. Land"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_2
msgid "A.2.2. Buildings and structures"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_3
msgid "A.2.3. Machinery and plant"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_4
msgid "A.2.4. Vehicles"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_5
msgid "A.2.5. Other equipment, fittings and tools"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_6_1
msgid "A.2.6.1. Land"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_6_2
msgid "A.2.6.2. Buildings"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_7
msgid "A.2.7. Advance payments and tangible assets under construction (production)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_1
msgid "A.3.1. Shares in entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_2
msgid "A.3.2. Loans to entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_3
msgid "A.3.3. Amounts receivable from entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_4
msgid "A.3.4. Shares in associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_5
msgid "A.3.5. Loans to associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_6
msgid "A.3.6. Amounts receivable from the associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_7
msgid "A.3.7. Long-term investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_8
msgid "A.3.8. Amounts receivable after one year"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_9
msgid "A.3.9. Other financial assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_4_1
msgid "A.4.1. Assets of the deferred tax on profit"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_4_2
msgid "A.4.2. Biological assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_4_3
msgid "A.4.3. Other assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_491
msgid "Accumulated Expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_292
msgid "Accumulative Income"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_3411
msgid "Acknowledged Net Profit (Loss) of The Accounting Year in The Profit (Loss) Report"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_3421
msgid "Acknowledged Net Profit (Loss) of The Previous Accounting Year in The Profit (Loss) Report"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1210
msgid "Acquisition Cost of Buildings"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_26200
msgid "Acquisition Cost of Equity Securities of Other Companies"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1220
msgid "Acquisition Cost of Machinery and Equipment"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1240
msgid "Acquisition Cost of Other Equipment, Appliances and Tools"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_166110
#: model:account.account.template,name:l10n_lt.account_account_template_262110
msgid "Acquisition Cost of Other Non-Equity Securities"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2010
msgid "Acquisition Cost of Raw Materials, Materials and Assembly Parts"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1230
msgid "Acquisition Cost of Vehicles"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1680
msgid "Advance Payments for Financial Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1260
msgid "Advance Payments for Long-Term Tangible Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6307
msgid "Amortization Costs of Intangible Assets Value"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1138
msgid "Amortization of Software Value (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6400
msgid "Asset Disposition Losses"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_21_vat16
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_21_vat20
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_neg_21_vat16
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_neg_21_vat20
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_negative_21
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_pos_21_vat16
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_pos_21_vat20
#: model:account.tax.template,description:l10n_lt.account_tax_template_purchase_assumed_positive_21
msgid "Assumed 21% VAT"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_21_vat16
msgid "Assumed Purchase 21% (VAT16) from EU"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_pos_21_vat16
msgid "Assumed Purchase 21% (VAT16) from EU (+)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_neg_21_vat16
msgid "Assumed Purchase 21% (VAT16) from EU (-)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_21_vat20
msgid "Assumed Purchase 21% (VAT20)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_pos_21_vat20
msgid "Assumed Purchase 21% (VAT20) (+)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_neg_21_vat20
msgid "Assumed Purchase 21% (VAT20) (-)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_21
msgid "Assumed Purchase 21% (VAT21) Goods/Services"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_positive_21
msgid "Assumed Purchase 21% (VAT21) Goods/Services (+)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_assumed_negative_21
msgid "Assumed Purchase 21% (VAT21) Goods/Services (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_1
msgid "B.1.1. Raw materials, materials and consumables"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_2
msgid "B.1.2. Production and work in progress"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_3
msgid "B.1.3. Finished goods"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_4
msgid "B.1.4. Goods for resale"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_5
msgid "B.1.5. Biological assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_6
msgid "B.1.6. Fixed tangible assets held for sale"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_7
msgid "B.1.7. Advance payments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_1
msgid "B.2.1. Trade debtors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_2
msgid "B.2.2. Amounts owed by entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_3
msgid "B.2.3. Amounts owed by associates entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_4
msgid "B.2.4. Other debtors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_3_1
msgid "B.3.1. Shares in entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_3_2
msgid "B.3.2. Other investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_4
msgid "B.4. CASH AND CASH EQUIVALENTS"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1212
msgid "Buildings being Prepared for Use"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_c_prepayments_accrued_income
msgid "C. PREPAYMENTS AND ACCRUED INCOME"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_274
msgid "Cash Equivalents"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_chart_template_lithuania_liquidity_transfer
msgid "Cash in Transit"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1201
msgid "Change of Land Value after Revaluation"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_390
msgid "Common Summary of Accounts"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_305
msgid "Company Owner’s Capital"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4486
msgid "Contributions of Payable Compulsory Health Insurance"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4483
msgid "Contributions of Payable Guarantee Fund"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4482
msgid "Contributions of Payable Social Security"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_20210
msgid "Cost of Executive Works"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6000
msgid "Cost of Goods Sold"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_20200
msgid "Cost of Incomplete Production"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2030
msgid "Cost of Production"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60030
msgid "Cost of Production Sold"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2040
msgid "Cost of Purchased Goods for Resale"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6001
msgid "Cost of Services Sold"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6202
msgid "Cost of Services and Goods Advertising"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2060
msgid "Cost of Tangible Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_12510
msgid "Cost of the Acquisition of Buildings as Investment Property"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_12500
msgid "Cost of the Acquisition of Land as Investment Property"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6901
msgid "Costs (Income) of Deferred Value-Added Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6311
#: model:account.account.template,name:l10n_lt.account_account_template_6804
msgid "Costs of Fines"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63092
msgid "Costs of Intangible Assets Value Loss"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6806
msgid "Costs of Interests for Financial Rent"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63091
msgid "Costs of Inventory Value Loss"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6301
msgid "Costs of Maintenance and Exploitation"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6802
msgid "Costs of Other Loan Interests"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63083
msgid "Costs of Other Taxes"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63080
msgid "Costs of Real Estate Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63090
msgid "Costs of Value Loss of Debts from Customers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6701
msgid "Costs of Value Loss of Long-Term Financial Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63093
msgid "Costs of Value Loss of Long-Term Tangible Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63094
msgid "Costs of Value Loss of Other Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6702
msgid "Costs of Value Loss of Short-Term Investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63081
msgid "Costs of not Deductible Value-Added Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_1_1
msgid "D.1.1. Authorized (subscribed) or primary capital"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_1_2
msgid "D.1.2. Subscribed capital unpaid (–)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_1_3
msgid "D.1.3. Own shares (–)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_2_share_premium
msgid "D.2. SHARE PREMIUM ACCOUNT"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_3_revaluation_reserve
msgid "D.3. REVALUATION RESERVE"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_4_1
msgid "D.4.1. Compulsory reserve or emergency (reserve) capital"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_4_2
msgid "D.4.2. Reserve for acquiring own shares"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_4_3
msgid "D.4.3. Other reserves"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_5_1
msgid "D.5.1. Profit (loss) for the reporting year "
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_5_2
msgid "D.5.2. Profit (loss) brought forward"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63122
msgid "Daily Allowance"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_424
msgid "Debts to Suppliers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4430
msgid "Debts to Suppliers for Goods and Services"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_171
msgid "Deferred Profit Tax Asset"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2084
msgid "Deposit"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6306
msgid "Depreciation Costs of Long-Term Tangible Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1217
msgid "Depreciation of the Acquisition Cost of Buildings (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_12517
msgid "Depreciation of the Acquisition Cost of Buildings as Investment Property (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1227
msgid "Depreciation of the Acquisition Cost of Machinery and Equipment (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1247
msgid "Depreciation of the Acquisition Cost of Other Equipment, Appliances and Tools (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1237
msgid "Depreciation of the Acquisition Cost of Vehicles (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1681
msgid "Derivative Financial Assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60033
msgid "Direct Commissions"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60034
msgid "Direct Payroll of Employees Working in Production"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60032
msgid "Direct Transport"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6209
msgid "Discounts Received (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_509
#: model:account.account.template,name:l10n_lt.account_account_template_609
msgid "Discounts, Returns (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_e_grants_subsidies
msgid "E. GRANTS, SUBSIDIES"
msgstr ""

#. module: l10n_lt
#: model:account.fiscal.position.template,name:l10n_lt.account_fiscal_position_template_eu
msgid "EU"
msgstr ""

#. module: l10n_lt
#: model:account.fiscal.position.template,name:l10n_lt.account_fiscal_position_template_b2c_eu
msgid "B2C EU"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60043
msgid "Energy Resources"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_f_1
msgid "F.1. Provisions for pensions and similar obligations"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_f_2
msgid "F.2. Provisions for taxation"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_f_3
msgid "F.3. Other provisions"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_279
msgid "Frozen Funds (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60044
msgid "Fuel"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6302
msgid "Fuel Costs"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_291
msgid "Future Expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_1
msgid "G.1.1. Debenture loans"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_2
msgid "G.1.2. Amounts owed to credit institutions"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_3
msgid "G.1.3. Payments received on account"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_4
msgid "G.1.4. Trade creditors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_5
msgid "G.1.5. Amounts payable under the bills and checks "
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_6
msgid "G.1.6. Amounts payable to the entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_7
msgid "G.1.7. Amounts payable to the associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_8
msgid "G.1.8. Other amounts payable and long-term liabilities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_1
msgid "G.2.1. Debenture loans"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_10
msgid "G.2.10. Other amounts payable and short-term liabilities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_2
msgid "G.2.2. Amounts owed to credit institutions"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_3
msgid "G.2.3. Payments received on account"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_4
msgid "G.2.4. Trade creditors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_5
msgid "G.2.5. Amounts payable under the bills and checks "
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_6
msgid "G.2.6. Amounts payable to the entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_7
msgid "G.2.7. Amounts payable to the associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_8
msgid "G.2.8. Liabilities of tax on profit"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_9
msgid "G.2.9. Liabilities related to employment relations"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_h_accruals_deferred_income
msgid "H. ACCRUALS AND DEFERRED INCOME"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5804
msgid "Income from Fines"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5000
msgid "Income from Goods Sold"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5802
msgid "Income from Other Loan Interests"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5600
msgid "Income from Other Long-Term Investments and Loan Interests"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5003
msgid "Income from Production Sold"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5001
msgid "Income from Services Sold"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_492
msgid "Income of Future Periods"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6005
msgid "Increase (Decrease) in Inventories"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6303
msgid "Insurance Costs"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_999
msgid "Interim Account"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6809
msgid "Investment Disposition Losses"
msgstr ""

#. module: l10n_lt
#: model:account.fiscal.position.template,name:l10n_lt.account_fiscal_position_template_lt
msgid "LT"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1200
msgid "Land Acquisition Cost"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4211
msgid "Leasing (Financial Rent) or Other Obligations"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4485
msgid "Leave Liabilities"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4410
msgid "Liabilities under Short-Term Loan Agreements"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4220
msgid "Long-Term Liabilities under Loan Agreements"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1222
msgid "Machinery and Equipment being Prepared for Use"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60042
msgid "Maintenance of Production Transport"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60041
msgid "Maintenance of Working Tools for Production"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_331
msgid "Mandatory or Reserve Capital"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6803
msgid "Negative Currency Exchange Difference"
msgstr ""

#. module: l10n_lt
#: model:account.fiscal.position.template,name:l10n_lt.account_fiscal_position_template_out_eu
msgid "Not EU"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4470
msgid "Obligations of Profit Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_3011
msgid "Ordinary Shares"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6401
msgid "Other Costs"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63120
msgid "Other Costs and Bank Taxes"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6208
msgid "Other Costs of Sales"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4214
msgid "Other Debt Obligations"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4431
msgid "Other Debts to Suppliers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1242
msgid "Other Equipment, Appliances and Tools being Prepared for Use"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5401
msgid "Other Income"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4221
#: model:account.account.template,name:l10n_lt.account_account_template_4471
msgid "Other Obligations"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4413
msgid "Other Obligations (Executables and etc.)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4493
msgid "Other Payable Taxes to Budget"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4494
msgid "Other Payables"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_428
msgid "Other Payables and Long-Term Liabilities"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_413
msgid "Other Provisions"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2449
msgid "Other Questionable Debts (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_303
msgid "Own Shares (Yawn) (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_308
msgid "Owners’ Contributions"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4411
msgid "Part of Debts to Credit Institutions in The Current Year"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4401
msgid "Part of Leasing (Financial Rent) or Other Obligations in The Current Year"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4403
msgid "Part of Other Long-Term Debts in The Current Year"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4491
msgid "Payable Bonuses"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4490
msgid "Payable Dividends"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4481
msgid "Payable Personal Income Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4480
msgid "Payable Salary"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4492
msgid "Payable Value-Added Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4495
msgid "Payables to the Owners of the Company"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_24472
msgid "Payed Funds for Personal Use to the Owners of the Company"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6304
msgid "Payroll and Other Costs of Employees Working in Administration"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6203
msgid "Payroll and Other Costs of Employees Working in Sales"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63082
msgid "Pollution Tax Costs"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5803
msgid "Positive Currency Exchange Difference"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_3012
msgid "Preference Shares"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2442
msgid "Prepaid Profit Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4230
#: model:account.account.template,name:l10n_lt.account_account_template_4420
msgid "Prepayments from Customers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_4231
#: model:account.account.template,name:l10n_lt.account_account_template_4421
msgid "Prepayments from Service Receivers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2080
msgid "Prepayments to Suppliers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2036
msgid "Production at Third Parties"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2035
msgid "Production in Transit"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_3424
msgid "Profit (Loss) of Material Corrections of The Previous Years"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6900
msgid "Profit and Other Similar Taxes of The Current Year"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5400
msgid "Profit from Asset Disposition"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_5809
msgid "Profit from Disposition of Investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_24471
msgid "Profit paid in Advance to the Owners of the Company"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6310
msgid "Provisions Costs"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_411
msgid "Provisions for Pensions and Similar Liabilities"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_0_vat100
msgid "Purchase 0% (VAT100) other cases"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_0_vat14
msgid "Purchase 0% (VAT14) export, transportation"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_0
msgid "Purchase 0% (VAT15)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_0_vat42
msgid "Purchase 0% (VAT42)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_0_vat5
msgid "Purchase 0% (VAT5) Tax Exempt LT"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_21
msgid "Purchase 21% (VAT1)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_not_deductible_21
msgid "Purchase 21% (VAT1) not deductible"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_5
msgid "Purchase 5% (VAT3)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_9
msgid "Purchase 9% (VAT2)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_not_deductible_9
msgid "Purchase 9% (VAT2) not deductible"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2046
msgid "Purchased Goods for Resale at Third Parties"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2045
msgid "Purchased Goods for Resale in Transit"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2012
msgid "Raw Materials, Materials and Assembly Parts at Third Parties"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2011
msgid "Raw Materials, Materials and Assembly Parts in Transit"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1672
msgid "Receivable Amount of Leasing (Financial Rent) after One Year"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2441
msgid "Receivable Value-Added Tax"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6300
msgid "Rental Costs"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_6805
msgid "Representation and other disallowed deductions"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63121
msgid "Representative Costs"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_reversed_21
msgid "Reversed Purchase 21% (VAT25)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_reversed_positive_21
msgid "Reversed Purchase 21% (VAT25) (+)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_purchase_reversed_negative_21
msgid "Reversed Purchase 21% (VAT25) (-)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_reversed_21
msgid "Reversed Sale 21% (VAT25)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_reversed_positive_21
msgid "Reversed Sale 21% (VAT25) (+)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_reversed_negative_21
msgid "Reversed Sale 21% (VAT25) (-)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_0_vat12
msgid "Sale 0% (VAT12) export"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_0_vat13
msgid "Sale 0% (VAT13)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_0_vat15
msgid "Sale 0% (VAT15) Service not in EU"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_0_vat5
msgid "Sale 0% (VAT5) Tax Exempt in LT"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_21
msgid "Sale 21% (VAT1)"
msgstr ""

#. module: l10n_lt
#: model:account.tax.template,name:l10n_lt.account_tax_template_sales_5
msgid "Sale 5% (VAT3)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60031
msgid "Short-Term Production Inventory"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1130
msgid "Software Acquisition Cost"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_302
msgid "Subscribed Capital Unpaid (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_63123
msgid "Support"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2443
msgid "Tax Overpayments"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_412
msgid "Tax Provisions"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1665
#: model:account.account.template,name:l10n_lt.account_account_template_2623
msgid "Term Deposits"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_3412
msgid "Unacknowledged Net Profit (Loss) of The Accounting Year in The Profit (Loss) Report"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_3422
msgid "Unacknowledged Net Profit (Loss) of The Previous Accounting Year in The Profit (Loss) Report"
msgstr ""

#. module: l10n_lt
#: model:account.tax.group,name:l10n_lt.tax_group_vat_0
msgid "VAT 0%"
msgstr ""

#. module: l10n_lt
#: model:account.tax.group,name:l10n_lt.tax_group_vat_5
msgid "VAT 5%"
msgstr ""

#. module: l10n_lt
#: model:account.tax.group,name:l10n_lt.tax_group_vat_9
msgid "VAT 9%"
msgstr ""

#. module: l10n_lt
#: model:account.tax.group,name:l10n_lt.tax_group_vat_21
msgid "VAT 21%"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2444
msgid "VSDF (Lithuanian State Social Insurance Fund) Debt to the Company"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_12519
msgid "Value Loss of Buildings as Investment Property (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_26209
msgid "Value Loss of Equity Securities of Other Companies (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_166119
#: model:account.account.template,name:l10n_lt.account_account_template_262119
msgid "Value Loss of Other Non-Equity Securities (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2039
msgid "Value Loss of Production (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2049
msgid "Value Loss of Purchased Goods for Resale (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2019
msgid "Value Loss of Raw Materials, Materials and Assembly Parts (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_12509
msgid "Value Loss of the Land as Investment Property (-)"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_2410
msgid "Value of Debts from Customers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_16710
#: model:account.account.template,name:l10n_lt.account_account_template_24400
msgid "Value of Loans Granted"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_24460
msgid "Value of Other Receivable Debts"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_16700
msgid "Value of Receivable Debts from Customers"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_16740
msgid "Value of Receivables"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_24450
msgid "Value of Receivables from Accountable Persons"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_1232
msgid "Vehicles being Prepared for Use"
msgstr ""

#. module: l10n_lt
#: model:account.account.template,name:l10n_lt.account_account_template_60040
msgid "Working Tools for Production"
msgstr ""

```

