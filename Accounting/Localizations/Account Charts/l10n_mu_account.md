# Odoo Module: l10n_mu_account

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
    "name": "Mauritius - Accounting",
    "version": "1.0",
    'countries': ['mu'],
    "category": "Accounting/Localizations/Account Charts",
    "description": """
This is the base module to manage the accounting chart for the Republic of Mauritius in Odoo.
==============================================================================================
    - Chart of accounts
    - Taxes
    - Fiscal positions
    - Default settings
    """,
    "author": "Odoo SA",
    "depends": [
        "account",
    ],
    "data": [
        "data/tax_report-mu.xml",
        "views/report_invoice.xml",
    ],
    "demo": [
        "demo/demo_company.xml",
    ],
    "license": "LGPL-3",
}

```

## File: data\tax_report-mu.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mu_tax_report" model="account.report">
        <field name="name">VAT3 Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.mu"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="mu_tr_col_percent" model="account.report.column">
                <field name="name">Percent</field>
                <field name="expression_label">percent</field>
                <field name="figure_type">percentage</field>
            </record>
            <record id="mu_tr_col_value" model="account.report.column">
                <field name="name">Value</field>
                <field name="expression_label">value</field>
            </record>
            <record id="mu_tr_col_vat" model="account.report.column">
                <field name="name">VAT</field>
                <field name="expression_label">vat</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="mu_tr_output" model="account.report.line">
                <field name="name">OUTPUT</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="mu_tr_T1" model="account.report.line">
                        <field name="name">1. Taxable supplies</field>
                        <field name="children_ids">
                            <record id="mu_tr_T1_1" model="account.report.line">
                                <field name="name">1.1. Zero-rated supplies (Exports)</field>
                                <field name="code">T1_1</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T1_1_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_1_value</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T1_2" model="account.report.line">
                                <field name="name">1.2. Zero-rated supplies other than exports</field>
                                <field name="code">T1_2</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T1_2_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_2_value</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T1_3" model="account.report.line">
                                <field name="name">1.3. Taxable supplies made to exempt bodies or persons</field>
                                <field name="code">T1_3</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T1_3_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_3_value</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T1_4" model="account.report.line">
                                <field name="name">1.4. Other taxable supplies</field>
                                <field name="code">T1_4</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T1_4_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_4_value</field>
                                    </record>
                                    <record id="mu_tr_T1_4_vat" model="account.report.expression">
                                        <field name="label">vat</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_4_vat</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T2" model="account.report.line">
                        <field name="name">2. Deferred VAT on importations</field>
                        <field name="code">T2</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T2_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">T2_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T3" model="account.report.line">
                        <field name="name">3. Exempt supplies</field>
                        <field name="code">T3</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T3_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">T3_value</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T4" model="account.report.line">
                        <field name="name">4. Penalty on excess amount overclaimed</field>
                        <field name="code">T4</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T4_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T5" model="account.report.line">
                        <field name="name">5. Total</field>
                        <field name="code">T5</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T5_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T1_1.value + T1_2.value + T1_3.value + T1_4.value + T3.value</field>
                            </record>
                            <record id="mu_tr_T5_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T1_4.vat + T2.vat + T4.vat</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="mu_tr_input" model="account.report.line">
                <field name="name">INPUT - Imports and Purchases</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="mu_tr_T6" model="account.report.line">
                        <field name="name">6. Taxable input on which input tax is allowed as credit</field>
                        <field name="children_ids">
                            <record id="mu_tr_T6_1" model="account.report.line">
                                <field name="name">6.1. Capital goods imported</field>
                                <field name="code">T6_1</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T6_1_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_1_value</field>
                                    </record>
                                    <record id="mu_tr_T6_1_vat" model="account.report.expression">
                                        <field name="label">vat</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_1_vat</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T6_2" model="account.report.line">
                                <field name="name">6.2. Zero-rated imports</field>
                                <field name="code">T6_2</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T6_2_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_2_value</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T6_3" model="account.report.line">
                                <field name="name">6.3. Other imports</field>
                                <field name="code">T6_3</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T6_3_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_3_value</field>
                                    </record>
                                    <record id="mu_tr_T6_3_vat" model="account.report.expression">
                                        <field name="label">vat</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_3_vat</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T6_4" model="account.report.line">
                                <field name="name">6.4. Capital goods purchased locally</field>
                                <field name="code">T6_4</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T6_4_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_4_value</field>
                                    </record>
                                    <record id="mu_tr_T6_4_vat" model="account.report.expression">
                                        <field name="label">vat</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_4_vat</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T6_5" model="account.report.line">
                                <field name="name">6.5. Zero-rated goods and services purchased locally</field>
                                <field name="code">T6_5</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T6_5_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_5_value</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T6_6" model="account.report.line">
                                <field name="name">6.6. Other goods and services purchased locally</field>
                                <field name="code">T6_6</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T6_6_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_6_value</field>
                                    </record>
                                    <record id="mu_tr_T6_6_vat" model="account.report.expression">
                                        <field name="label">vat</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_6_vat</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T7" model="account.report.line">
                        <field name="name">7. Taxable input on which no input tax is allowed as credit</field>
                        <field name="code">T7</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T7_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T8" model="account.report.line">
                        <field name="name">8. Exempt input</field>
                        <field name="children_ids">
                            <record id="mu_tr_T8_1" model="account.report.line">
                                <field name="name">8.1. Imported goods</field>
                                <field name="code">T8_1</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T8_1_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T8_1_value</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T8_2" model="account.report.line">
                                <field name="name">8.2. Goods and services purchased locally</field>
                                <field name="code">T8_2</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T8_2_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T8_2_value</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T9" model="account.report.line">
                        <field name="name">9. Total</field>
                        <field name="code">T9</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T9_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T6_1.value + T6_2.value + T6_3.value + T6_4.value + T6_5.value + T6_6.value + T7.value + T8_1.value + T8_2.value</field>
                            </record>
                            <record id="mu_tr_T9_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T6_1.vat + T6_3.vat + T6_4.vat + T6_6.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T10" model="account.report.line">
                        <field name="name">10. Input tax deductible</field>
                        <field name="code">T10</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T10_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T6_1.vat + T6_6.vat</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="mu_tr_vat_account" model="account.report.line">
                <field name="name">VAT ACCOUNT</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="mu_tr_T11" model="account.report.line">
                        <field name="name">11. VAT due and payable / (Excess VAT) (5B minus 10B)</field>
                        <field name="code">T11</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T11_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T5.vat - T10.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T12" model="account.report.line">
                        <field name="name">12. Excess amount of VAT brought forward</field>
                        <field name="code">T12</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T12_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="mu_tr_T12_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">T12_tag</field>
                            </record>
                            <record id="mu_tr_T12_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T12.tag + T12._applied_carryover_balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T13" model="account.report.line">
                        <field name="name">13. VAT adjustment: Increase /(Decrease)</field>
                        <field name="code">T13</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T13_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T14" model="account.report.line">
                        <field name="name">14. VAT due and payable / (Excess VAT)</field>
                        <field name="code">T14</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T14_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T11.vat - T12.vat + T13.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T15" model="account.report.line">
                        <field name="name">15. Claim for repayment of VAT - Proportion claimable</field>
                        <field name="code">T15</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T15_rounded" model="account.report.expression">
                                <field name="label">_rounded</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">(T1_1.value + T1_2.value) / (T5.value - T3.value) * 100</field>
                                <field name="subformula">round(0)</field>
                            </record>
                            <record id="mu_tr_T15_percent" model="account.report.expression">
                                <field name="label">percent</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T15._rounded</field>
                                <field name="subformula">if_other_expr_below(T14.vat, MUR(0))</field>
                                <field name="figure_type">percentage</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="mu_tr_T15_1" model="account.report.line">
                                <field name="name">15.1. On capital goods</field>
                                <field name="code">T15_1</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T15_1_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T15_2" model="account.report.line">
                                <field name="name">15.2. In respect of other goods and services</field>
                                <field name="code">T15_2</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T15_2_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="mu_tr_T15_3" model="account.report.line">
                                <field name="name">15.3. Total repayment claimed</field>
                                <field name="code">T15_3</field>
                                <field name="expression_ids">
                                    <record id="mu_tr_T15_3_vat" model="account.report.expression">
                                        <field name="label">vat</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(T15.percent / 100) * (T15_1.value + T15_2.value)</field>
                                        <field name="subformula">if_other_expr_below(T14.vat, MUR(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T16" model="account.report.line">
                        <field name="name">16. Excess VAT carried forward</field>
                        <field name="code">T16</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T16_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T14.vat + T15_3.vat</field>
                            </record>
                            <record id="mu_tr_T16_vat_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T16.vat</field>
                                <field name="carryover_target">T12._applied_carryover_balance</field>
                                <field name="subformula" eval="False"/>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T17" model="account.report.line">
                        <field name="name">17. Penalty for submission after due date</field>
                        <field name="code">T17</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T17_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T18" model="account.report.line">
                        <field name="name">18. Penalty and interest for payment of VAT after due date</field>
                        <field name="code">T18</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T18_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="mu_tr_T19" model="account.report.line">
                        <field name="name">19. Total VAT / Penalties / Interests due and payable</field>
                        <field name="code">T19</field>
                        <field name="expression_ids">
                            <record id="mu_tr_T19_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">T16.vat + T17.vat + T18.vat</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-mu.csv

```csv
"id","name","code","account_type","tag_ids","reconcile"
"mu_current_assets","Current Assets","1010","asset_current","","False"
"mu_stock_in","Stock Interim (Received)","1102","asset_current","","True"
"mu_stock_valuation","Stock Valuation","1101","asset_current","","False"
"mu_stock_out","Stock Interim (Delivered)","1103","asset_current","","True"
"mu_cost_of_production","Cost of Production","1104","asset_current","","True"
"mu_receivable","Account Receivable","1210","asset_receivable","","True"
"mu_pos_receivable","Account Receivable (PoS)","122O","asset_receivable","","True"
"mu_to_receive_rec","Products to receive","1230","asset_current","","True"
"mu_tax_paid","Tax Paid","1310","asset_current","","False"
"mu_tax_receivable","Tax Receivable","1320","asset_current","","False"
"mu_prepayments","Prepayments","1410","asset_prepayments","","False"
"mu_fixed_assets","Fixed Asset","1510","asset_fixed","","False"
"mu_non_current_assets","Non-current assets","1910","asset_non_current","","False"
"mu_current_liabilities","Current Liabilities","2010","liability_current","","False"
"mu_payable","Account Payable","2110","liability_payable","","True"
"mu_to_receive_pay","Bills to receive","2111","liability_current","","True"
"mu_salary_payable","Salary Payable","2300","liability_current","","True"
"mu_employee_payroll_taxes","Employee Payroll Taxes","2301","liability_current","","True"
"mu_employer_payroll_taxes","Employer Payroll Taxes","2302","liability_current","","True"
"mu_tax_received","Tax Received","2510","liability_current","","False"
"mu_tax_payable","Tax Payable","2520","liability_current","","False"
"mu_non_current_liabilities","Non-current Liabilities","2910","liability_non_current","","False"
"mu_capital","Capital","3010","equity","","False"
"mu_dividends","Dividends","3020","equity","","False"
"mu_income","Product Sales","4000","income","account.account_tag_operating","False"
"mu_income_currency_exchange","Foreign Exchange Gain","4410","income","account.account_tag_financing","False"
"mu_cash_diff_income","Cash Difference Gain","4420","income","account.account_tag_investing","False"
"mu_cash_discount_loss","Cash Discount Loss","4430","expense","","False"
"mu_other_income","Other Income","4500","income_other","","False"
"mu_cost_of_goods_sold","Cost of Goods Sold","5000","expense_direct_cost","account.account_tag_operating","False"
"mu_expense","Expenses","6000","expense","account.account_tag_operating","False"
"mu_expense_invest","Purchase of Equipments","6110","expense","account.account_tag_investing","False"
"mu_expense_rent","Rent","6120","expense","account.account_tag_investing","False"
"mu_expense_finance","Bank Fees","6200","expense","account.account_tag_financing","False"
"mu_expense_salary","Salary Expenses","6300","expense","account.account_tag_operating","False"
"mu_expense_currency_exchange","Foreign Exchange Loss","6410","expense","account.account_tag_financing","False"
"mu_cash_diff_expense","Cash Difference Loss","6420","expense","account.account_tag_investing","False"
"mu_cash_discount_gain","Cash Discount Gain","6430","income","","False"
"mu_expense_rd","RD Expenses","9610","expense","account.account_tag_investing","False"
"mu_expense_sales","Sales Expenses","9620","expense","account.account_tag_investing","False"

