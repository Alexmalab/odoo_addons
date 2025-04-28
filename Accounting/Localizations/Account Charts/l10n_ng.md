# Odoo Module: l10n_ng

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
    'name': "Nigeria - Accounting",
    'description': """
Nigerian localization.
=========================================================
    """,
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'version': '1.0',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ng'],
    'category': 'Accounting/Localizations/Account Charts',
    'depends': ['base_vat'],
    'data': [
        'data/tax_report.xml',
        'data/withholding_vat_report.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_ng_tax_report" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ng"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ng_tr_balance" model="account.report.column">
                <field name="name">Amount</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ng_tr_a" model="account.report.line">
                <field name="name">A - TRANSACTION SUMMARY</field>
                <field name="code">L10N_NG_A</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_ng_tr_a_5" model="account.report.line">
                        <field name="name">5. Total No of Branches</field>
                        <field name="code">L10N_NG_A5</field>
                        <field name="external_formula">integer</field>
                    </record>
                    <record id="l10n_ng_tr_a_10" model="account.report.line">
                        <field name="name">10. Total Sales/Income Exclusive of VAT</field>
                        <field name="code">L10N_NG_A10</field>
                        <field name="aggregation_formula">L10N_NG_B20.balance</field>
                    </record>
                    <record id="l10n_ng_tr_a_15" model="account.report.line">
                        <field name="name">15. Total Purchases</field>
                        <field name="code">L10N_NG_A15</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_ng_tr_b" model="account.report.line">
                <field name="name">B - SALES/ INCOME</field>
                <field name="code">L10N_NG_B</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_ng_tr_b_20" model="account.report.line">
                        <field name="name">20. Total Income Received from Sale for the Month Excluding VAT</field>
                        <field name="code">L10N_NG_B20</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">20</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_b_25" model="account.report.line">
                        <field name="name">25. Less: Value of Goods and Services Exempted Included in Line 20</field>
                        <field name="code">L10N_NG_B25</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_25_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_b_30" model="account.report.line">
                        <field name="name">30. Less: Value of Zero Rated Goods &amp; Services Included in line 20</field>
                        <field name="code">L10N_NG_B30</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_30_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">30</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_b_35" model="account.report.line">
                        <field name="name">35. Sales Adjustments (Gross amount)</field>
                        <field name="code">L10N_NG_B35</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">35</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_b_40" model="account.report.line">
                        <field name="name">40. Income Received from Sales Subject to VAT </field>
                        <field name="code">L10N_NG_B40</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">L10N_NG_B20.balance - L10N_NG_B25.balance - L10N_NG_B30.balance + L10N_NG_B35.balance</field>
                    </record>
                    <record id="l10n_ng_tr_b_45" model="account.report.line">
                        <field name="name">45. TOTAL Output Tax Collected @ 7.5%</field>
                        <field name="code">L10N_NG_B45</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_45_tax_tag" model="account.report.expression">
                                <field name="label">_tax_tags</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">45</field>
                            </record>
                            <record id="l10n_ng_tr_b_45_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_NG_B45._tax_tags - L10N_NG_B35.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_ng_tr_c" model="account.report.line">
                <field name="name">C - VAT ON PURCHASES/EXPENSES</field>
                <field name="code">L10N_NG_C</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_ng_tr_c_50" model="account.report.line">
                        <field name="name">50. Payments for Domestic Purchases other than zero rated and exempted goods and services For the Month</field>
                        <field name="code">L10N_NG_C50</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_50_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">50</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_55" model="account.report.line">
                        <field name="name">55. Payments for Domestic Purchases for Zero Rated Goods</field>
                        <field name="code">L10N_NG_C55</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_55_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">55</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_60" model="account.report.line">
                        <field name="name">60. Total Domestic Purchases Subject to Input Tax</field>
                        <field name="code">L10N_NG_C60</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">L10N_NG_C50.balance + L10N_NG_C55.balance</field>
                    </record>
                    <record id="l10n_ng_tr_c_65" model="account.report.line">
                        <field name="name">65. Payment for Imported Goods For the Month</field>
                        <field name="code">L10N_NG_C65</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_65_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">65</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_70" model="account.report.line">
                        <field name="name">70. TOTAL Purchases Subject to Input Tax</field>
                        <field name="code">L10N_NG_C70</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">L10N_NG_C60.balance + L10N_NG_C65.balance</field>
                    </record>
                    <record id="l10n_ng_tr_c_75" model="account.report.line">
                        <field name="name">75. Total Input Tax Paid Line 70 @ 7.5%</field>
                        <field name="code">L10N_NG_C75</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_75_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">75</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_80" model="account.report.line">
                        <field name="name">80. VAT Payable /(Credit) for Current Month</field>
                        <field name="code">L10N_NG_C80</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">L10N_NG_B45.balance - L10N_NG_C75.balance</field>
                    </record>
                    <record id="l10n_ng_tr_c_85" model="account.report.line">
                        <field name="name">85. Less VAT deducted at source (by MDAs &amp; Oil and Gas) Current Month</field>
                        <field name="code">L10N_NG_C85</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_85_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">85</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_90" model="account.report.line">
                        <field name="name">90. Less Automatic/Electronic VAT Payment in Current Month</field>
                        <field name="code">L10N_NG_C90</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_b_90_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">90</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_95" model="account.report.line">
                        <field name="name">95. Net VAT Payable/(Refundable) Current Month</field>
                        <field name="code">L10N_NG_C95</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">L10N_NG_C80.balance - L10N_NG_C85.balance - L10N_NG_C90.balance</field>
                    </record>
                    <!-- the value should be either negative or equal to 0 -->
                    <record id="l10n_ng_tr_c_100" model="account.report.line">
                        <field name="name">100. Previous Unrelieved VAT Credit Brought Forward</field>
                        <field name="code">L10N_NG_C100</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_c_100_balance_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <!-- tax tag to populate the very first credit brought forward -->
                            <record id="l10n_ng_tr_100_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">100</field>
                            </record>
                            <record id="l10n_ng_tr_c_100_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_NG_C100.tag + L10N_NG_C100._applied_carryover_balance</field>
                            </record>
                        </field>
                    </record>
                    <!-- this one should be either negative, meaning there is something to claim -->
                    <!-- or equal to zero -->
                    <record id="l10n_ng_tr_c_105" model="account.report.line">
                        <field name="name">105. Total VAT Credit claimable</field>
                        <field name="code">L10N_NG_C105</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_c_105_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_NG_C95.balance + L10N_NG_C100.balance</field>
                                <field name="subformula">if_below(NGN(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_110" model="account.report.line">
                        <field name="name">110. VAT Credit Relieved (expected value: negative or zero)</field>
                        <field name="code">L10N_NG_C110</field>
                        <field name="hierarchy_level">1</field>
                        <field name="external_formula">monetary</field>
                    </record>
                    <record id="l10n_ng_tr_c_115" model="account.report.line">
                        <field name="name">115. Unrelieved VAT Credit Carried Forward</field>
                        <field name="code">L10N_NG_C115</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_c_115_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_NG_C105.balance - L10N_NG_C110.balance</field>
                                <field name="subformula">if_below(NGN(0))</field>
                            </record>
                            <record id="tax_report_c_115_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_NG_C115.balance</field>
                                <field name="carryover_target">L10N_NG_C100._applied_carryover_balance</field>
                                <field name="subformula" eval="False"/>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ng_tr_c_120" model="account.report.line">
                        <field name="name">120. VAT Payable</field>
                        <field name="code">L10N_NG_C120</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="l10n_ng_tr_c_120_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_NG_C95.balance + L10N_NG_C100.balance</field>
                                <field name="subformula">if_above(NGN(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\withholding_vat_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_ng_wh_vat_report" model="account.report">
        <field name="name">Withholding VAT returns (form 006)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ng"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ng_wh_vat_column" model="account.report.column">
                <field name="name">Amount</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ng_wh_vat_10" model="account.report.line">
                <field name="name">Total VAT withheld</field>
                <field name="code">L10N_NG_WHVAT_10</field>
                <field name="expression_ids">
                    <record id="l10n_ng_wh_vat_10_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">10</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ng_wh_vat_20" model="account.report.line">
                <field name="name">Total withheld VAT payable</field>
                <field name="code">L10N_NG_WHVAT_20</field>
                <field name="expression_ids">
                    <record id="l10n_ng_wh_vat_20_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">10</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-ng.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id"
"l10n_ng_fp_domestic","1","Domestic","1","1","base.ng","","","","",""
"l10n_ng_fp_import","2","Import","1","","","","","","",""
"","","","","","","","l10n_ng_vat_in_7_5","l10n_ng_vat_in_7_5_import","",""

```

## File: data\template\account.tax-ng.csv

```csv
"id","sequence","name","type_tax_use","amount","amount_type","description","tax_group_id","active","tax_exigibility","cash_basis_transition_account_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","price_include_override"
"l10n_ng_vat_out_7_5","10","7.5%","sale","7.5","percent","7.5%","l10n_ng_tax_group_vat_7_5","","","","base","invoice","+20","","",""
"","","","","","","","","","","","tax","invoice","+45","l10n_ng_tax_received","",""
"","","","","","","","","","","","base","refund","-20","","",""
"","","","","","","","","","","","tax","refund","-45","l10n_ng_tax_received","",""
"l10n_ng_vat_out_0","20","0%","sale","0","percent","0%","l10n_ng_tax_group_vat_0","","","","base","invoice","+20||+30","","",""
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-20||-30","","",""
"","","","","","","","","","","","tax","refund","","","",""
"l10n_ng_vat_out_exempt","30","7.5% Exempt","sale","0","percent","7.5% Exempt","l10n_ng_tax_group_vat_7_5","","","","base","invoice","+20||+25","","",""
"","","","","","","","","","","","tax","invoice","","l10n_ng_tax_received","",""
"","","","","","","","","","","","base","refund","-20||-25","","",""
"","","","","","","","","","","","tax","refund","","l10n_ng_tax_received","",""
"l10n_ng_vat_out_5_wh","40","5% WH","sale","-5","percent","5% Withholding","l10n_ng_tax_group_wh_5","","","","base","invoice","+20","","","tax_excluded"
"","","","","","","","","","","","tax","invoice","-35","l10n_ng_tax_received","",""
"","","","","","","","","","","","base","refund","-20","","",""
"","","","","","","","","","","","tax","refund","+35","l10n_ng_tax_received","",""
"l10n_ng_vat_out_10_wh","50","10% WH","sale","-10","percent","10% Withholding","l10n_ng_tax_group_wh_10","","","","base","invoice","+20","","","tax_excluded"
"","","","","","","","","","","","tax","invoice","-35","l10n_ng_tax_received","",""
"","","","","","","","","","","","base","refund","-20","","",""
"","","","","","","","","","","","tax","refund","+35","l10n_ng_tax_received","",""
"l10n_ng_vat_in_7_5","60","7.5%","purchase","7.5","percent","7.5%","l10n_ng_tax_group_vat_7_5","","","","base","invoice","+50","","",""
"","","","","","","","","","","","tax","invoice","+75","l10n_ng_tax_paid","",""
"","","","","","","","","","","","base","refund","-50","","",""
"","","","","","","","","","","","tax","refund","-75","l10n_ng_tax_paid","",""
"l10n_ng_vat_in_0","70","0%","purchase","0","percent","0%","l10n_ng_tax_group_vat_0","","","","base","invoice","+55","","",""
"","","","","","","","","","","","tax","invoice","","l10n_ng_tax_paid","",""
"","","","","","","","","","","","base","refund","-55","","",""
"","","","","","","","","","","","tax","refund","","l10n_ng_tax_paid","",""
"l10n_ng_vat_in_7_5_import","80","7.5% IM","purchase","7.5","percent","7.5% Import","l10n_ng_tax_group_vat_7_5","","","","base","invoice","+65","","",""
"","","","","","","","","","","","tax","invoice","+75","l10n_ng_tax_paid","",""
"","","","","","","","","","","","base","refund","-65","","",""
"","","","","","","","","","","","tax","refund","-75","l10n_ng_tax_paid","",""
"l10n_ng_vat_in_7_5_auto","90","7.5% Auto","purchase","7.5","percent","7.5% Automatic/electronic","l10n_ng_tax_group_vat_7_5","","","","base","invoice","+50","","",""
"","","","","","","","","","","","tax","invoice","+85","l10n_ng_tax_paid","",""
"","","","","","","","","","","","base","refund","-50","","",""
"","","","","","","","","","","","tax","refund","-85","l10n_ng_tax_paid","",""
"l10n_ng_vat_in_7_5_oil","100","7.5% Oil & Gas","purchase","7.5","percent","7.5% Oil & Gas","l10n_ng_tax_group_vat_7_5","","","","base","invoice","+50","","",""
"","","","","","","","","","","","tax","invoice","+90","l10n_ng_tax_paid","",""
"","","","","","","","","","","","base","refund","-50","","",""
"","","","","","","","","","","","tax","refund","-90","l10n_ng_tax_paid","",""
"l10n_ng_vat_in_5_wh","110","5% WH","purchase","-5","percent","5% Withholding","l10n_ng_tax_group_wh_5","","on_payment","l10n_ng_withholding_transitional","base","invoice","","","",""
"","","","","","","","","","","","tax","invoice","-10","l10n_ng_withholding","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","+10","l10n_ng_withholding","",""
"l10n_ng_vat_in_10_wh","120","10% WH","purchase","-10","percent","10% Withholding","l10n_ng_tax_group_wh_10","","on_payment","l10n_ng_withholding_transitional","base","invoice","","","",""
"","","","","","","","","","","","tax","invoice","-10","l10n_ng_withholding","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","+10","l10n_ng_withholding","",""

```

## File: data\template\account.tax.group-ng.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"l10n_ng_tax_group_vat_0","VAT 0%","base.ng","l10n_ng_tax_receivable","l10n_ng_tax_payable"
"l10n_ng_tax_group_vat_7_5","VAT 7.5%","base.ng","l10n_ng_tax_receivable","l10n_ng_tax_payable"
"l10n_ng_tax_group_wh_5","Withholding 5%","base.ng","l10n_ng_withholding","l10n_ng_withholding"
"l10n_ng_tax_group_wh_10","Withholding 10%","base.ng","l10n_ng_withholding","l10n_ng_withholding"

```

## File: models\template_ng.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ng', 'account.account')
    def _get_ng_account_account(self):
        """ Nigerian companies are fine with using the generic COA
        but we need to add Nigeria-specific taxes and a tax report
        """
        return {
            **{f'l10n_ng_{k}': v for k, v in self._parse_csv('generic_coa', 'account.account').items()},
            'l10n_ng_withholding': {
                'name': _("Withholding Tax on Purchases"),
                'code': '252001',
                'account_type': 'liability_current',
                'reconcile': False,
            },
            'l10n_ng_withholding_transitional': {
                'name': _("Withholding Tax on Purchases - Transition Account"),
                'code': '252002',
                'account_type': 'liability_current',
                'reconcile': False,
            },
        }

    @template('ng')
    def _get_ng_template_data(self):
        """ Copies the generic CoA template data.
        Changes to it will be reflected here as well.
        We remove the name and country to use the default values,
        whereas the generic CoA has to override these.
        """
        res = self._get_generic_coa_template_data()
        return {k: f'l10n_ng_{v}' for k, v in res.items() if k not in ('name', 'country')}

    @template('ng', 'res.company')
    def _get_ng_res_company(self):
        res_company_data = self._get_generic_coa_res_company()[self.env.company.id]
        res_company_data['account_fiscal_country_id'] = 'base.ng'

        for field, value in res_company_data.items():
            if 'account_id' in field:
                res_company_data[field] = f'l10n_ng_{value}'
        return {self.env.company.id: res_company_data}

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import template_ng

```

