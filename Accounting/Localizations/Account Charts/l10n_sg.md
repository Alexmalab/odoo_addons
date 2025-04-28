# Odoo Module: l10n_sg

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 Tech Receptives (<http://techreceptives.com>).
from . import models

def _preserve_tag_on_taxes(env):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(env, 'l10n_sg')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Singapore - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/singapore.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['sg'],
    'author': 'Tech Receptives',
    'version': '2.2',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Singapore accounting chart and localization.
=======================================================

This module add, for accounting:
 - The Chart of Accounts of Singapore
 - Field UEN (Unique Entity Number) on company and partner
 - Field PermitNo and PermitNoDate on invoice

    """,
    'depends': [
        'account_qr_code_emv',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/l10n_sg_chart_data.xml',
        'data/account_tax_report_data.xml',
        'views/account_invoice_view.xml',
        'views/res_bank_views.xml',
        'views/res_company_view.xml',
        'views/res_partner_view.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_preserve_tag_on_taxes',
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Singapore Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.sg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_supplies" model="account.report.line">
                <field name="name">Supplies</field>
                <field name="aggregation_formula">BOX1.balance + BOX2.balance + BOX3.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_std_rated_supplies" model="account.report.line">
                        <field name="name">Box 1 - Total value of standard-rated supplies</field>
                        <field name="code">BOX1</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_std_rated_supplies_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 1</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_zero_rated_supplies" model="account.report.line">
                        <field name="name">Box 2 - Total value of zero-rated supplies</field>
                        <field name="code">BOX2</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_zero_rated_supplies_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 2</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_exempt_supplies" model="account.report.line">
                        <field name="name">Box 3 - Total value of exempt supplies</field>
                        <field name="code">BOX3</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_exempt_supplies_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 3</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_total_supplies" model="account.report.line">
                        <field name="name">Box 4 - Total value of (1) + (2) + (3)</field>
                        <field name="aggregation_formula">BOX1.balance + BOX2.balance + BOX3.balance</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_purchases" model="account.report.line">
                <field name="name">Taxable Purchases and Imports</field>
                <field name="aggregation_formula">BOX_5.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_total_taxable_purchases" model="account.report.line">
                        <field name="name">Box 5 - Total value of taxable purchases</field>
                        <field name="code">BOX_5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_total_taxable_purchases_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 5</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_taxes" model="account.report.line">
                <field name="name">Taxes</field>
                <field name="aggregation_formula">BOX6.balance + BOX7.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_output_tax_due" model="account.report.line">
                        <field name="name">Box 6 - Output tax due</field>
                        <field name="code">BOX6</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_output_tax_due_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 6</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_inp_tax_refund_claim" model="account.report.line">
                        <field name="name">Box 7 - Less : Input tax and refunds claimed</field>
                        <field name="code">BOX7</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_inp_tax_refund_claim_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 7</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_total_gst_paid_iras" model="account.report.line">
                        <field name="name">Box 8 - Equals : Net GST to be paid to IRAS</field>
                        <field name="code">BOX_8</field>
                        <field name="aggregation_formula">BOX6.balance - BOX7.balance</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_applicable" model="account.report.line">
                <field name="name">Applicable to Taxable Persons under Major Exported Scheme / Approved 3rd Party Logistics Company / Other Approved Schemes Only</field>
                <field name="aggregation_formula">BOX_9.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_applicable_goods_imported_value" model="account.report.line">
                        <field name="name">Box 9 - Total value of goods imported under this scheme</field>
                        <field name="code">BOX_9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_applicable_goods_imported_value_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 9</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_claims" model="account.report.line">
                <field name="name">Did you make the following claims in Box 7?</field>
                <field name="aggregation_formula">BOX_10.balance + BOX_11.balance + BOX_12.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_claim_refunded_to_tourist" model="account.report.line">
                        <field name="name">Box 10 - Did you claim for GST you had refunded to tourist?</field>
                        <field name="code">BOX_10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_claim_refunded_to_tourist_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 10</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_claim_reverse_charge" model="account.report.line">
                        <field name="name">Box 11 - Did you make any bad debt relief claims and/or refund claims for reverse charge transactions?</field>
                        <field name="code">BOX_11</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_claim_reverse_charge_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 11</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_claim_pre_registration" model="account.report.line">
                        <field name="name">Box 12 - Did you make any pre-registration input tax claims?</field>
                        <field name="code">BOX_12</field>
                        <field name="code">BOX_12</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_claim_pre_registration_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 12</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_revenues" model="account.report.line">
                <field name="name">Revenue</field>
                <field name="aggregation_formula">BOX_13.balance + BOX_14.balance + BOX_15.balance + BOX_16.balance + BOX_17.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_revenues_accounting_period" model="account.report.line">
                        <field name="name">Box 13 - Revenue for the accounting period</field>
                        <field name="code">BOX_13</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_revenues_accounting_period_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">REV.balance</field>
                                <field name="subformula">cross_report</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_revenues_reverse_charge" model="account.report.line">
                        <field name="name">Box 14 - Did you import services and/or low-value goods subject to GST under Reverse Charge?</field>
                        <field name="code">BOX_14</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_revenues_reverse_charge_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 14</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_revenues_electronic_marketplace" model="account.report.line">
                        <field name="name">Box 15 - Did you operate an electronic marketplace to supply remote services subject to GST on behalf on third-party suppliers?</field>
                        <field name="code">BOX_15</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_revenues_electronic_marketplace_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 15</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_revenues_electronic_marketplace_redeliver" model="account.report.line">
                        <field name="name">Box 16 - Did you operate as a redeliverer, or an electronic marketplace to supply imported low-value goods subject to GST on behalf of third-party suppliers?</field>
                        <field name="code">BOX_16</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_revenues_electronic_marketplace_redeliver_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 16</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_revenues_low_value_good_supply" model="account.report.line">
                        <field name="name">Box 17 - Did you make your own supply of imported low-value goods that is subject to GST?</field>
                        <field name="code">BOX_17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_revenues_low_value_good_supply_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 17</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_deferment" model="account.report.line">
                <field name="name">Import GST Deferment Scheme</field>
                <field name="aggregation_formula">BOX_20.balance + BOX_21.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_deferment_net_gst" model="account.report.line">
                        <field name="name">Box 18 - Net GST per box 8 above</field>
                        <field name="code">BOX_18</field>
                        <field name="aggregation_formula">BOX_8.balance</field>
                    </record>
                    <record id="account_tax_report_line_deferment_import_gst" model="account.report.line">
                        <field name="name">Box 19 - Add: Deferred Import GST payable</field>
                        <field name="code">BOX_19</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deferment_import_gst_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 19</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_deferment_total_18_19" model="account.report.line">
                        <field name="name">Box 20 - Total Value of (18) + (19)</field>
                        <field name="code">BOX_20</field>
                        <field name="aggregation_formula">BOX_18.balance + BOX_19.balance</field>
                    </record>
                    <record id="account_tax_report_line_deferment_total_good_imported" model="account.report.line">
                        <field name="name">Box 21 - Total value of goods imported under this scheme</field>
                        <field name="code">BOX_21</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deferment_total_good_imported_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Box 21</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_sg_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_sg_statements_menu" name="Singapore" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\template\account.account-sg.csv

```csv
"id","name","code","account_type","reconcile"
"account_account_696","Allowance for Bad Debts","101110","asset_current","False"
"account_account_697","Development Costs","104100","asset_non_current","False"
"account_account_698","Employee Cash Advances","101120","asset_current","False"
"account_account_699","Inventory","101130","asset_current","False"
"account_account_700","Investments - Other","101140","asset_current","False"
"account_account_701","Loans to Officers","101150","asset_current","False"
"account_account_702","Loans to Others","101160","asset_current","False"
"account_account_703","Loans to Shareholders","101170","asset_current","False"
"account_account_704","Prepaid Expenses","101180","asset_current","False"
"account_account_705","Retainage","101190","asset_current","False"
"account_account_706","Undeposited Funds","101200","asset_current","False"
"account_account_707","Other Current Assets","101210","asset_current","False"
"account_account_857","GST Receivable/Refund","101231","asset_current","False"
"account_account_858","Other Tax Receivable/Refund","101232","asset_current","False"
"account_account_709","Accumulated Amortisation","103100","expense_depreciation","False"
"account_account_710","Accumulated Depreciation","103110","expense_depreciation","False"
"account_account_711","Accumulated Depletion","103120","expense_depreciation","False"
"account_account_712","Buildings","102100","asset_fixed","False"
"account_account_713","Depletable Assets","102110","asset_fixed","False"
"account_account_714","Furniture and Fixtures","102120","asset_fixed","False"
"account_account_715","Leasehold Improvements","102130","asset_fixed","False"
"account_account_716","Machinery and Equipment","102140","asset_fixed","False"
"account_account_717","Vehicles","102150","asset_fixed","False"
"account_account_718","Other Assets","102160","asset_fixed","False"
"account_account_720","Intangible Assets","104110","asset_non_current","False"
"account_account_721","Accumulated Amortization of Non-current Assets","103130","expense_depreciation","False"
"account_account_722","Available-for-sale Financial Assets","101220","asset_current","False"
"account_account_723","Deferred Tax Assets","101230","asset_current","False"
"account_account_724","Goodwill","104120","asset_non_current","False"
"account_account_725","Investments","104130","asset_non_current","False"
"account_account_726","Lease Buyout","104140","asset_non_current","False"
"account_account_727","Licences","104150","asset_non_current","False"
"account_account_728","Organisational Costs","104160","asset_non_current","False"
"account_account_729","Other Intangible Assets","104170","asset_non_current","False"
"account_account_730","Other Non-current Assets","104180","asset_non_current","False"
"account_account_731","Prepayments and Accrued Income","101240","asset_current","False"
"account_account_732","Security Deposits","101250","asset_current","False"
"account_account_735","Trade Receivable Account","100010","asset_receivable","True"
"account_account_736","Other Receivable Account","100020","asset_receivable","True"
"account_account_737","Trade Receivable Account (PoS)","100030","asset_receivable","True"
"account_account_738","Input Tax & Refunds Claimed","101260","asset_current","False"
"account_account_752","Current Portion of Employee Benefits Obligations","201000","liability_current","False"
"account_account_862","CPF Employer Liability","201001","liability_current","False"
"account_account_863","CPF Employer Liability","201002","liability_current","False"
"account_account_864","SDL Employer Liability","201003","liability_current","False"
"account_account_753","Current Portion of Obligations under Finance Leases","201010","liability_current","False"
"account_account_754","Current Tax Liability","201020","liability_current","False"
"account_account_755","Insurance Payable","201030","liability_current","False"
"account_account_756","Interest Payables","201040","liability_current","False"
"account_account_757","Line of Credit","201050","liability_current","False"
"account_account_758","Loan Payable","201060","liability_current","False"
"account_account_759","Payroll Clearing","201070","liability_current","True"
"account_account_760","Payroll Liabilities","201080","liability_current","False"
"account_account_761","Prepaid Expenses Payable","201090","liability_current","False"
"account_account_762","Provision for Warranty Obligations","201100","liability_current","False"
"account_account_763","Rents in Trust - Liability","201110","liability_current","False"
"account_account_764","GST Payable","201120","liability_current","False"
"account_account_765","Short Term Borrowings","201130","liability_current","False"
"account_account_766","Client Trust Accounts - Liabilities","201140","liability_current","False"
"account_account_768","Accruals and Deferred Income","201150","liability_current","False"
"account_account_769","Bank Loans","202000","liability_non_current","False"
"account_account_770","Obligations under Finance Leases","201160","liability_current","False"
"account_account_771","Long Term Borrowings","202010","liability_non_current","False"
"account_account_772","Long Term Employee Benefit Obligations","202020","liability_non_current","False"
"account_account_773","Notes Payable","202030","liability_non_current","False"
"account_account_774","Shareholder Notes Payable","202040","liability_non_current","False"
"account_account_775","Other Non-current Liabilities","202050","liability_non_current","False"
"account_account_777","Trade Payable Account","200010","liability_payable","True"
"account_account_778","Other Payable Account","200020","liability_payable","True"
"account_account_780","Partner's Equity","203000","equity","False"
"account_account_781","Accumulated Adjustment","203010","equity","False"
"account_account_782","Ordinary Shares","203020","equity","False"
"account_account_783","Opening Balance Equity","203030","equity","False"
"account_account_784","Owner's Equity","203040","equity","False"
"account_account_785","Paid-in Capital or Surplus","203050","equity","False"
"account_account_786","Preferred Shares","203060","equity","False"
"account_account_787","Retained Earnings","203070","equity","False"
"account_account_788","Share Capital","203080","equity","False"
"account_account_789","Treasury Shares","203090","equity","False"
"account_account_791","Output Tax Due","201170","liability_current","False"
"account_account_800","Discounts/Refunds -Given","300001","income","False"
"account_account_801","Non-profit Revenue","300002","income","False"
"account_account_802","Other Primary Revenue","300003","income","False"
"account_account_803","Sales of Product Revenue","300004","income","False"
"account_account_804","Service/Fee Revenue","300005","income","False"
"account_account_805","Unapplied Cash Payment Income","300006","income","False"
"account_account_807","Dividend Revenue","301001","income_other","False"
"account_account_808","Gain/Loss on Sale of Fixed Assets or Investments","301002","income_other","False"
"account_account_809","Interest Earned","301003","income_other","False"
"account_account_810","Other Investment Revenue","301004","income_other","False"
"account_account_811","Other Miscellaneous Revenue","301005","income_other","False"
"account_account_812","Tax-exempt Interest","301006","income_other","False"
"account_account_815","Cost of Labour - COS","401001","expense_direct_cost","False"
"account_account_816","Equipment Rental - COS","401002","expense_direct_cost","False"
"account_account_817","Freight and Delivery - COS","401003","expense_direct_cost","False"
"account_account_818","Other Costs of Sales - COS","401004","expense_direct_cost","False"
"account_account_819","Supplies and Materials - COS","401005","expense_direct_cost","False"
"account_account_822","Administrative Expenses","501001","expense","False"
"account_account_823","Advertising/Promotional","501002","expense","False"
"account_account_824","Auto","501003","expense","False"
"account_account_825","Bad Debts","501004","expense","False"
"account_account_826","Bank Charges","501005","expense","False"
"account_account_827","Charitable Contributions","501006","expense","False"
"account_account_828","Cost of Labour","501007","expense","False"
"account_account_829","Distribution Costs","501008","expense","False"
"account_account_830","Dues and Subscriptions","501009","expense","False"
"account_account_831","Entertainment","501010","expense","False"
"account_account_832","Meals and Entertainment","501011","expense","False"
"account_account_833","Equipment Rental","501012","expense","False"
"account_account_834","Finance Costs","501013","expense","False"
"account_account_835","Insurance","501014","expense","False"
"account_account_836","Interest Paid","501015","expense","False"
"account_account_837","Legal and Professional Fees","501016","expense","False"
"account_account_838","Other Miscellaneous Service Cost","501017","expense","False"
"account_account_839","Payroll Expenses","501018","expense","False"
"account_account_840","Promotional Meals","501019","expense","False"
"account_account_841","Rent or Lease of Buildings","501020","expense","False"
"account_account_842","Repair and Maintenance","501021","expense","False"
"account_account_843","Shipping, Freight, and Delivery","501022","expense","False"
"account_account_844","Supplies","501023","expense","False"
"account_account_845","Taxes Paid","501024","expense","False"
"account_account_846","Travel","501025","expense","False"
"account_account_847","Travel Meals","501026","expense","False"
"account_account_848","Utilities","501027","expense","False"
"account_account_864","Employee Expense","501029","expense","False"
"account_account_849","Unapplied Cash Bill Payment Expense","501028","expense","False"
"account_account_859","CPF Employer Expense","501100","expense","False"
"account_account_860","CPF Employer Expense","501101","expense","False"
"account_account_861","SDL Employer Expense","501102","expense","False"
"account_account_851","Amortisation","502001","expense","False"
"account_account_852","Depreciation","502002","expense","False"
"account_account_853","Exchange Gain or Loss","502003","expense","False"
"account_account_854","Other Expense","502004","expense","False"
"account_account_855","Penalties and Settlements","502005","expense","False"
"account_account_856","Discounts Taken","502006","expense","False"

```

## File: data\template\account.tax-sg.csv

```csv
"id","name","active","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/use_in_tax_closing"
"sg_sale_tax_sr_8","8% SR","False","Standard Rated - Local supply of goods and services","8% SR","8.0","percent","sale","tax_group_8","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_sr_9","9% SR","True","Standard Rated - Local supply of goods and services","9% SR","9.0","percent","sale","tax_group_9","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_es_0","0% ES33","True","0% Regulation 33 Exempt Supplies -specific categories of exempt supplies listed in Regulation 33 of the GST (General) Regulations","0% ES33","0.0","percent","sale","tax_group_exempt","100","base","invoice","+Box 3","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","-Box 3","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_sale_tax_ds_7","7% DS","False","Deemed - Supplies required to be reported under the GST legislation","7% DS","7.0","percent","sale","tax_group_7","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_ds_8","8% DS","False","Deemed - Supplies required to be reported under the GST legislation","8% DS","8.0","percent","sale","tax_group_8","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_ds_9","9% DS","True","Deemed - Supplies required to be reported under the GST legislation","9% DS","9.0","percent","sale","tax_group_deemed","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_esn_0","0% ESN33","True","0% Non-Regulation 33 Exempt Supplies - Exempt supplies other than those listed ion Regulation 33 of the GST (General) Regulations","0% ESN33","0.0","percent","sale","tax_group_0","100","base","invoice","+Box 3","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","-Box 3","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_sale_tax_os_0","0% OS","False","Out-of-Scope - Supplies outside the scope of the GST Act","0% OS","0.0","percent","sale","tax_group_oos","100","base","invoice","","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_sale_tax_zr_0","0% ZR","True","Supplies involving goods that are exported or provision of international services","0% ZR","0.0","percent","sale","tax_group_0","100","base","invoice","+Box 2","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","-Box 2","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_sale_tax_sr_7","7% SR","False","Standard Rated - Local supply of goods and services","7% SR","7.0","percent","sale","tax_group_7","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_srca_s_na","0% SRCA-S","False","Local supply of prescribed good (i.e. mobile phones) subject to customer accounting","SRCA-S","0.0","percent","sale","tax_group_0","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_sale_tax_srca_c_7","7% SRCA-C","False","Local supply of prescribed good (i.e. mobile phones), where the GST-required customer is required to account for the supply on behalf of the supplier","7% SRCA-C","7.0","percent","sale","tax_group_7","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_srca_c_8","8% SRCA-C","False","Local supply of prescribed good (i.e. mobile phones), where the GST-required customer is required to account for the supply on behalf of the supplier","8% SRCA-C","8.0","percent","sale","tax_group_8","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_srca_c_9","9% SRCA-C","True","Local supply of prescribed good (i.e. mobile phones), where the GST-required customer is required to account for the supply on behalf of the supplier","9% SRCA-C","9.0","percent","sale","tax_group_9","100","base","invoice","+Box 1","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_srrc_9","9% SRRC","True","Imported services & LCG accountable by the customer under the reverse charge mechanism","9% SRRC","9.0","percent","sale","tax_group_rc","100","base","invoice","+Box 1||+Box 14","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1||-Box 14","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_srovr_rs_9","9% SROVR-RS","False","Supply of remote services accountable by the electronic marketplace under the Overseas Vendor Registration Scheme","9% SROVR-RS","9.0","percent","sale","tax_group_9","100","base","invoice","+Box 1||+Box 15","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1||-Box 15","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_srovr_lvg_9","9% SROVR-LVG","False","Supply of LVG accountable by the re-deliverer or electronic marketplace on behalf of third-party suppliers","9% SROVR-LVG","9.0","percent","sale","tax_group_9","100","base","invoice","+Box 1||+Box 16","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1||-Box 16","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_sale_tax_srlvg_9","9% SRLVG","True","Own supply of LVG - GS","9% SRLVG","9.0","percent","sale","tax_group_9","100","base","invoice","+Box 1||+Box 17","","True"
"","","","","","","","","","100","tax","invoice","+Box 6","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1||-Box 17","","True"
"","","","","","","","","","100","tax","refund","-Box 6","account_account_791","True"
"sg_purchase_tax_tx8_8","8% TX8","False","Rated - Purchases from GST-registered suppliers that are subject to 8%","8% TX8","8.0","percent","purchase","tax_group_8","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_tx8_9","9% TX","True","Standard Rated - Purchases from GST-registered suppliers that are subject to 9%","9% TX","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txn33_7","7% TX-N33","False","Purchases from GST-registered suppliers that are subject to GST and are directly attributable to the making of Non-Regulation 33 exempt supplies","7% TX-N33","7.0","percent","purchase","tax_group_7","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txn33_8","8% TX-N33","False","Purchases from GST-registered suppliers that are subject to GST and are directly attributable to the making of Non-Regulation 33 exempt supplies","8% TX-N33","8.0","percent","purchase","tax_group_8","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txn33_9","9% TX-N33","True","Purchases from GST-registered suppliers that are subject to GST and are directly attributable to the making of Non-Regulation 33 exempt supplies","9% TX-N33","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txe33_7","7% TX-E33","False","Standard-rated purchases of directly attributable to Regulation 33 exempt supplies - purchases from GST-registered supplies that are subject to GST and are directly attributable to the making of Regulation 33 exempt supplies","7% TX-E33","7.0","percent","purchase","tax_group_7","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txe33_8","8% TX-E33","False","Standard-rated purchases of directly attributable to Regulation 33 exempt supplies - purchases from GST-registered supplies that are subject to GST and are directly attributable to the making of Regulation 33 exempt supplies","8% TX-E33","8.0","percent","purchase","tax_group_8","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txe33_9","9% TX-E33","False","Standard-rated purchases of directly attributable to Regulation 33 exempt supplies - purchases from GST-registered supplies that are subject to GST and are directly attributable to the making of Regulation 33 exempt supplies","9% TX-E33","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_bl_7","7% BL","False","Disallowed Expenses - Purchases where GST is incurred but is specifically not claimable","7% BL","7.0","percent","purchase","tax_group_7","100","base","invoice","","","False"
"","","","","","","","","","100","tax","invoice","","account_account_738","False"
"","","","","","","","","","100","base","refund","","","False"
"","","","","","","","","","100","tax","refund","","account_account_738","False"
"sg_purchase_tax_bl_8","8% BL","False","Disallowed Expenses - Purchases where GST is incurred but is specifically not claimable","8% BL","8.0","percent","purchase","tax_group_8","100","base","invoice","","","False"
"","","","","","","","","","100","tax","invoice","","account_account_738","False"
"","","","","","","","","","100","base","refund","","","False"
"","","","","","","","","","100","tax","refund","","account_account_738","False"
"sg_purchase_tax_bl_9","9% BL","True","Disallowed Expenses - Purchases where GST is incurred but is specifically not claimable","9% BL","9.0","percent","purchase","tax_group_de","100","base","invoice","","","False"
"","","","","","","","","","100","tax","invoice","","account_account_738","False"
"","","","","","","","","","100","base","refund","","","False"
"","","","","","","","","","100","tax","refund","","account_account_738","False"
"sg_purchase_tax_im_7","7% IM","False","Import of Goods - GST paid to Singapore Customs on the import of goods into Singapore","7% IM","7.0","percent","purchase","tax_group_7","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_im_8","8% IM","False","Import of Goods - GST paid to Singapore Customs on the import of goods into Singapore","8% IM","8.0","percent","purchase","tax_group_8","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_im_9","9% IM","True","Import of Goods - GST paid to Singapore Customs on the import of goods into Singapore","9% IM","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txre_7","7% TX-RE","False","Purchases from GST-registered suppliers that are subject to GST and are either att","7% TX-RE","7.0","percent","purchase","tax_group_7","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txre_8","8% TX-RE","False","Purchases from GST-registered suppliers that are subject to GST and are either att","8% TX-RE","8.0","percent","purchase","tax_group_8","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txre_9","9% TX-RE","True","Purchases from GST-registered suppliers that are subject to GST and are either att","9% TX-RE","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_me_0","7% IGDS","False","Import GST Deferment Scheme - Imports where the GST payment is deferred until the filing of the GST return","7% IGDS","7.0","percent","purchase","tax_group_7","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_me_08","8% IGDS","False","Import GST Deferment Scheme - Imports where the GST payment is deferred until the filing of the GST return","8% IGDS","8.0","percent","purchase","tax_group_8","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_igds_9","9% IGDS","False","Import GST Deferment Scheme - Imports where the GST payment is deferred until the filing of the GST return","9% IGDS","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5||+Box 21","","True"
"","","","","","","","","","100","tax","invoice","+Box 7||+Box 19","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5||-Box 21","","True"
"","","","","","","","","","100","tax","refund","-Box 7||-Box 19","account_account_738","True"
"sg_purchase_tax_nr_0","0% NR","True","Non-GST Registered Suppliers - purchases from non-GST registered suppliers","0% NR","0.0","percent","purchase","tax_group_supplies","100","base","invoice","","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_purchase_tax_zp_0","0% ZP","True","0% Zero-rated Purchases - purchases from GST-registered suppliers that are subject to GST at 0%","0% ZP","0.0","percent","purchase","tax_group_0","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_purchase_tax_op_0","0% OP","True","Out-of-scope - supplies outside the scope of the GST Act","0% OP","0.0","percent","purchase","tax_group_0","100","base","invoice","","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_purchase_tax_ep_0","0% EP","False","Exempt - purchases specifically exempt from GST","0% EP","0.0","percent","purchase","tax_group_0","100","base","invoice","","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_purchase_tax_mes_0","0% ME","False","Major Exporter Scheme - imports where the GST payable is suspended","0% ME","0.0","percent","purchase","tax_group_0","100","base","invoice","+Box 9||+Box 5","","True"
"","","","","","","","","","100","tax","invoice","","","True"
"","","","","","","","","","100","base","refund","-Box 9||-Box 5","","True"
"","","","","","","","","","100","tax","refund","","","True"
"sg_purchase_tax_tx7_7","7% TX7","False","Rated - Purchases from GST-registered suppliers that are subject to 7%","7% TX7","7.0","percent","purchase","tax_group_7","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txca_7","7% TXCA","False","Standard-rated purchase of prescribed goods subject to Customer Accounting","7% TXCA","7.0","percent","purchase","tax_group_7","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txca_8","8% TXCA","False","Standard-rated purchase of prescribed goods subject to Customer Accounting","8% TXCA","8.0","percent","purchase","tax_group_8","100","base","invoice","+Box 1||+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 6||+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 1||-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 6||-Box 7","account_account_738","True"
"sg_purchase_tax_txca_9","9% TXCA","True","Standard-rated purchase of prescribed goods subject to Customer Accounting","9% TXCA","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 6||+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 6||-Box 7","account_account_738","True"
"sg_purchase_tax_txrc_ts_9","9% TXRC-TS","False","Imported services & LVG directly attributable to the making of taxable supplies","9% TXRC-TS","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_im_ess_9","9% IM-ESS","False","Import of goods that are subject to GST and are directly attributable to the making of Regulation 33 exempt supplies","9% IM-ESS","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txrc_ess_9","9% TXRC-ESS","False","Imported services and LVG that are subject to reverse charge and are directly attributable to Regulation 33 exempt supplies","9% TXRC-ESS","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txrc_n33_9","9% TXRC-N33","True","Imported services & LVG that are subject to the reverse charge and are directly attibutable to Non-Regulation 33 exempt supplies","9% TXRC-N33","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_im_n33_9","9% IM-N33","True","Import of goods that are subject to GST and are directly attributaible to the making of Non-Regulation 33 exempt supplies","9% IM-N33","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_txrc_re_9","9% TXRC-RE","True","Imported services and LVG that are subject to reverse charge and are either attributable to the making of both taxable and exempt supplies or incurred for the overall running of the business","9% TXRC-RE","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_im_re_9","9% IM-RE","True","Import of goods that are subject to GST and are either attributable to the making of both taxable and exempt supplies or incurred for the overall running of the business","9% IM-RE","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 7","account_account_738","True"
"","","","","","","","","","100","base","refund","-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 7","account_account_738","True"
"sg_purchase_tax_srca_c_9","9% SRCA-C","True","Local supply of prescribed good (i.e. mobile phones), where the GST-required customer is required to account for the supply on behalf of the supplier","9% SRCA-C","9.0","percent","purchase","tax_group_9","100","base","invoice","+Box 1||+Box 5","","True"
"","","","","","","","","","100","tax","invoice","+Box 6||+Box 7","account_account_791","True"
"","","","","","","","","","100","base","refund","-Box 1||-Box 5","","True"
"","","","","","","","","","100","tax","refund","-Box 6||-Box 7","account_account_791","True"

```

## File: data\template\account.tax.group-sg.csv

```csv
"id","name","country_id","active","tax_payable_account_id","tax_receivable_account_id","advance_tax_payment_account_id"
"tax_group_0","Zero-rated","base.sg","True","account_account_764","account_account_857",""
"tax_group_exempt","Exempt","base.sg","True","account_account_778","account_account_736",""
"tax_group_oos","Out-of-Scope Supplies","base.sg","True","account_account_778","account_account_736",""
"tax_group_supplies","Supplies from Non GST registered company","base.sg","True","account_account_778","account_account_736",""
"tax_group_9","9% GST","base.sg","True","account_account_764","account_account_857","account_account_723"
"tax_group_8","TAX 8%","base.sg","False","account_account_764","account_account_857","account_account_723"
"tax_group_7","TAX 7%","base.sg","False","account_account_764","account_account_857","account_account_723"
"tax_group_deemed","Deemed Supplies","base.sg","True","account_account_764","account_account_857","account_account_723"
"tax_group_rc","Reverse Charge","base.sg","True","account_account_764","account_account_857","account_account_723"
"tax_group_de","Disallowed Expenses","base.sg","True","account_account_764","account_account_857","account_account_723"

```

## File: migrations\2.1\end-migrate_update_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'sg')], order="parent_path"):
        env['account.chart.template'].try_loading('sg', company)

```

## File: migrations\2.2\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'sg')], order="parent_path"):
        env['account.chart.template'].try_loading('sg', company)

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
from openerp.modules.registry import RegistryManager

def migrate(cr, version):
    registry = RegistryManager.get(cr.dbname)
    from openerp.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_sg')

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_sg_permit_number = fields.Char(string="Permit No.")

    l10n_sg_permit_number_date = fields.Date(string="Date of permit number")

```

## File: models\res_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    proxy_type = fields.Selection(selection_add=[('mobile', 'Mobile Number'), ('uen', 'UEN')],
                                  ondelete={'mobile': 'set default', 'uen': 'set default'})

    @api.constrains('proxy_type', 'proxy_value', 'partner_id')
    def _check_sg_proxy(self):
        for bank in self.filtered(lambda b: b.country_code == 'SG'):
            if bank.proxy_type not in ['mobile', 'uen', 'none', False]:
                raise ValidationError(_("The PayNow Type must be either Mobile or UEN to generate a PayNow QR code for account number %s.", bank.acc_number))

    @api.depends('country_code')
    def _compute_display_qr_setting(self):
        bank_sg = self.filtered(lambda b: b.country_code == 'SG')
        bank_sg.display_qr_setting = True
        super(ResPartnerBank, self - bank_sg)._compute_display_qr_setting()

    def _get_merchant_account_info(self):
        if self.country_code == 'SG':
            proxy_type_mapping = {
                'mobile': 0,
                'uen': 2,
            }
            merchant_account_vals = [
                (0, 'SG.PAYNOW'),                                           # GUID
                (1, proxy_type_mapping[self.proxy_type]),                   # Proxy Type
                (2, self.proxy_value),                                      # Proxy Value
                (3, 0),                                                     # Is Amount Editable
            ]
            merchant_account_info = ''.join([self._serialize(*val) for val in merchant_account_vals])
            return (26, merchant_account_info)
        return super()._get_merchant_account_info()

    def _get_additional_data_field(self, comment):
        if self.country_code == 'SG':
            return self._serialize(1, comment)
        return super()._get_additional_data_field(comment)

    def _get_error_messages_for_qr(self, qr_method, debtor_partner, currency):
        if qr_method == 'emv_qr' and self.country_code == 'SG':
            if currency.name not in ['SGD']:
                return _("Can't generate a PayNow QR code with a currency other than SGD.")
            return None

        return super()._get_error_messages_for_qr(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'emv_qr' and self.country_code == 'SG' and self.proxy_type not in ['mobile', 'uen']:
            return _("The PayNow Type must be either Mobile Number or UEN.")

        return super()._check_for_qr_code_errors(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models

class ResCompany(models.Model):
    _name = 'res.company'
    _description = 'Companies'
    _inherit = 'res.company'

    l10n_sg_unique_entity_number = fields.Char(string='UEN', related="partner_id.l10n_sg_unique_entity_number", readonly=False)

    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        company_vat_label = self.env.company.country_id.vat_label
        if company_vat_label:
            for node in arch.iterfind(".//field[@name='vat']"):
                node.set("string", company_vat_label)
        return arch, view

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models

class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    l10n_sg_unique_entity_number = fields.Char(string='UEN')

    def _deduce_country_code(self):
        if self.l10n_sg_unique_entity_number:
            return 'SG'
        return super()._deduce_country_code()

    def _peppol_eas_endpoint_depends(self):
        # extends account_edi_ubl_cii
        return super()._peppol_eas_endpoint_depends() + ['l10n_sg_unique_entity_number']

```

## File: models\template_sg.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('sg')
    def _get_sg_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'account_account_735',
            'property_account_payable_id': 'account_account_777',
            'property_account_expense_categ_id': 'account_account_819',
            'property_account_income_categ_id': 'account_account_803',
        }

    @template('sg', 'res.company')
    def _get_sg_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.sg',
                'bank_account_code_prefix': '10141',
                'cash_account_code_prefix': '10140',
                'transfer_account_code_prefix': '101100',
                'account_default_pos_receivable_account_id': 'account_account_737',
                'income_currency_exchange_account_id': 'account_account_853',
                'expense_currency_exchange_account_id': 'account_account_853',
                'account_journal_early_pay_discount_loss_account_id': 'account_account_800',
                'account_journal_early_pay_discount_gain_account_id': 'account_account_856',
                'account_sale_tax_id': 'sg_sale_tax_sr_9',
                'account_purchase_tax_id': 'sg_purchase_tax_tx8_9',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_sg
from . import account_move
from . import res_company
from . import res_bank
from . import res_partner

```

## File: views\account_invoice_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="view_invoice_form_l10n_sg" model="ir.ui.view">
            <field name="name">l10n_sg.invoice.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='fiscal_position_id']" position="after">
                    <field name="l10n_sg_permit_number" invisible="country_code != 'SG'"/>
                    <field name="l10n_sg_permit_number_date" invisible="country_code != 'SG'"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_bank_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_partner_bank_form_inherit_account" model="ir.ui.view">
        <field name="name">res.partner.bank.form.inherit</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="base.view_partner_bank_form"/>
        <field name="arch" type="xml">
            <field name="include_reference" position="after">
                <p invisible="country_code != 'SG'">
                    <widget name="documentation_link" path="/applications/finance/fiscal_localizations/singapore.html" label="Documentation"/>
                </p>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\res_company_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="view_company_form_l10n_sg" model="ir.ui.view">
            <field name="name">l10n_sg.company.form</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="base.view_company_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='email']" position="after">
                  <field name="l10n_sg_unique_entity_number" invisible="country_code != 'SG'"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="view_partner_form_l10n_sg" model="ir.ui.view">
            <field name="name">l10n_sg.partner.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="before">
                    <field name="l10n_sg_unique_entity_number" invisible="'SG' not in fiscal_country_codes"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