```

## File: data\template\account.fiscal.position-mu.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id"
"mu_fp_domestic","1","Domestic","1","1","base.mu","","","","",""
"mu_fp_ex","4","Import/Export (EX)","1","","","","mu_tax_sale_15","mu_tax_sale_0_export","",""
"","","","","","","","mu_tax_purchase_15","mu_tax_purchase_15_import","",""
"","","","","","","","mu_tax_purchase_15_capital","mu_tax_purchase_15_import_capital","",""
"","","","","","","","mu_tax_purchase_exempt","mu_tax_purchase_import_exempt","",""

```

## File: data\template\account.tax-mu.csv

```csv
"id","name","description","invoice_label","type_tax_use","amount_type","amount","tax_group_id","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/factor_percent","repartition_line_ids/account_id","repartition_line_ids/tag_ids"
"mu_tax_sale_15","15%","15% Standard rate","15%","sale","percent","15","mu_tax_group_vat_15","True","base","invoice","","","+T1_4_value"
"","","","","","","","","","tax","invoice","","mu_tax_received","+T1_4_vat"
"","","","","","","","","","base","refund","","","-T1_4_value"
"","","","","","","","","","tax","refund","","mu_tax_received","-T1_4_vat"
"mu_tax_sale_0","0%","0% Zero rated","0%","sale","percent","0","mu_tax_group_vat_0","True","base","invoice","","","+T1_2_value"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","","","-T1_2_value"
"","","","","","","","","","tax","refund","","",""
"mu_tax_sale_0_export","0% EX","0% Export","0%","sale","percent","0","mu_tax_group_vat_0","True","base","invoice","","","+T1_1_value"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","","","-T1_1_value"
"","","","","","","","","","tax","refund","","",""
"mu_tax_sale_exempt","0% Exempt","0% Exempt","0%","sale","percent","0","mu_tax_group_vat_0","True","base","invoice","","","+T3_value"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","","","-T3_value"
"","","","","","","","","","tax","refund","","",""
"mu_tax_purchase_15","15%","15%","15%","purchase","percent","15","mu_tax_group_vat_15","True","base","invoice","","","+T6_6_value"
"","","","","","","","","","tax","invoice","","mu_tax_paid","+T6_6_vat"
"","","","","","","","","","base","refund","","","-T6_6_value"
"","","","","","","","","","tax","refund","","mu_tax_paid","-T6_6_vat"
"mu_tax_purchase_15_import","15% EX","15% Import","15%","purchase","percent","15","mu_tax_group_vat_15","True","base","invoice","","","+T6_3_value"
"","","","","","","","","","tax","invoice","","mu_tax_paid","+T6_3_vat"
"","","","","","","","","","base","refund","","","-T6_3_value"
"","","","","","","","","","tax","refund","","mu_tax_paid","-T6_3_vat"
"mu_tax_purchase_15_capital","15% Capital","15% Capital","15%","purchase","percent","15","mu_tax_group_vat_15","True","base","invoice","","","+T6_4_value"
"","","","","","","","","","tax","invoice","","mu_tax_paid","+T6_4_vat"
"","","","","","","","","","base","refund","","","-T6_4_value"
"","","","","","","","","","tax","refund","","mu_tax_paid","-T6_4_vat"
"mu_tax_purchase_15_import_capital","15% EX Capital","15% Import Capital","15%","purchase","percent","15","mu_tax_group_vat_15","True","base","invoice","","","+T6_1_value"
"","","","","","","","","","tax","invoice","","mu_tax_paid","+T6_1_vat"
"","","","","","","","","","base","refund","","","-T6_1_value"
"","","","","","","","","","tax","refund","","mu_tax_paid","-T6_1_vat"
"mu_tax_purchase_0","0%","0% Zero rated","0%","purchase","percent","0","mu_tax_group_vat_0","True","base","invoice","","","+T6_5_value"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","","","-T6_5_value"
"","","","","","","","","","tax","refund","","",""
"mu_tax_purchase_0_import","0% EX","0% Import","0%","purchase","percent","0","mu_tax_group_vat_0","True","base","invoice","","","+T6_2_value"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","","","-T6_2_value"
"","","","","","","","","","tax","refund","","",""
"mu_tax_purchase_exempt","0% Exempt","0% Exempt","0%","purchase","percent","0","mu_tax_group_vat_0","True","base","invoice","","","+T8_2_value"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","","","-T8_2_value"
"","","","","","","","","","tax","refund","","",""
"mu_tax_purchase_import_exempt","0% EX Exempt","0% Import Exempt","0%","purchase","percent","0","mu_tax_group_vat_0","True","base","invoice","","","+T8_1_value"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","","","-T8_1_value"
"","","","","","","","","","tax","refund","","",""

```

