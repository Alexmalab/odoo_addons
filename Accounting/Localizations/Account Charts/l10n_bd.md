# Odoo Module: l10n_bd

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Bangladesh - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['bd'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': ' This is the base module to manage chart of accounts and localization for the Bangladesh ',
    'depends': [
        'account',
    ],
    'data': [
        'data/account.account.tag.csv',
        'data/res.country.state.csv',
        'data/account_tax_report_data.xml',
        'views/menu_items.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
"id","name"
"account_tags_tax_c_exempt","Bangladesh Corporate Tax - Exempt Income"
"account_tags_tax_c_cog","Bangladesh Corporate Tax - Cost of Goods Sold"
"account_tags_tax_c_exp","Bangladesh Corporate Tax - Expense"

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tr_form" model="account.report">
        <field name="name">Bangladesh Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.bd"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tr_column_net" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
            <record id="tr_column_tax" model="account.report.column">
                <field name="name">Tax Amount</field>
                <field name="expression_label">taxbalance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tr_title_purchases_vat" model="account.report.line">
                <field name="name">Purchases Taxes</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tr_line_P_EX_0" model="account.report.line">
                        <field name="name">Imports</field>
                        <field name="code">PUR_EX0</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_EX_0_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_EX_0 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_0" model="account.report.line">
                        <field name="name">Purchases at 0% VAT</field>
                        <field name="code">PUR0</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_0_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_0 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_2" model="account.report.line">
                        <field name="name">Purchases at 2% VAT</field>
                        <field name="code">PUR2</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_2_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_2 Base</field>
                            </record>
                            <record id="tr_expression_P_2_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_2 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_45" model="account.report.line">
                        <field name="name">Purchases at 4.5% VAT</field>
                        <field name="code">PUR45</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_45_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_45 Base</field>
                            </record>
                            <record id="tr_expression_P_45_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_45 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_5" model="account.report.line">
                        <field name="name">Purchases at 5% VAT</field>
                        <field name="code">PUR5</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_5_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_5 Base</field>
                            </record>
                            <record id="tr_expression_P_5_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_5 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_75" model="account.report.line">
                        <field name="name">Purchases at 7.5% VAT</field>
                        <field name="code">PUR75</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_75_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_75 Base</field>
                            </record>
                            <record id="tr_expression_P_75_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_75 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_10" model="account.report.line">
                        <field name="name">Purchases at 10% VAT</field>
                        <field name="code">PUR10</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_10_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_10 Base</field>
                            </record>
                            <record id="tr_expression_P_10_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_10 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_15" model="account.report.line">
                        <field name="name">Purchases at 15% VAT</field>
                        <field name="code">PUR15</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_15_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_15 Base</field>
                            </record>
                            <record id="tr_expression_P_15_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">P_15 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tr_title_sales_vat" model="account.report.line">
                <field name="name">Sales Taxes</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tr_line_S_EX_0" model="account.report.line">
                        <field name="name">Exports</field>
                        <field name="code">SAL_EX0</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_EX_0_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_EX_0 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_S_0" model="account.report.line">
                        <field name="name">Sales at 0% VAT</field>
                        <field name="code">SAL0</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_0_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_0 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_S_2" model="account.report.line">
                        <field name="name">Sales at 2% VAT</field>
                        <field name="code">SAL2</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_2_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_2 Base</field>
                            </record>
                            <record id="tr_expression_S_2_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_2 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_S_45" model="account.report.line">
                        <field name="name">Sales at 4.5% VAT</field>
                        <field name="code">SAL45</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_45_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_45 Base</field>
                            </record>
                            <record id="tr_expression_S_45_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_45 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_S_5" model="account.report.line">
                        <field name="name">Sales at 5% VAT</field>
                        <field name="code">SAL5</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_5_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_5 Base</field>
                            </record>
                            <record id="tr_expression_S_5_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_5 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_S_75" model="account.report.line">
                        <field name="name">Sales at 7.5% VAT</field>
                        <field name="code">SAL75</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_75_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_75 Base</field>
                            </record>
                            <record id="tr_expression_S_75_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_75 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_S_10" model="account.report.line">
                        <field name="name">Sales at 10% VAT</field>
                        <field name="code">SAL10</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_10_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_10 Base</field>
                            </record>
                            <record id="tr_expression_S_10_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_10 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_S_15" model="account.report.line">
                        <field name="name">Sales at 15% VAT</field>
                        <field name="code">SAL15</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_15_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_15 Base</field>
                            </record>
                            <record id="tr_expression_S_15_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">S_15 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tr_title_net_vat_due" model="account.report.line">
                <field name="name">Net VAT Due</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tr_line_S_net_vat" model="account.report.line">
                        <field name="name">Sales VAT Received (Payable)</field>
                        <field name="code">l10n_bd_S_net_vat</field>
                        <field name="expression_ids">
                            <record id="tr_expression_S_net_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL2.taxbalance + SAL45.taxbalance + SAL5.taxbalance 
                                                      + SAL75.taxbalance + SAL10.taxbalance + SAL15.taxbalance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tr_line_P_net_vat" model="account.report.line">
                        <field name="name">Purchases VAT Paid (Recoverable)</field>
                        <field name="code">l10n_bd_P_net_vat</field>
                        <field name="expression_ids">
                            <record id="tr_expression_P_net_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PUR2.taxbalance + PUR45.taxbalance + PUR5.taxbalance 
                                                      + PUR75.taxbalance + PUR10.taxbalance + PUR15.taxbalance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\res.country.state.csv

```csv
"id","country_id/id","name","code"
state_bd_a,base.bd,"Barishal","BD-A"
state_bd_b,base.bd,"Chattogram","BD-B"
state_bd_c,base.bd,"Dhaka","BD-C"
state_bd_d,base.bd,"Khulna","BD-D"
state_bd_e,base.bd,"Rajshahi","BD-E"
state_bd_f,base.bd,"Rangpur","BD-F"
state_bd_g,base.bd,"Sylhet","BD-G"
state_bd_h,base.bd,"Mymensingh","BD-H"

```

## File: data\template\account.account-bd.csv

```csv
"id","name","reconcile","tag_ids","code","account_type"
"l10n_bd_100101","Liquidity Transfer","True","","100101","asset_current"
"l10n_bd_100102","Bank Suspense Account","True","","100102","asset_current"
"l10n_bd_100103","Outstanding Receipts","True","","100103","asset_current"
"l10n_bd_100104","Outstanding Payments","True","","100104","asset_current"
"l10n_bd_100106","Credit cards","True","","100106","asset_current"
"l10n_bd_100107","Post Dated Cheques Received","True","","100107","asset_current"
"l10n_bd_100201","Account Receivable","True","","100201","asset_receivable"
"l10n_bd_100202","Accounts Receivable (PoS)","True","","100202","asset_receivable"
"l10n_bd_100203","Other Receivable","False","","100203","asset_current"
"l10n_bd_100301","Deposit - Office Rent","False","","100301","asset_current"
"l10n_bd_100302","Deposits - Customs","False","","100302","asset_current"
"l10n_bd_100303","Deposit to Immigration (Visa)","False","","100303","asset_current"
"l10n_bd_100304","Deposit Others","False","","100304","asset_current"
"l10n_bd_100401","Prepaid Medical Insurance","False","","100401","asset_current"
"l10n_bd_100402","Prepaid Life Insurance","False","","100402","asset_current"
"l10n_bd_100403","Prepaid Office Rent","False","","100403","asset_current"
"l10n_bd_100404","Prepaid Other Insurance","False","","100404","asset_current"
"l10n_bd_100405","Prepaid License Fees","False","","100405","asset_current"
"l10n_bd_100406","Prepaid Maintenance","False","","100406","asset_current"
"l10n_bd_100407","Prepaid Employees Housing","False","","100407","asset_current"
"l10n_bd_100408","Prepaid Schooling Fees","False","","100408","asset_current"
"l10n_bd_100409","Prepaid Consultancy Fees","False","","100409","asset_current"
"l10n_bd_100410","Prepaid Legal Fees","False","","100410","asset_current"
"l10n_bd_100411","Prepaid Sponsorship Fees","False","","100411","asset_current"
"l10n_bd_100412","PrePaid Advertisement Expenses","False","","100412","asset_current"
"l10n_bd_100413","Prepaid Bank Guarantee","False","","100413","asset_current"
"l10n_bd_100414","Prepaid Finance charge for Loans","False","","100414","asset_current"
"l10n_bd_100415","Other Prepayments","False","","100415","asset_current"
"l10n_bd_100501","Handling Difference in Inventory","False","","100501","asset_current"
"l10n_bd_100502","Inventory Valuation","False","","100502","asset_current"
"l10n_bd_100503","Stock Incoming","False","","100503","asset_current"
"l10n_bd_100504","Stock Outgoing","False","","100504","asset_current"
"l10n_bd_100601","Acc. Depreciation of Motor Vehicles","False","","100601","asset_fixed"
"l10n_bd_100602","Amortisation on Leasehold Improvement","False","","100602","asset_fixed"
"l10n_bd_100603","Leasehold Improvement","False","","100603","asset_fixed"
"l10n_bd_100604","Furniture and Equipment","False","","100604","asset_fixed"
"l10n_bd_100605","Computer Hardware & Software","False","","100605","asset_fixed"
"l10n_bd_100606","Acc.Deprn.of Furniture & Office Equipment","False","","100606","asset_fixed"
"l10n_bd_100607","Acc. Deprn.Computer Hardware & Software","False","","100607","asset_fixed"
"l10n_bd_100701","Registration of Trademarks","False","","100701","asset_current"
"l10n_bd_100801","Right of use Asset (IFRS 16)","False","","100801","asset_fixed"
"l10n_bd_100802","Accumulated Depreciation right use asset (IFRS 16)","False","","100802","asset_fixed"
"l10n_bd_200101","Account Payable","True","","200101","liability_payable"
"l10n_bd_200102","Trade Payables","True","","200102","liability_payable"
"l10n_bd_200103","Employees Payables","True","","200103","liability_payable"
"l10n_bd_200104","Credit Notes to Customers","False","","200104","liability_current"
"l10n_bd_200201","Accrued - Salaries","False","","200201","liability_current"
"l10n_bd_200202","Accrued - Commissions","False","","200202","liability_current"
"l10n_bd_200203","Accrued-Staff Bonus","False","","200203","liability_current"
"l10n_bd_200204","Accrued Other Personnel Cost","False","","200204","liability_current"
"l10n_bd_200205","Accrued - Sponsorship","False","","200205","liability_current"
"l10n_bd_200301","Accrued - Utilities","False","","200301","liability_current"
"l10n_bd_200302","Accrued - Telephone","False","","200302","liability_current"
"l10n_bd_200303","Accrued - Audit Fees","False","","200303","liability_current"
"l10n_bd_200304","Accrued - Office Rent","False","","200304","liability_current"
"l10n_bd_200305","Accrued Others","False","","200305","liability_current"
"l10n_bd_200306","Accrued Bangladesh Customs","False","","200306","liability_current"
"l10n_bd_200401","Deferred income","False","","200401","liability_current"
"l10n_bd_200501","Leave Tickets Provision","False","","200501","liability_non_current"
"l10n_bd_200502","Leave Days Provision","False","","200502","liability_non_current"
"l10n_bd_200503","End of Service Provision","False","","200503","liability_non_current"
"l10n_bd_200504","Income Tax Provision","False","","200504","liability_non_current"
"l10n_bd_200901","VAT Input","True","","200901","asset_current"
"l10n_bd_200902","VAT Output","True","","200902","liability_current"
"l10n_bd_200903","VAT Receivable","True","","200903","asset_non_current"
"l10n_bd_200904","VAT Payable","True","","200904","liability_non_current"
"l10n_bd_200905","Tax Payable","True","","200905","liability_current"
"l10n_bd_200906","Tax Receivable","True","","200906","asset_current"
"l10n_bd_300100","Retained Earnings","False","","300100","equity"
"l10n_bd_300200","Undistributed Profits/Losses","False","","300200","equity_unaffected"
"l10n_bd_400100","Income Clearing Account","False","","400100","income"
"l10n_bd_400101","Sales Account","False","","400101","income"
"l10n_bd_400102","Sales of I/C","False","","400102","income"
"l10n_bd_400103","Sales from Other Region","False","","400103","income"
"l10n_bd_400104","Management Consultancy Fees","False","","400104","income"
"l10n_bd_400105","Advertising Income","False","","400105","income"
"l10n_bd_400201","income_other","False","","400201","income_other"
"l10n_bd_400301","Gain On Difference Of Exchange","False","","400301","income_other"
"l10n_bd_400302","Cash Difference Gain","False","","400302","income_other"
"l10n_bd_400303","Excess In Till","False","","400303","income_other"
"l10n_bd_400304","Cash Discount Gain","False","","400304","income_other"
"l10n_bd_500101","Cost of Goods Sold in Trading","False","l10n_bd.account_tags_tax_c_cog","500101","expense_direct_cost"
"l10n_bd_500102","Cost Of Goods Sold I/C Sales","False","l10n_bd.account_tags_tax_c_cog","500102","expense_direct_cost"
"l10n_bd_500200","Expense Clearing Account","False","l10n_bd.account_tags_tax_c_exp","500200","expense"
"l10n_bd_500201","Medical Insurance","False","l10n_bd.account_tags_tax_c_exp","500201","expense"
"l10n_bd_500202","End Of Service Indemnity","False","l10n_bd.account_tags_tax_c_exp","500202","expense"
"l10n_bd_500203","Sponsorship Fees","False","l10n_bd.account_tags_tax_c_exp","500203","expense"
"l10n_bd_500301","Basic Salary","False","l10n_bd.account_tags_tax_c_exp","500301","expense"
"l10n_bd_500302","Housing Allowance","False","l10n_bd.account_tags_tax_c_exp","500302","expense"
"l10n_bd_500303","Transportation Allowance","False","l10n_bd.account_tags_tax_c_exp","500303","expense"
"l10n_bd_500304","Leave Ticket","False","l10n_bd.account_tags_tax_c_exp","500304","expense"
"l10n_bd_500305","Leave Salary","False","l10n_bd.account_tags_tax_c_exp","500305","expense"
"l10n_bd_500306","Sales Commission","False","l10n_bd.account_tags_tax_c_exp","500306","expense"
"l10n_bd_500307","Visa Expenses","False","l10n_bd.account_tags_tax_c_exp","500307","expense"
"l10n_bd_500308","Staff Other Allowances","False","l10n_bd.account_tags_tax_c_exp","500308","expense"
"l10n_bd_500309","Air tickets","False","l10n_bd.account_tags_tax_c_exp","500309","expense"
"l10n_bd_500401","Office Rent","False","l10n_bd.account_tags_tax_c_exp","500401","expense"
"l10n_bd_500402","Warehouse Rent","False","l10n_bd.account_tags_tax_c_exp","500402","expense"
"l10n_bd_500403","Water & Electricity","False","l10n_bd.account_tags_tax_c_exp","500403","expense"
"l10n_bd_500404","Other Utility Cahrges","False","l10n_bd.account_tags_tax_c_exp","500404","expense"
"l10n_bd_500501","Audit Fees","False","l10n_bd.account_tags_tax_c_exp","500501","expense"
"l10n_bd_500502","Legal fees","False","l10n_bd.account_tags_tax_c_exp","500502","expense"
"l10n_bd_500503","Trade License Fees","False","l10n_bd.account_tags_tax_c_exp","500503","expense"
"l10n_bd_500504","Others - Professional Fees","False","l10n_bd.account_tags_tax_c_exp","500504","expense"
"l10n_bd_500505","Insurance","False","l10n_bd.account_tags_tax_c_exp","500505","expense"
"l10n_bd_500506","Previous Year Adjustments Account","False","l10n_bd.account_tags_tax_c_exp","500506","expense"
"l10n_bd_500601","Credit Card Charges","False","l10n_bd.account_tags_tax_c_exp","500601","expense"
"l10n_bd_500602","Other Bank Charges","False","l10n_bd.account_tags_tax_c_exp","500602","expense"
"l10n_bd_500603","Bank Finance & Loan Charges","False","l10n_bd.account_tags_tax_c_exp","500603","expense"
"l10n_bd_500651","Income Tax Expense","False","","500651","expense"
"l10n_bd_500701","Other - Advertising Expenses","False","l10n_bd.account_tags_tax_c_exp","500701","expense"
"l10n_bd_500702","Training","False","l10n_bd.account_tags_tax_c_exp","500702","expense"
"l10n_bd_500703","Consultancy Fees","False","l10n_bd.account_tags_tax_c_exp","500703","expense"
"l10n_bd_500801","Amortisation on Leasehold Improvement","False","l10n_bd.account_tags_tax_c_exp","500801","expense_depreciation"
"l10n_bd_500802","Vehicle Expenses","False","l10n_bd.account_tags_tax_c_exp","500802","expense_depreciation"
"l10n_bd_500803","Depreciation Of Motor Vehicles","False","l10n_bd.account_tags_tax_c_exp","500803","expense_depreciation"
"l10n_bd_500804","Depreciation Of Furniture & Office Equipment","False","l10n_bd.account_tags_tax_c_exp","500804","expense_depreciation"
"l10n_bd_500805","Depreciation Of Computer Hard & Soft","False","l10n_bd.account_tags_tax_c_exp","500805","expense_depreciation"
"l10n_bd_500851","Depreciation on right of use asset (IFRS 16)","False","l10n_bd.account_tags_tax_c_exp","500851","expense_depreciation"
"l10n_bd_500901","Loss On Fixed Assets Disposal","False","l10n_bd.account_tags_tax_c_exp","500901","expense"
"l10n_bd_500902","Cash Shortage","False","l10n_bd.account_tags_tax_c_exp","500902","expense"
"l10n_bd_500903","Loss on Difference on Exchange","False","l10n_bd.account_tags_tax_c_exp","500903","expense"
"l10n_bd_500904","Write Off Receivables & Payables","False","l10n_bd.account_tags_tax_c_exp","500904","expense"
"l10n_bd_500905","Write Off Inventory","False","l10n_bd.account_tags_tax_c_exp","500905","expense"
"l10n_bd_500906","Others - Provision & Write off","False","l10n_bd.account_tags_tax_c_exp","500906","expense"
"l10n_bd_500907","Others","False","l10n_bd.account_tags_tax_c_exp","500907","expense"
"l10n_bd_500908","Other Non Operating Expenses","False","l10n_bd.account_tags_tax_c_exp","500908","expense"
"l10n_bd_500909","Cash Difference Loss","False","l10n_bd.account_tags_tax_c_exp","500909","expense"
"l10n_bd_501101","Telephone","False","l10n_bd.account_tags_tax_c_exp","501101","expense"
"l10n_bd_501102","Others - Communication","False","l10n_bd.account_tags_tax_c_exp","501102","expense"
"l10n_bd_501103","Maintenance","False","l10n_bd.account_tags_tax_c_exp","501103","expense"
"l10n_bd_501104","Security & Guard","False","l10n_bd.account_tags_tax_c_exp","501104","expense"
"l10n_bd_501105","Cleaning","False","l10n_bd.account_tags_tax_c_exp","501105","expense"
"l10n_bd_501106","Others - Office Various Expenses","False","l10n_bd.account_tags_tax_c_exp","501106","expense"
"l10n_bd_501107","Cash Discount Loss","False","l10n_bd.account_tags_tax_c_exp","501107","expense"

```

## File: data\template\account.fiscal.position-bd.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.bd","",""
"fiscal_position_template_2","2","International","1","","","",""
"","","","","","","VAT_S_IN_BD_0","VAT_S_EX_IN_BD_0"
"","","","","","","VAT_P_IN_BD_0","VAT_P_EX_IN_BD_0"
"","","","","","","VAT_S_IN_BD_2","VAT_S_EX_IN_BD_0"
"","","","","","","VAT_P_IN_BD_2","VAT_P_EX_IN_BD_0"
"","","","","","","VAT_S_IN_BD_45","VAT_S_EX_IN_BD_0"
"","","","","","","VAT_P_IN_BD_45","VAT_P_EX_IN_BD_0"
"","","","","","","VAT_S_IN_BD_5","VAT_S_EX_IN_BD_0"
"","","","","","","VAT_P_IN_BD_5","VAT_P_EX_IN_BD_0"
"","","","","","","VAT_S_IN_BD_75","VAT_S_EX_IN_BD_0"
"","","","","","","VAT_P_IN_BD_75","VAT_P_EX_IN_BD_0"
"","","","","","","VAT_S_IN_BD_10","VAT_S_EX_IN_BD_0"
"","","","","","","VAT_P_IN_BD_10","VAT_P_EX_IN_BD_0"
"","","","","","","VAT_S_IN_BD_15","VAT_S_EX_IN_BD_0"
"","","","","","","VAT_P_IN_BD_15","VAT_P_EX_IN_BD_0"

```

## File: data\template\account.group-bd.csv

```csv
"id","code_prefix_start","code_prefix_end","name"
"l10n_bd_group_100100_100199","100100","100199","Liquidity"
"l10n_bd_group_100200_100299","100200","100299","Receivables"
"l10n_bd_group_100300_100399","100300","100399","Deposits"
"l10n_bd_group_100400_100499","100400","100499","Prepaid Expenses"
"l10n_bd_group_100500_100599","100500","100599","Inventory"
"l10n_bd_group_100600_100699","100600","100699","Company Assets & Depreciation"
"l10n_bd_group_100700_100799","100700","100799","Licensing and Copyrights"
"l10n_bd_group_100800_100899","100800","100899","IFRS16 Assets & Depreciation"
"l10n_bd_group_200100_200199","200100","200199","Payables"
"l10n_bd_group_200200_200299","200200","200299","Accrued Employee Expenses"
"l10n_bd_group_200300_200399","200300","200399","Accrued Expenses"
"l10n_bd_group_200400_200499","200400","200499","Deferrals"
"l10n_bd_group_200500_200599","200500","200599","Provisions"
"l10n_bd_group_200900_200999","200900","200999","VAT"
"l10n_bd_group_300900_300100","300900","300100","Equity"
"l10n_bd_group_400100_400199","400100","400199","Operating Income"
"l10n_bd_group_400200_400299","400200","400299","Non-Operating Income"
"l10n_bd_group_400300_400399","400300","400399","Other gains & losses - Other Income"
"l10n_bd_group_400400_400499","400400","400499","Operating Results"
"l10n_bd_group_500100_500199","500100","500199","Cost of Sales"
"l10n_bd_group_500200_500299","500200","500299","Employees Expenses"
"l10n_bd_group_500300_500399","500300","500399","Payroll Expenses"
"l10n_bd_group_500400_500499","500400","500499","Office and Location Expenses"
"l10n_bd_group_500500_500599","500500","500599","Company Expenses"
"l10n_bd_group_500600_500649","500600","500649","Finance Expenses"
"l10n_bd_group_500650_500699","500650","500699","Income Tax"
"l10n_bd_group_500700_500799","500700","500799","Misc. Company Expenses"
"l10n_bd_group_500800_500849","500800","500849","Assets Depreciation Expenses"
"l10n_bd_group_500851_500899","500851","500899","IFRS16 Depreciation"
"l10n_bd_group_500900_500999","500900","500999","Other gains & losses - Expenses"
"l10n_bd_group_501100_501199","501100","501199","Misc. Office Expenses"

```

## File: data\template\account.tax-bd.csv

```csv
"id","sequence","description","name","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id"
"VAT_S_IN_BD_10","1","","10%","10%","10","percent","sale","tax_group_bd_10","base","invoice","+S_10 Base",""
"","","","","","","","","","tax","invoice","+S_10 Tax","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_10 Base",""
"","","","","","","","","","tax","refund","-S_10 Tax","l10n_bd_200904"
"VAT_P_IN_BD_10","2","","10%","10%","10","percent","purchase","tax_group_bd_10","base","invoice","+P_10 Base",""
"","","","","","","","","","tax","invoice","+P_10 Tax","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_10 Base",""
"","","","","","","","","","tax","refund","-P_10 Tax","l10n_bd_200903"
"VAT_S_IN_BD_15","3","","15%","15%","15","percent","sale","tax_group_bd_15","base","invoice","+S_15 Base",""
"","","","","","","","","","tax","invoice","+S_15 Tax","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_15 Base",""
"","","","","","","","","","tax","refund","-S_15 Tax","l10n_bd_200904"
"VAT_P_IN_BD_15","4","","15%","15%","15","percent","purchase","tax_group_bd_15","base","invoice","+P_15 Base",""
"","","","","","","","","","tax","invoice","+P_15 Tax","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_15 Base",""
"","","","","","","","","","tax","refund","-P_15 Tax","l10n_bd_200903"
"VAT_S_IN_BD_0","5","","0%","0%","0","percent","sale","tax_group_bd_0","base","invoice","+S_0 Base",""
"","","","","","","","","","tax","invoice","","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_0 Base",""
"","","","","","","","","","tax","refund","","l10n_bd_200904"
"VAT_P_IN_BD_0","6","","0%","0%","0","percent","purchase","tax_group_bd_0","base","invoice","+P_0 Base",""
"","","","","","","","","","tax","invoice","","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_0 Base",""
"","","","","","","","","","tax","refund","","l10n_bd_200903"
"VAT_S_IN_BD_2","7","","2%","2%","2","percent","sale","tax_group_bd_2","base","invoice","+S_2 Base",""
"","","","","","","","","","tax","invoice","+S_2 Tax","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_2 Base",""
"","","","","","","","","","tax","refund","-S_2 Tax","l10n_bd_200904"
"VAT_P_IN_BD_2","8","","2%","2%","2","percent","purchase","tax_group_bd_2","base","invoice","+P_2 Base",""
"","","","","","","","","","tax","invoice","+P_2 Tax","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_2 Base",""
"","","","","","","","","","tax","refund","-P_2 Tax","l10n_bd_200903"
"VAT_S_IN_BD_45","9","","4.5%","4.50%","4.5","percent","sale","tax_group_bd_45","base","invoice","+S_45 Base",""
"","","","","","","","","","tax","invoice","+S_45 Tax","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_45 Base",""
"","","","","","","","","","tax","refund","-S_45 Tax","l10n_bd_200904"
"VAT_P_IN_BD_45","10","","4.5%","4.50%","4.5","percent","purchase","tax_group_bd_45","base","invoice","+P_45 Base",""
"","","","","","","","","","tax","invoice","+P_45 Tax","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_45 Base",""
"","","","","","","","","","tax","refund","-P_45 Tax","l10n_bd_200903"
"VAT_S_IN_BD_75","11","","7.5%","7.50%","7.5","percent","sale","tax_group_bd_75","base","invoice","+S_75 Base",""
"","","","","","","","","","tax","invoice","+S_75 Tax","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_75 Base",""
"","","","","","","","","","tax","refund","-S_75 Tax","l10n_bd_200904"
"VAT_P_IN_BD_75","12","","7.5%","7.50%","7.5","percent","purchase","tax_group_bd_75","base","invoice","+P_75 Base",""
"","","","","","","","","","tax","invoice","+P_75 Tax","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_75 Base",""
"","","","","","","","","","tax","refund","-P_75 Tax","l10n_bd_200903"
"VAT_S_IN_BD_5","13","","5%","5%","5","percent","sale","tax_group_bd_5","base","invoice","+S_5 Base",""
"","","","","","","","","","tax","invoice","+S_5 Tax","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_5 Base",""
"","","","","","","","","","tax","refund","-S_5 Tax","l10n_bd_200904"
"VAT_P_IN_BD_5","14","","5%","5%","5","percent","purchase","tax_group_bd_5","base","invoice","+P_5 Base",""
"","","","","","","","","","tax","invoice","+P_5 Tax","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_5 Base",""
"","","","","","","","","","tax","refund","-P_5 Tax","l10n_bd_200903"
"VAT_S_EX_IN_BD_0","15","","0% EX","0%","0","percent","sale","tax_group_bd_0","base","invoice","+S_EX_0 Base",""
"","","","","","","","","","tax","invoice","","l10n_bd_200904"
"","","","","","","","","","base","refund","-S_EX_0 Base",""
"","","","","","","","","","tax","refund","","l10n_bd_200904"
"VAT_P_EX_IN_BD_0","16","","0% EX","0%","0","percent","purchase","tax_group_bd_0","base","invoice","+P_EX_0 Base",""
"","","","","","","","","","tax","invoice","","l10n_bd_200903"
"","","","","","","","","","base","refund","-P_EX_0 Base",""
"","","","","","","","","","tax","refund","","l10n_bd_200903"

```

## File: data\template\account.tax.group-bd.csv

```csv
"id","country_id","name","tax_payable_account_id","tax_receivable_account_id"
"tax_group_bd_0","base.bd","VAT at 0%","l10n_bd_200904","l10n_bd_200903"
"tax_group_bd_2","base.bd","VAT at 2%","l10n_bd_200904","l10n_bd_200903"
"tax_group_bd_45","base.bd","VAT at 4.5%","l10n_bd_200904","l10n_bd_200903"
"tax_group_bd_5","base.bd","VAT at 5%","l10n_bd_200904","l10n_bd_200903"
"tax_group_bd_75","base.bd","VAT at 7.5%","l10n_bd_200904","l10n_bd_200903"
"tax_group_bd_10","base.bd","VAT at 10%","l10n_bd_200904","l10n_bd_200903"
"tax_group_bd_15","base.bd","VAT at 15%","l10n_bd_200904","l10n_bd_200903"

```

## File: models\template_bd.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bd')
    def _get_bd_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_bd_100201',
            'property_account_payable_id': 'l10n_bd_200101',
            'property_account_expense_categ_id': 'l10n_bd_500200',
            'property_account_income_categ_id': 'l10n_bd_400100'
        }

    @template('bd', 'res.company')
    def _get_bd_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.bd',
                'bank_account_code_prefix': '10010',
                'cash_account_code_prefix': '10010',
                'account_default_pos_receivable_account_id': 'l10n_bd_100202',
                'account_journal_suspense_account_id': 'l10n_bd_100102',
                'account_journal_payment_debit_account_id': 'l10n_bd_100103',
                'account_journal_payment_credit_account_id': 'l10n_bd_100104',
                'default_cash_difference_income_account_id': 'l10n_bd_400302',
                'default_cash_difference_expense_account_id': 'l10n_bd_500909',
                'income_currency_exchange_account_id': 'l10n_bd_400301',
                'expense_currency_exchange_account_id': 'l10n_bd_500903',
                'transfer_account_id': 'l10n_bd_100101',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_bd_501107',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_bd_400304',
                'account_sale_tax_id': 'VAT_S_IN_BD_10',
                'account_purchase_tax_id': 'VAT_P_IN_BD_10'
            },
        }

    @template('bd', 'account.journal')
    def _get_bd_account_journal(self):
        return {
            "tax_adjustment": {
                "name": "Tax Adjustments",
                "code": "TA",
                "type": "general",
                "show_on_dashboard": True,
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_bd

```

## File: views\menu_items.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_bd_corporate_report_menu" name="Bangladesh" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>
</odoo>

```