## File: data\template\account.tax.group-mu.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"mu_tax_group_vat_15","VAT 15%","base.mu","mu_tax_receivable","mu_tax_payable"
"mu_tax_group_vat_0","VAT 0%","base.mu","mu_tax_receivable","mu_tax_payable"

```

## File: models\account_move.py

```python
from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_name_invoice_report(self):
        # EXTENDS account
        self.ensure_one()
        if self.company_id.account_fiscal_country_id.code == 'MU':
            return 'l10n_mu_account.report_invoice_document'
        return super()._get_name_invoice_report()

```

## File: models\base_document_layout.py

```python
from markupsafe import Markup

from odoo import api, fields, models


class BaseDocumentLayout(models.TransientModel):
    _inherit = 'base.document.layout'

    @api.model
    def _default_company_details(self):
        company_details = super()._default_company_details()
        company = self.env.company
        if company.company_registry and company.country_code == 'MU':
            return company_details + Markup('<br/> %s') % company.company_registry
        return company_details

    company_details = fields.Html(default=_default_company_details)

```

## File: models\template_mu.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('mu')
    def _get_mu_template_data(self):
        return {
            'property_account_receivable_id': 'mu_receivable',
            'property_account_payable_id': 'mu_payable',
            'property_account_expense_categ_id': 'mu_expense',
            'property_account_income_categ_id': 'mu_income',
            'property_stock_valuation_account_id': 'mu_stock_valuation',
            'property_advance_tax_payment_account_id': 'mu_tax_paid',
            'property_tax_payable_account_id': 'mu_tax_payable',
            'property_tax_receivable_account_id': 'mu_tax_receivable',
            'use_anglo_saxon': False,
            'code_digits': '6',
        }

    @template('mu', 'res.company')
    def _get_mu_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.mu',
                'bank_account_code_prefix': '230',
                'cash_account_code_prefix': '231',
                'transfer_account_code_prefix': '232',
                'account_default_pos_receivable_account_id': 'mu_pos_receivable',
                'income_currency_exchange_account_id': 'mu_income_currency_exchange',
                'expense_currency_exchange_account_id': 'mu_expense_currency_exchange',
                'account_journal_early_pay_discount_gain_account_id': 'mu_cash_discount_gain',
                'account_journal_early_pay_discount_loss_account_id': 'mu_cash_discount_loss',
                'default_cash_difference_income_account_id': 'mu_cash_diff_income',
                'default_cash_difference_expense_account_id': 'mu_cash_diff_expense',
                'account_sale_tax_id': 'mu_tax_sale_15',
                'account_purchase_tax_id': 'mu_tax_purchase_15',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move
from . import base_document_layout
from . import template_mu

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//div[@id='informations']" position="before">
            <t t-set="forced_vat" t-value="o.company_id.vat"/>
        </xpath>

        <xpath expr="//div[hasclass('page')]/h2" position="replace">
            <h2>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'">VAT Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'draft'">Draft VAT Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'cancel'">Cancelled VAT Invoice</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'posted'">Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'draft'">Draft Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'cancel'">Cancelled Credit Note</span>
                <span t-elif="o.move_type == 'in_refund'">Vendor Credit Note</span>
                <span t-elif="o.move_type == 'in_invoice'">Vendor Bill</span>
                <span t-if="o.name != '/'" t-field="o.name"/>
            </h2>
        </xpath>

        <xpath expr="//div[@name='address_not_same_as_shipping']//t[@t-set='address']" position="inside">
            <span t-field="o.partner_id.company_registry"/>
        </xpath>
        <xpath expr="//div[@name='address_same_as_shipping']//t[@t-set='address']" position="inside">
            <span t-field="o.partner_id.company_registry"/>
        </xpath>
        <xpath expr="//div[@name='no_shipping']//t[@t-set='address']" position="inside">
            <span t-field="o.partner_id.company_registry"/>
        </xpath>

        <xpath expr="//div[@id='total']" position="after">
            <div t-if="o.currency_id.id != o.company_id.currency_id.id" class="ms-1">
                <t t-set="rate" t-value="round(o.env['res.currency']._get_conversion_rate(o.company_id.currency_id, o.currency_id, o.company_id, o.invoice_date), 8)"/>
                <strong>Exchange Rate</strong>: 1 <span t-esc="o.company_id.currency_id.name"/> = <span t-esc="rate"/> <span t-esc="o.currency_id.name"/>
            </div>
        </xpath>
    </template>

    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_mu_account.report_invoice_document'"
               t-call="l10n_mu_account.report_invoice_document"
               t-lang="lang"/>
        </xpath>
    </template>
</odoo>

```

