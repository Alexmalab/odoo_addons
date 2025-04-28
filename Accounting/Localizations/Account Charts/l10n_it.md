# Odoo Module: l10n_it

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Italy - Accounting',
    'countries': ['it'],
    'version': '0.7',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'auto_install': ['account'],
    'author': 'OpenERP Italian Community',
    'description': """
Piano dei conti italiano di un'impresa generica.
================================================

Italian accounting chart and localization.
    """,
    'category': 'Accounting/Localizations/Account Charts',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/italy.html',
    'data': [
        'data/account_account_tag.xml',
        'data/tax_report/account_monthly_tax_report_data.xml',
        'data/tax_report/annual_report_sections/va.xml',
        'data/tax_report/annual_report_sections/ve.xml',
        'data/tax_report/annual_report_sections/vf.xml',
        'data/tax_report/annual_report_sections/vh.xml',
        'data/tax_report/annual_report_sections/vj.xml',
        'data/tax_report/annual_report_sections/vl.xml',
        'data/tax_report/account_annual_tax_report_data.xml',
        'data/account_tax_report_data.xml',
        'views/account_tax_views.xml'
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <!-- Account Tags Balance Sheet -->
        <record id="account_tag_A_ATT" model="account.account.tag">
            <field name="name">Receivables from shareholders</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_B_ATT" model="account.account.tag">
            <field name="name">Fixed assets</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_C_ATT" model="account.account.tag">
            <field name="name">Current assets</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_D_ATT" model="account.account.tag">
            <field name="name">Accruals and deferrals - Assets</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_A_PASS" model="account.account.tag">
            <field name="name">Shareholders' Equity</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_B_PASS" model="account.account.tag">
            <field name="name">Provisions for risks and charges</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_C_PASS" model="account.account.tag">
            <field name="name">Severance pay</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_D_PASS" model="account.account.tag">
            <field name="name">Debts</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_E_PASS" model="account.account.tag">
            <field name="name">Accruals and deferrals - Liabilities</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_RISCHI" model="account.account.tag">
            <field name="name">Risks</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_IMPEGNI" model="account.account.tag">
            <field name="name">Commitments</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_BENI" model="account.account.tag">
            <field name="name">Third party assets</field>
            <field name="applicability">accounts</field>
        </record>
        <!-- Account Tags Profit & Loss -->
        <record id="account_tag_A_PL" model="account.account.tag">
            <field name="name">Value of production</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_B_PL" model="account.account.tag">
            <field name="name">Production costs</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_C_PL" model="account.account.tag">
            <field name="name">Financial income and expenses</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_D_PL" model="account.account.tag">
            <field name="name">Value adjustments of financial assets and liabilities</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_E_PL" model="account.account.tag">
            <field name="name">Extraordinary income and expenses</field>
            <field name="applicability">accounts</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report_vat" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.it"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="active" eval="False"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_vat_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_operazione_imponibile" model="account.report.line">
                <field name="name">Taxable transaction</field>
                <field name="code">h1</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp2" model="account.report.line">
                        <field name="name">VP2 - Total active transactions</field>
                        <field name="code">VP2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">02</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp3" model="account.report.line">
                        <field name="name">VP3 - Total passive transactions</field>
                        <field name="code">VP3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">03</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_iva" model="account.report.line">
                <field name="name">VAT</field>
                <field name="code">h2</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp4" model="account.report.line">
                        <field name="name">VP4 - VAT due</field>
                        <field name="code">VP4</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4v</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp5" model="account.report.line">
                        <field name="name">VP5 - VAT Deductible</field>
                        <field name="code">VP5</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5v</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_saldi_riporti_e_interessi" model="account.report.line">
                <field name="name">Balances, carryovers and interest</field>
                <field name="code">h3</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp6" model="account.report.line">
                        <field name="name">VP6 - VAT due</field>
                        <field name="code">VP6</field>
                        <field name="children_ids">
                            <record id="tax_report_line_vp6a" model="account.report.line">
                                <field name="name">VP6a - VAT due (payable)</field>
                                <field name="code">VP6a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp6a_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_vp6b" model="account.report.line">
                                <field name="name">VP6b - VAT due (credit)</field>
                                <field name="code">VP6b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp6b_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp7" model="account.report.line">
                        <field name="name">VP7 - Previous period debt not to exceed 25,82</field>
                        <field name="code">VP7</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp7_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp7</field>
                            </record>
                            <record id="tax_report_line_vp7_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_vp7_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP7._applied_carryover_balance + VP7.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp8" model="account.report.line">
                        <field name="name">VP8 - Previous period credit</field>
                        <field name="code">VP8</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp8_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp8</field>
                            </record>
                            <record id="tax_report_line_vp8_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_vp8_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP8._applied_carryover_balance + VP8.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp9" model="account.report.line">
                        <field name="name">VP9 - Previous year credit</field>
                        <field name="code">VP9</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp9_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp9</field>
                            </record>
                            <record id="tax_report_line_vp9_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_vp9_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP9._applied_carryover_balance + VP9.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp10" model="account.report.line">
                        <field name="name">VP10 - EU car payments</field>
                        <field name="code">VP10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp10</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp11" model="account.report.line">
                        <field name="name">VP11 - Tax Credit</field>
                        <field name="code">VP11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp12" model="account.report.line">
                        <field name="name">VP12 - Interest due for quarterly settlements</field>
                        <field name="code">VP12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp13" model="account.report.line">
                        <field name="name">VP13 - Down payment due</field>
                        <field name="code">VP13</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp13</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_conto_corrente_iva" model="account.report.line">
                <field name="name">VAT account</field>
                <field name="code">h4</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp14" model="account.report.line">
                        <field name="name">VP14 - VAT payable</field>
                        <field name="code">VP14</field>
                        <field name="children_ids">
                            <record id="tax_report_line_vp14a" model="account.report.line">
                                <field name="name">VP14a - VAT payable (debit)</field>
                                <field name="code">VP14a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp14a_vp4_vp5_dif_pos" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_pos</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14a_vp4_vp5_dif_neg" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_neg</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14a_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(VP14a.vp4_vp5_dif_pos + VP7.balance + VP12.balance) - (-VP14a.vp4_vp5_dif_neg + VP8.balance + VP9.balance + VP10.balance + VP11.balance + VP13.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14a_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP14a.balance</field>
                                        <field name="subformula">if_between(EUR(0), EUR(25.82))</field>
                                        <field name="carryover_target">VP7._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_vp14b" model="account.report.line">
                                <field name="name">VP14b - VAT payable (credit)</field>
                                <field name="code">VP14b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp14b_vp4_vp5_dif_pos" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_pos</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14b_vp4_vp5_dif_neg" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_neg</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14b_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(-VP14b.vp4_vp5_dif_neg + VP8.balance + VP9.balance + VP10.balance + VP11.balance + VP13.balance) - (VP14b.vp4_vp5_dif_pos + VP7.balance + VP12.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14b_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP14b.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                        <field name="carryover_target">VP8._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_turnover" model="account.report.line">
                <field name="name">Turnover</field>
                <field name="code">VE</field>
                <field name="children_ids">
                    <record id="tax_report_line_turnover_section_1" model="account.report.line">
                        <field name="name">Contributions of agricultural products and transfers from exempt farmers (in case of exceeding 1/3</field>
                        <field name="code">VE_section1</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve1" model="account.report.line">
                                <field name="name">VE1 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 2%</field>
                                <field name="code">VE1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve2" model="account.report.line">
                                <field name="name">VE2 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 4%</field>
                                <field name="code">VE2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve3" model="account.report.line">
                                <field name="name">VE3 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 6,4%</field>
                                <field name="code">VE3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve4" model="account.report.line">
                                <field name="name">VE4 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 7,3%</field>
                                <field name="code">VE4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve5" model="account.report.line">
                                <field name="name">VE5 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 7,5%</field>
                                <field name="code">VE5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve6" model="account.report.line">
                                <field name="name">VE6 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 8,3%</field>
                                <field name="code">VE6</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve7" model="account.report.line">
                                <field name="name">VE7 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 8,5%</field>
                                <field name="code">VE7</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve7</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve8" model="account.report.line">
                                <field name="name">VE8 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 8,8%</field>
                                <field name="code">VE8</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve8</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve9" model="account.report.line">
                                <field name="name">VE9 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 9,5%</field>
                                <field name="code">VE9</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve9</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve10" model="account.report.line">
                                <field name="name">VE10 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 10%</field>
                                <field name="code">VE10</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve10</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve11" model="account.report.line">
                                <field name="name">VE11 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 12,3%</field>
                                <field name="code">VE11</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve11_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve11</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_turnover_section_2" model="account.report.line">
                        <field name="name">Agricultural taxable transactions (Article 34 paragraph 1) and commercial and professional taxable transactions</field>
                        <field name="code">VE_section2</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve20" model="account.report.line">
                                <field name="name">VE20 - Taxable transactions rate 4%</field>
                                <field name="code">VE20</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve21" model="account.report.line">
                                <field name="name">VE21 - Taxable transactions rate 5%</field>
                                <field name="code">VE21</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve21</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve22" model="account.report.line">
                                <field name="name">VE22 - Taxable transactions rate 10%</field>
                                <field name="code">VE22</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve22</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve23" model="account.report.line">
                                <field name="name">VE23 - Taxable transactions rate 22%</field>
                                <field name="code">VE23</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve23</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_turnover_section_3" model="account.report.line">
                        <field name="name">Total taxable income and tax</field>
                        <field name="code">VE_section3</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve24" model="account.report.line">
                                <field name="name">VE24 - Total lines VE1 to VE11 and lines VE20 to VE23</field>
                                <field name="code">VE24</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve24_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">
                                            VE1.balance + VE2.balance + VE3.balance + VE4.balance + VE5.balance
                                            + VE6.balance + VE7.balance + VE8.balance + VE9.balance + VE10.balance
                                            + VE11.balance + VE20.balance + VE21.balance + VE22.balance + VE23.balance
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve25" model="account.report.line">
                                <field name="name">VE25 - Variations and rounding (use &#43;/&#8722; sign)</field>
                                <field name="code">VE25</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve25_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve25</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve26" model="account.report.line">
                                <field name="name">VE26 - Total VE24 and VE25</field>
                                <field name="code">VE26</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve26_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">
                                            VE24.balance + VE25.balance
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_turnover_section_4" model="account.report.line">
                        <field name="name">Other operations</field>
                        <field name="code">VE_section4</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve30" model="account.report.line">
                                <field name="name">VE30 - Transactions that contribute to the formation of the ceiling</field>
                                <field name="code">VE30</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_ve30_I" model="account.report.line">
                                        <field name="name">VE30_I - Total</field>
                                        <field name="code">VE30_I</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_i_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">
                                                    VE30_II.balance + VE30_III.balance + VE30_IV.balance + VE30_V.balance
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_ii" model="account.report.line">
                                        <field name="name">VE30_II - Exports</field>
                                        <field name="code">VE30_II</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_ii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_ii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_iii" model="account.report.line">
                                        <field name="name">VE30_III - Intra-Community supplies</field>
                                        <field name="code">VE30_III</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_iii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_iii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_iv" model="account.report.line">
                                        <field name="name">VE30_IV - Transfers to San Marino</field>
                                        <field name="code">VE30_IV</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_iv_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_iv</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_v" model="account.report.line">
                                        <field name="name">VE30_V - Assimilated operations</field>
                                        <field name="code">VE30_V</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_v_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_v</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve31" model="account.report.line">
                                <field name="name">VE31 - Non-taxable transactions as a result of declarations of intent</field>
                                <field name="code">VE31</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve31_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve31</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve32" model="account.report.line">
                                <field name="name">VE32 - Other non-taxable transactions</field>
                                <field name="code">VE32</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve32_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve32</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve33" model="account.report.line">
                                <field name="name">VE33 - Exempt transactions (art.10</field>
                                <field name="code">VE33</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve33_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve33</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve34" model="account.report.line">
                                <field name="name">VE34 - Transactions not subject to the tax under Articles 7 to 7-septies</field>
                                <field name="code">VE34</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve34_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve34</field>
                                    </record>
                                </field>
                            </record>

                            <record id="tax_report_line_ve35" model="account.report.line">
                                <field name="name">VE35 - Transactions with application of internal reverse charge</field>
                                <field name="code">VE35</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_ve35_I" model="account.report.line">
                                        <field name="name">VE35_I - Total</field>
                                        <field name="code">VE35_I</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_i_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">
                                                    VE35_II.balance + VE35_III.balance + VE35_IV.balance
                                                    + VE35_V.balance + VE35_VI.balance + VE35_VII.balance
                                                    + VE35_VIII.balance + VE35_IX.balance
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_ii" model="account.report.line">
                                        <field name="name">VE35_II - Disposal of scrap and other recovered materials</field>
                                        <field name="code">VE35_II</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_ii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_ii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_iii" model="account.report.line">
                                        <field name="name">VE35_III - Disposals of pure gold and silver</field>
                                        <field name="code">VE35_III</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_iii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_iii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_iv" model="account.report.line">
                                        <field name="name">VE35_IV - Subcontracting in the construction industry</field>
                                        <field name="code">VE35_IV</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_iv_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_iv</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_v" model="account.report.line">
                                        <field name="name">VE35_V - Disposal of capital buildings</field>
                                        <field name="code">VE35_V</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_v_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_v</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_vi" model="account.report.line">
                                        <field name="name">VE35_VI - Disposal of cell phones</field>
                                        <field name="code">VE35_VI</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_vi_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_vi</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_vii" model="account.report.line">
                                        <field name="name">VE35_VII - Disposal of electronic products</field>
                                        <field name="code">VE35_VII</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_vii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_vii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_viii" model="account.report.line">
                                        <field name="name">VE35_VIII - Benefits construction and related industries</field>
                                        <field name="code">VE35_VIII</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_viii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_viii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_ix" model="account.report.line">
                                        <field name="name">VE35_IX - Energy sector operations</field>
                                        <field name="code">VE35_IX</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_ix_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_ix</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve36" model="account.report.line">
                                <field name="name">VE36 - Non-taxable transactions made to earthquake victims</field>
                                <field name="code">VE36</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve36_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve36</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve37" model="account.report.line">
                                <field name="name">VE37 - Transactions made during the year but with tax due in subsequent years</field>
                                <field name="code">VE37</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_ve37_I" model="account.report.line">
                                        <field name="name">VE37_I - Total</field>
                                        <field name="code">VE37_I</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve37_i_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve37_i</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve37_ii" model="account.report.line">
                                        <field name="name">VE37_II - ex art. 32-bis, DL no. 83/2012</field>
                                        <field name="code">VE37_II</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve37_ii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve37_ii</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve38" model="account.report.line">
                                <field name="name">VE38 - Transactions with parties referred to in Article 17-ter.</field>
                                <field name="code">VE38</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve38_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve38</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve39" model="account.report.line">
                                <field name="name">VE39 - (minus) Transactions made in previous years but with tax due in 2022</field>
                                <field name="code">VE39</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve39_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve39</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve40" model="account.report.line">
                                <field name="name">VE40 - (minus) Disposals of depreciable assets and internal transfers.</field>
                                <field name="code">VE40</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve40_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve40</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_reverse_charge_iva" model="account.report.line">
                <field name="name">Reverse Charge</field>
                <field name="code">VJ</field>
                <field name="children_ids">
                    <record id="tax_report_line_vj1" model="account.report.line">
                        <field name="name">VJ1 - Purchases of goods from Vatican City and San Marino</field>
                        <field name="code">VJ1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj2" model="account.report.line">
                        <field name="name">VJ2 - Extraction of goods from VAT warehouses</field>
                        <field name="code">VJ2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj3" model="account.report.line">
                        <field name="name">VJ3 - Purchases of goods already in Italy or services, from non-residents</field>
                        <field name="code">VJ3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj3</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj4" model="account.report.line">
                        <field name="name">VJ4 - Fees paid to resellers of travel tickets and resellers of parking documents</field>
                        <field name="code">VJ4</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj4</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj5" model="account.report.line">
                        <field name="name">VJ5 - Commissions paid by travel agents to their intermediaries</field>
                        <field name="code">VJ5</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj5</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj6" model="account.report.line">
                        <field name="name">VJ6 - Purchases of scrap and other recovered materials</field>
                        <field name="code">VJ6</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj6</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj7" model="account.report.line">
                        <field name="name">VJ7 - Purchases of industrial gold and pure silver made in Italy</field>
                        <field name="code">VJ7</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj7</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj8" model="account.report.line">
                        <field name="name">VJ8 - Investment gold purchases made in Italy</field>
                        <field name="code">VJ8</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj8</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj9" model="account.report.line">
                        <field name="name">VJ9 - Intra-EU Purchases of Goods</field>
                        <field name="code">VJ9</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj9</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj10" model="account.report.line">
                        <field name="name">VJ10 - Imports of scrap and other recovered materials</field>
                        <field name="code">VJ10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj10</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj11" model="account.report.line">
                        <field name="name">VJ11 - Imports of industrial gold and pure silver</field>
                        <field name="code">VJ11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj12" model="account.report.line">
                        <field name="name">VJ12 - Subcontracting of services in the construction field</field>
                        <field name="code">VJ12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj13" model="account.report.line">
                        <field name="name">VJ13 - Purchases of buildings or portions of buildings used for capital purposes</field>
                        <field name="code">VJ13</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj13</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj14" model="account.report.line">
                        <field name="name">VJ14 - Purchases of cell phones</field>
                        <field name="code">VJ14</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj14</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj15" model="account.report.line">
                        <field name="name">VJ15 - Purchases of electronic products</field>
                        <field name="code">VJ15</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj15</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj16" model="account.report.line">
                        <field name="name">VJ16 - Provision of services in the construction field</field>
                        <field name="code">VJ16</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj16</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj17" model="account.report.line">
                        <field name="name">VJ17 - Purchases of energy sector goods and services</field>
                        <field name="code">VJ17</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj17</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj18" model="account.report.line">
                        <field name="name">VJ18 - Purchases made by VAT-registered public administrations</field>
                        <field name="code">VJ18</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj18</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj19" model="account.report.line">
                        <field name="name">VJ19 - Total frame VJ</field>
                        <field name="code">VJ19</field>
                        <field name="aggregation_formula">VJ1.balance + VJ2.balance + VJ3.balance + VJ4.balance + VJ5.balance + VJ6.balance + VJ7.balance + VJ8.balance + VJ9.balance + VJ10.balance + VJ11.balance + VJ12.balance + VJ13.balance + VJ14.balance + VJ15.balance + VJ16.balance + VJ17.balance + VJ18.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\account_annual_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_annual_report_vat" model="account.report">
        <field name="name">Annual Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.it"/>
        <field name="availability_condition">country</field>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="filter_hierarchy">never</field>
        <field name="integer_rounding">HALF-UP</field>
        <field name="use_sections" eval="True"/>
        <field name="section_report_ids" eval="[Command.set([ref('l10n_it.tax_annual_report_vat_va'),
                                                            ref('l10n_it.tax_annual_report_vat_ve'),
                                                            ref('l10n_it.tax_annual_report_vat_vf'),
                                                            ref('l10n_it.tax_annual_report_vat_vh'),
                                                            ref('l10n_it.tax_annual_report_vat_vj'),
                                                            ref('l10n_it.tax_annual_report_vat_vl')])]"/>
    </record>
</odoo>

```

## File: data\tax_report\account_monthly_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_monthly_report_vat" model="account.report">
        <field name="name">Monthly VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.it"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="integer_rounding">HALF-UP</field>
        <field name="column_ids">
            <record id="tax_monthly_report_vat_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_monthly_report_line_operazione_imponibile" model="account.report.line">
                <field name="name">Taxable transaction</field>
                <field name="code">h1</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_monthly_report_line_vp2" model="account.report.line">
                        <field name="name">VP2 - Total active transactions</field>
                        <field name="code">VP2</field>
                        <field name="tax_tags_formula">02</field>
                    </record>
                    <record id="tax_monthly_report_line_vp3" model="account.report.line">
                        <field name="name">VP3 - Total passive transactions</field>
                        <field name="code">VP3</field>
                        <field name="tax_tags_formula">03</field>
                    </record>
                </field>
            </record>
            <record id="tax_monthly_report_line_iva" model="account.report.line">
                <field name="name">VAT</field>
                <field name="code">h2</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_monthly_report_line_vp4" model="account.report.line">
                        <field name="name">VP4 - VAT due</field>
                        <field name="code">VP4</field>
                        <field name="tax_tags_formula">4v</field>
                    </record>
                    <record id="tax_monthly_report_line_vp5" model="account.report.line">
                        <field name="name">VP5 - VAT Deductible</field>
                        <field name="code">VP5</field>
                        <field name="tax_tags_formula">5v</field>
                    </record>
                </field>
            </record>
            <record id="tax_monthly_report_line_saldi_riporti_e_interessi" model="account.report.line">
                <field name="name">Balances, carryovers and interest</field>
                <field name="code">h3</field>
                <field name="children_ids">
                    <record id="tax_monthly_report_line_vp6" model="account.report.line">
                        <field name="name">VP6 - VAT due</field>
                        <field name="code">VP6</field>
                        <field name="children_ids">
                            <record id="tax_monthly_report_line_vp6a" model="account.report.line">
                                <field name="name">VP6a - VAT due (payable)</field>
                                <field name="code">VP6a</field>
                                <field name="expression_ids">
                                    <record id="tax_monthly_report_line_vp6a_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_monthly_report_line_vp6b" model="account.report.line">
                                <field name="name">VP6b - VAT due (credit)</field>
                                <field name="code">VP6b</field>
                                <field name="expression_ids">
                                    <record id="tax_monthly_report_line_vp6b_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_monthly_report_line_vp7" model="account.report.line">
                        <field name="name">VP7 - Previous period debt not to exceed 25,82</field>
                        <field name="code">VP7</field>
                        <field name="expression_ids">
                            <record id="tax_monthly_report_line_vp7_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp7</field>
                            </record>
                            <record id="tax_monthly_report_line_vp7_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_monthly_report_line_vp7_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP7._applied_carryover_balance + VP7.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_monthly_report_line_vp8" model="account.report.line">
                        <field name="name">VP8 - Previous period credit</field>
                        <field name="code">VP8</field>
                        <field name="expression_ids">
                            <record id="tax_monthly_report_line_vp8_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp8</field>
                            </record>
                            <record id="tax_monthly_report_line_vp8_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_monthly_report_line_vp8_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP8._applied_carryover_balance + VP8.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_monthly_report_line_vp9" model="account.report.line">
                        <field name="name">VP9 - Previous year credit</field>
                        <field name="code">VP9</field>
                        <field name="expression_ids">
                            <record id="tax_monthly_report_line_vp9_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp9</field>
                            </record>
                            <record id="tax_monthly_report_line_vp9_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_monthly_report_line_vp9_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP9._applied_carryover_balance + VP9.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_monthly_report_line_vp10" model="account.report.line">
                        <field name="name">VP10 - EU car payments</field>
                        <field name="code">VP10</field>
                        <field name="tax_tags_formula">vp10</field>
                    </record>
                    <record id="tax_monthly_report_line_vp11" model="account.report.line">
                        <field name="name">VP11 - Tax Credit</field>
                        <field name="code">VP11</field>
                        <field name="tax_tags_formula">vp11</field>
                    </record>
                    <record id="tax_monthly_report_line_vp12" model="account.report.line">
                        <field name="name">VP12 - Interest due for quarterly settlements</field>
                        <field name="code">VP12</field>
                        <field name="tax_tags_formula">vp12</field>
                    </record>
                    <record id="tax_monthly_report_line_vp13" model="account.report.line">
                        <field name="name">VP13 - Down payment due</field>
                        <field name="code">VP13</field>
                        <field name="tax_tags_formula">vp13</field>
                    </record>
                </field>
            </record>
            <record id="tax_monthly_report_line_conto_corrente_iva" model="account.report.line">
                <field name="name">VAT account</field>
                <field name="code">h4</field>
                <field name="children_ids">
                    <record id="tax_monthly_report_line_vp14" model="account.report.line">
                        <field name="name">VP14 - VAT payable</field>
                        <field name="code">VP14</field>
                        <field name="children_ids">
                            <record id="tax_monthly_report_line_vp14a" model="account.report.line">
                                <field name="name">VP14a - VAT payable (debit)</field>
                                <field name="code">VP14a</field>
                                <field name="expression_ids">
                                    <record id="tax_monthly_report_line_vp14a_vp4_vp5_dif_pos" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_pos</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_monthly_report_line_vp14a_vp4_vp5_dif_neg" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_neg</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_monthly_report_line_vp14a_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(VP14a.vp4_vp5_dif_pos + VP7.balance + VP12.balance) - (-VP14a.vp4_vp5_dif_neg + VP8.balance + VP9.balance + VP10.balance + VP11.balance + VP13.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_monthly_report_line_vp14a_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP14a.balance</field>
                                        <field name="subformula">if_between(EUR(0), EUR(25.82))</field>
                                        <field name="carryover_target">VP7._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_monthly_report_line_vp14b" model="account.report.line">
                                <field name="name">VP14b - VAT payable (credit)</field>
                                <field name="code">VP14b</field>
                                <field name="expression_ids">
                                    <record id="tax_monthly_report_line_vp14b_vp4_vp5_dif_pos" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_pos</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_monthly_report_line_vp14b_vp4_vp5_dif_neg" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_neg</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_monthly_report_line_vp14b_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(-VP14b.vp4_vp5_dif_neg + VP8.balance + VP9.balance + VP10.balance + VP11.balance + VP13.balance) - (VP14b.vp4_vp5_dif_pos + VP7.balance + VP12.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_monthly_report_line_vp14b_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP14b.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                        <field name="carryover_target">VP8._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\annual_report_sections\va.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_annual_report_vat_va" model="account.report">
        <field name="name">VA VAT Report</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_annual_report_vat_balance_va" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
            <record id="tax_annual_report_vat_tax_va" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_annual_report_line_dag" model="account.report.line">
                <field name="name">Business Information</field>
                <field name="code">VA</field>
                <field name="children_ids">
                    <record id="tax_annual_report_line_dag_section1" model="account.report.line">
                        <field name="name">General analytical data</field>
                        <field name="code">VA_section1</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_va1" model="account.report.line">
                                <field name="name">VA1 - To be filled in by the entity of originator in cases of extraordinary transactions</field>
                                <field name="code">VA1</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_va1_4" model="account.report.line">
                                        <field name="name">VAT/2023 declaration credit transferred</field>
                                        <field name="code">VA1.4</field>
                                        <field name="expression_ids">
                                            <record id="tax_annual_report_line_va1_4_extval" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_va5" model="account.report.line">
                                <field name="name">VA5 - Terminals for mobile telecommunication radio service with more than 50% deduction</field>
                                <field name="code">VA5</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_va5_section_1" model="account.report.line">
                                        <field name="name">Equipment purchases</field>
                                        <field name="code">VA5_AA</field>
                                        <field name="expression_ids">
                                            <record id="tax_annual_report_line_va5_aa_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                            <record id="tax_annual_report_line_va5_aa_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">VA5_AA.balance * 0.5</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_annual_report_line_va5_section_2" model="account.report.line">
                                        <field name="name">Management services</field>
                                        <field name="code">VA5_SDG</field>
                                        <field name="expression_ids">
                                            <record id="tax_annual_report_line_va5_sdg_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                            <record id="tax_annual_report_line_va5_sdg_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">VA5_SDG.balance * 0.5</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_dag_section2" model="account.report.line">
                        <field name="name">Summary data for all activities</field>
                        <field name="code">VA_section2</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_va16" model="account.report.line">
                                <field name="name">VA11 - Data on amounts suspended as a result of the health emergency by COVID-19</field>
                                <field name="code">VA11</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_va16_extval" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_va12" model="account.report.line">
                                <field name="name">VA12 - Reserved for indication of surplus credit of former parent companies to be guaranteed</field>
                                <field name="code">VA12</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_va12_extval" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_va13" model="account.report.line">
                                <field name="name">VA13 - Transactions carried out with respect to condominiums</field>
                                <field name="code">VA13</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_va13_extval" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\annual_report_sections\ve.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_annual_report_vat_ve" model="account.report">
        <field name="name">VE VAT Report</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_annual_report_vat_balance_ve" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
            <record id="tax_annual_report_vat_tax_ve" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_annual_report_line_turnover" model="account.report.line">
                <field name="name">Turnover</field>
                <field name="code">VE</field>
                <field name="children_ids">
                    <record id="tax_annual_report_line_turnover_section_1" model="account.report.line">
                        <field name="name">Contributions of agricultural products and transfers from exempt farmers (in case of exceeding 1/3</field>
                        <field name="code">VE_section1</field>
                        <field name="foldable" eval="True"/>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_ve1" model="account.report.line">
                                <field name="name">VE1 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 2%</field>
                                <field name="code">VE1</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve1</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE1.balance * 0.02</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve2" model="account.report.line">
                                <field name="name">VE2 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 4%</field>
                                <field name="code">VE2</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve2</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE2.balance * 0.04</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve3" model="account.report.line">
                                <field name="name">VE3 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 6,4%</field>
                                <field name="code">VE3</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve3</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve3_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE3.balance * 0.064</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve4" model="account.report.line">
                                <field name="name">VE4 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 7,3%</field>
                                <field name="code">VE4</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve4</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve4_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE4.balance * 0.07</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve5" model="account.report.line">
                                <field name="name">VE5 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 7,5%</field>
                                <field name="code">VE5</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve5</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve5_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE5.balance * 0.073</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve6" model="account.report.line">
                                <field name="name">VE6 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 8,3%</field>
                                <field name="code">VE6</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve6</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve6_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE6.balance * 0.075</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve7" model="account.report.line">
                                <field name="name">VE7 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 8,5%</field>
                                <field name="code">VE7</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve7</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve7_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE7.balance * 0.083</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve8" model="account.report.line">
                                <field name="name">VE8 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 8,8%</field>
                                <field name="code">VE8</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve8</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve8_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE8.balance * 0.085</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve9" model="account.report.line">
                                <field name="name">VE9 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 9,5%</field>
                                <field name="code">VE9</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve9</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve9_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE9.balance * 0.088</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve10" model="account.report.line">
                                <field name="name">VE10 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 10%</field>
                                <field name="code">VE10</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve10</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve10_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE10.balance * 0.10</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve11" model="account.report.line">
                                <field name="name">VE11 - Transfers to cooperatives art.34 paragraph 2 with compensation percentage 12,3%</field>
                                <field name="code">VE11</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve11_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve11</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve11_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE11.balance * 0.123</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_turnover_section_2" model="account.report.line">
                        <field name="name">Agricultural taxable transactions (Article 34 paragraph 1) and commercial and professional taxable transactions</field>
                        <field name="code">VE_section2</field>
                        <field name="foldable" eval="True"/>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_ve20" model="account.report.line">
                                <field name="name">VE20 - Taxable transactions rate 4%</field>
                                <field name="code">VE20</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve20</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve20_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE20.balance * 0.4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve21" model="account.report.line">
                                <field name="name">VE21 - Taxable transactions rate 5%</field>
                                <field name="code">VE21</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve21</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve21_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE21.balance * 0.5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve22" model="account.report.line">
                                <field name="name">VE22 - Taxable transactions rate 10%</field>
                                <field name="code">VE22</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve22</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve22_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE22.balance * 0.10</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve23" model="account.report.line">
                                <field name="name">VE23 - Taxable transactions rate 22%</field>
                                <field name="code">VE23</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve23</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve23_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE23.balance * 0.22</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_turnover_section_3" model="account.report.line">
                        <field name="name">Total taxable income and tax</field>
                        <field name="code">VE_section3</field>
                        <field name="foldable" eval="True"/>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_ve24" model="account.report.line">
                                <field name="name">VE24 - Total lines VE1 to VE11 and lines VE20 to VE23</field>
                                <field name="code">VE24</field>
                                <field name="aggregation_formula">
                                            VE1.balance + VE2.balance + VE3.balance + VE4.balance + VE5.balance
                                            + VE6.balance + VE7.balance + VE8.balance + VE9.balance + VE10.balance
                                            + VE11.balance + VE20.balance + VE21.balance + VE22.balance + VE23.balance
                                </field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_ve24_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">
                                            VE1.tax + VE2.tax + VE3.tax + VE4.tax + VE5.tax
                                            + VE6.tax + VE7.tax + VE8.tax + VE9.tax + VE10.tax
                                            + VE11.tax + VE20.tax + VE21.tax + VE22.tax + VE23.tax
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve25" model="account.report.line">
                                <field name="name">VE25 - Variations and rounding (use &#43;/&#8722; sign)</field>
                                <field name="code">VE25</field>
                                <field name="tax_tags_formula">ve25</field>
                            </record>
                            <record id="tax_annual_report_line_ve26" model="account.report.line">
                                <field name="name">VE26 - Total VE24 and VE25</field>
                                <field name="code">VE26</field>
                                <field name="aggregation_formula">VE24.balance + VE25.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_turnover_section_4" model="account.report.line">
                        <field name="name">Other operations</field>
                        <field name="code">VE_section4</field>
                        <field name="foldable" eval="True"/>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_ve30" model="account.report.line">
                                <field name="name">VE30 - Transactions that contribute to the formation of the ceiling</field>
                                <field name="code">VE30</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_ve30_I" model="account.report.line">
                                        <field name="name">VE30_I - Total</field>
                                        <field name="code">VE30_I</field>
                                        <field name="aggregation_formula">VE30_II.balance + VE30_III.balance + VE30_IV.balance + VE30_V.balance</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve30_ii" model="account.report.line">
                                        <field name="name">VE30_II - Exports</field>
                                        <field name="code">VE30_II</field>
                                        <field name="tax_tags_formula">ve30_ii</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve30_iii" model="account.report.line">
                                        <field name="name">VE30_III - Intra-Community supplies</field>
                                        <field name="code">VE30_III</field>
                                        <field name="tax_tags_formula">ve30_iii</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve30_iv" model="account.report.line">
                                        <field name="name">VE30_IV - Transfers to San Marino</field>
                                        <field name="code">VE30_IV</field>
                                        <field name="tax_tags_formula">ve30_iv</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve30_v" model="account.report.line">
                                        <field name="name">VE30_V - Assimilated operations</field>
                                        <field name="code">VE30_V</field>
                                        <field name="tax_tags_formula">ve30_v</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve31" model="account.report.line">
                                <field name="name">VE31 - Non-taxable transactions as a result of declarations of intent</field>
                                <field name="code">VE31</field>
                                <field name="tax_tags_formula">ve31</field>
                            </record>
                            <record id="tax_annual_report_line_ve32" model="account.report.line">
                                <field name="name">VE32 - Other non-taxable transactions</field>
                                <field name="code">VE32</field>
                                <field name="tax_tags_formula">ve32</field>
                            </record>
                            <record id="tax_annual_report_line_ve33" model="account.report.line">
                                <field name="name">VE33 - Exempt transactions (art.10</field>
                                <field name="code">VE33</field>
                                <field name="tax_tags_formula">ve33</field>
                            </record>
                            <record id="tax_annual_report_line_ve34" model="account.report.line">
                                <field name="name">VE34 - Transactions not subject to the tax under Articles 7 to 7-septies</field>
                                <field name="code">VE34</field>
                                <field name="tax_tags_formula">ve34</field>
                            </record>
                            <record id="tax_annual_report_line_ve35" model="account.report.line">
                                <field name="name">VE35 - Transactions with application of internal reverse charge</field>
                                <field name="code">VE35</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_ve35_I" model="account.report.line">
                                        <field name="name">VE35_I - Total</field>
                                        <field name="code">VE35_I</field>
                                        <field name="aggregation_formula">
                                            VE35_II.balance + VE35_III.balance + VE35_IV.balance
                                            + VE35_V.balance + VE35_VI.balance + VE35_VII.balance
                                            + VE35_VIII.balance + VE35_IX.balance
                                        </field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_ii" model="account.report.line">
                                        <field name="name">VE35_II - Disposal of scrap and other recovered materials</field>
                                        <field name="code">VE35_II</field>
                                        <field name="tax_tags_formula">ve35_ii</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_iii" model="account.report.line">
                                        <field name="name">VE35_III - Disposals of pure gold and silver</field>
                                        <field name="code">VE35_III</field>
                                        <field name="tax_tags_formula">ve35_iii</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_iv" model="account.report.line">
                                        <field name="name">VE35_IV - Subcontracting in the construction industry</field>
                                        <field name="code">VE35_IV</field>
                                        <field name="tax_tags_formula">ve35_iv</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_v" model="account.report.line">
                                        <field name="name">VE35_V - Disposal of capital buildings</field>
                                        <field name="code">VE35_V</field>
                                        <field name="tax_tags_formula">ve35_v</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_vi" model="account.report.line">
                                        <field name="name">VE35_VI - Disposal of cell phones</field>
                                        <field name="code">VE35_VI</field>
                                        <field name="tax_tags_formula">ve35_vi</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_vii" model="account.report.line">
                                        <field name="name">VE35_VII - Disposal of electronic products</field>
                                        <field name="code">VE35_VII</field>
                                        <field name="tax_tags_formula">ve35_vii</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_viii" model="account.report.line">
                                        <field name="name">VE35_VIII - Benefits construction and related industries</field>
                                        <field name="code">VE35_VIII</field>
                                        <field name="tax_tags_formula">ve35_viii</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve35_ix" model="account.report.line">
                                        <field name="name">VE35_IX - Energy sector operations</field>
                                        <field name="code">VE35_IX</field>
                                        <field name="tax_tags_formula">ve35_ix</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve36" model="account.report.line">
                                <field name="name">VE36 - Non-taxable transactions made to earthquake victims</field>
                                <field name="code">VE36</field>
                                <field name="tax_tags_formula">ve36</field>
                            </record>
                            <record id="tax_annual_report_line_ve37" model="account.report.line">
                                <field name="name">VE37 - Transactions made during the year but with tax due in subsequent years</field>
                                <field name="code">VE37</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_ve37_I" model="account.report.line">
                                        <field name="name">VE37_I - Total</field>
                                        <field name="code">VE37_I</field>
                                        <field name="tax_tags_formula">ve37_i</field>
                                    </record>
                                    <record id="tax_annual_report_line_ve37_ii" model="account.report.line">
                                        <field name="name">VE37_II - ex art. 32-bis, DL no. 83/2012</field>
                                        <field name="code">VE37_II</field>
                                        <field name="tax_tags_formula">ve37_ii</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_ve38" model="account.report.line">
                                <field name="name">VE38 - Transactions with parties referred to in Article 17-ter.</field>
                                <field name="code">VE38</field>
                                <field name="tax_tags_formula">ve38</field>
                            </record>
                            <record id="tax_annual_report_line_ve39" model="account.report.line">
                                <field name="name">VE39 - (minus) Transactions made in previous years but with tax due in 2022</field>
                                <field name="code">VE39</field>
                                <field name="tax_tags_formula">ve39</field>
                            </record>
                            <record id="tax_annual_report_line_ve40" model="account.report.line">
                                <field name="name">VE40 - (minus) Disposals of depreciable assets and internal transfers.</field>
                                <field name="code">VE40</field>
                                <field name="tax_tags_formula">ve40</field>
                            </record>
                            <record id="tax_annual_report_line_ve45" model="account.report.line">
                                <field name="name">VE50 - TURNOVER</field>
                                <field name="code">VE50</field>
                                <field name="aggregation_formula">
                                    VE24.balance + VE30_I.balance + VE31.balance
                                    + VE32.balance + VE33.balance + VE34.balance
                                    + VE35_I.balance + VE36.balance + VE37_I.balance
                                    + VE37_II.balance + VE38.balance
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\annual_report_sections\vf.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_annual_report_vat_vf" model="account.report">
        <field name="name">VF VAT Report</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_annual_report_vat_balance_vf" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
            <record id="tax_annual_report_vat_tax_vf" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_annual_report_line_passive_op" model="account.report.line">
                <field name="name">Passive operations and VAT deduction allowed</field>
                <field name="code">VF</field>
                <field name="children_ids">
                    <record id="tax_annual_report_line_passive_op_1" model="account.report.line">
                        <field name="name">Purchases (Domestic, intra-Community and imports)</field>
                        <field name="code">VF_1</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_VF1" model="account.report.line">
                                <field name="name">VF1 - compensation percentage 2%</field>
                                <field name="code">VF1</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf1</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF1.balance * 0.02</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF2" model="account.report.line">
                                <field name="name">VF2 - compensation percentage 4%</field>
                                <field name="code">VF2</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf2</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF2.balance * 0.04</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF3" model="account.report.line">
                                <field name="name">VF3 - compensation percentage 5%</field>
                                <field name="code">VF3</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf3</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF3_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF3.balance * 0.5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF4" model="account.report.line">
                                <field name="name">VF4 - compensation percentage 6,4%</field>
                                <field name="code">VF4</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf4</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF4_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF4.balance * 0.064</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF5" model="account.report.line">
                                <field name="name">VF5 - compensation percentage 7%</field>
                                <field name="code">VF5</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf5</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF5_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF5.balance * 0.07</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF6" model="account.report.line">
                                <field name="name">VF6 - compensation percentage 7,3%</field>
                                <field name="code">VF6</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf6</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF6_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF6.balance * 0.073</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF7" model="account.report.line">
                                <field name="name">VF7 - compensation percentage 7,5%</field>
                                <field name="code">VF7</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf7</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF7_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF7.balance * 0.075</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF8" model="account.report.line">
                                <field name="name">VF8 - compensation percentage 8,3%</field>
                                <field name="code">VF8</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf8</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF8_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF8.balance * 0.083</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF9" model="account.report.line">
                                <field name="name">VF9 - compensation percentage 8,5%</field>
                                <field name="code">VF9</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf9</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF9_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF9.balance * 0.085</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF10" model="account.report.line">
                                <field name="name">VF10 - compensation percentage 8,8%</field>
                                <field name="code">VF10</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf10</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF10_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF10.balance * 0.088</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF11" model="account.report.line">
                                <field name="name">VF11 - compensation percentage 10%</field>
                                <field name="code">VF11</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF11_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf11</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF11_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF11.balance * 0.10</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF12" model="account.report.line">
                                <field name="name">VF12 - compensation percentage 12,3%</field>
                                <field name="code">VF12</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF12_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf12</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF12_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF12.balance * 0.123</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF13" model="account.report.line">
                                <field name="name">VF13 - compensation percentage 22%</field>
                                <field name="code">VF13</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf13</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF13_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF13.balance * 0.22</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF17" model="account.report.line">
                                <field name="name">VF17 - Purchases and imports without payment of tax, with use of the ceiling</field>
                                <field name="code">VF17</field>
                                <field name="tax_tags_formula">vf17</field>
                            </record>
                            <record id="tax_annual_report_line_VF18_I" model="account.report.line">
                                <field name="name">VF18 - Other non-tax purchases, not subject to tax and relating to certain special regimes</field>
                                <field name="code">VF18_I</field>
                                <field name="tax_tags_formula">vf18_I</field>
                            </record>
                            <record id="tax_annual_report_line_VF18_II" model="account.report.line">
                                <field name="name">VF18 - Exempt purchases and imports not subject to the tax</field>
                                <field name="code">VF18_II</field>
                                <field name="tax_tags_formula">vf18_II</field>
                            </record>
                            <record id="tax_annual_report_line_VF19" model="account.report.line">
                                <field name="name">VF19 - Purchases from subjects who have made use of concessional schemes</field>
                                <field name="code">VF19</field>
                                <field name="tax_tags_formula">vf19</field>
                            </record>
                            <record id="tax_annual_report_line_VF20" model="account.report.line">
                                <field name="name">VF20 - Purchases and imports not subject to the tax made by earthquake victims</field>
                                <field name="code">VF20</field>
                                <field name="tax_tags_formula">vf20</field>
                            </record>
                            <record id="tax_annual_report_line_VF21" model="account.report.line">
                                <field name="name">VF21 - Purchases and imports for which the deduction is excluded or reduced (art. 19-bis1)</field>
                                <field name="code">VF21</field>
                                <field name="tax_tags_formula">vf21</field>
                            </record>
                            <record id="tax_annual_report_line_VF22" model="account.report.line">
                                <field name="name">VF22 - Purchases and imports for which the deduction is not permitted</field>
                                <field name="code">VF22</field>
                                <field name="tax_tags_formula">vf22</field>
                            </record>
                            <record id="tax_annual_report_line_VF23" model="account.report.line">
                                <field name="name">VF23 - Purchases recorded in the year but with tax deduction deferred to subsequent years</field>
                                <field name="code">VF23</field>
                                <field name="tax_tags_formula">vf23</field>
                            </record>
                            <record id="tax_annual_report_line_VF24" model="account.report.line">
                                <field name="name">VF24 - Purchases recorded in previous years but with tax</field>
                                <field name="code">VF24</field>
                                <field name="tax_tags_formula">vf24</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_passive_op_2" model="account.report.line">
                        <field name="name">Total purchases and imports, total tax, purchases intra-community, imports and purchases</field>
                        <field name="code">VF_2</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_VF25" model="account.report.line">
                                <field name="name">VF25 - Total purchases and imports</field>
                                <field name="code">VF25</field>
                                <field name="aggregation_formula">VF1.tax + VF2.tax + VF3.tax + VF4.tax + VF5.tax + VF6.tax
                                                    + VF7.tax + VF8.tax + VF9.tax + VF10.tax + VF11.tax + VF12.tax
                                                    + VF13.tax + VF17.balance + VF18_I.balance + VF18_II.balance + VF19.balance
                                                    + VF20.balance + VF21.balance + VF22.balance + VF23.balance - VF24.balance</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF25_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF1.tax + VF2.tax + VF3.tax + VF4.tax + VF5.tax + VF6.tax + VF7.tax
                                                            + VF8.tax + VF9.tax + VF10.tax + VF11.tax + VF12.tax + VF13.tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF26" model="account.report.line">
                                <field name="name">VF26 - Tax variations and roundings (indicate with the +/- sign)</field>
                                <field name="code">VF26</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF26_tag" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf26</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF27" model="account.report.line">
                                <field name="name">VF27 - Total tax on taxable purchases and imports</field>
                                <field name="code">VF27</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF27_tag" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF25.tax + VF26.tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF28_I" model="account.report.line">
                                <field name="name">VF28 - Intra-community purchases</field>
                                <field name="code">VF28_I</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF28_I_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf28_i base</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF28_I_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf28_i tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF28_II" model="account.report.line">
                                <field name="name">VF28 - Imports</field>
                                <field name="code">VF28_II</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF28_II_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf28_ii base</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF28_II_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf28_ii tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF28_III" model="account.report.line">
                                <field name="name">VF28 - Purchases from San Marino</field>
                                <field name="code">VF28_III</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF28_III_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf28_iii base</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF28_III_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf28_iii tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF29" model="account.report.line">
                                <field name="name">VF29 - Total distribution of purchases and imports</field>
                                <field name="code">VF29</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_VF29_I" model="account.report.line">
                                        <field name="name">Depreciable assets</field>
                                        <field name="code">VF29_I</field>
                                        <field name="tax_tags_formula">vf29_i</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF29_II" model="account.report.line">
                                        <field name="name">Non-depreciable capital goods</field>
                                        <field name="code">VF29_II</field>
                                        <field name="tax_tags_formula">vf29_ii</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF29_III" model="account.report.line">
                                        <field name="name">Goods intended for resale or for the production of goods and services</field>
                                        <field name="code">VF29_III</field>
                                        <field name="tax_tags_formula">vf29_iii</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF29_IV" model="account.report.line">
                                        <field name="name">Other purchases and imports</field>
                                        <field name="code">VF29_IV</field>
                                        <field name="tax_tags_formula">vf29_iv</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_passive_op_3a" model="account.report.line">
                        <field name="name">Exempt operations</field>
                        <field name="code">VF_3A</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_VF31" model="account.report.line">
                                <field name="name">VF31 - Purchases intended for occasional taxable transactions</field>
                                <field name="code">VF31</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF31_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf31</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF31_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF1.tax + VF2.tax + VF3.tax + VF4.tax + VF5.tax + VF6.tax + VF7.tax
                                                            + VF8.tax + VF9.tax + VF10.tax + VF11.tax + VF12.tax + VF13.tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF34" model="account.report.line">
                                <field name="name">VF34 - Data for calculating the deduction percentage</field>
                                <field name="code">VF34</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_VF34_I" model="account.report.line">
                                        <field name="name">Exempt transactions relating to gold from investments made by the subjects referred to in the art. 19, co. 3, letter. d)</field>
                                        <field name="code">VF34_I</field>
                                        <field name="tax_tags_formula">vf34_i</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_II" model="account.report.line">
                                        <field name="name">Exempt operations referred to in nos. from 1 to 9 of the art. 10 not included in their own business of the company or ancillary to taxable operations</field>
                                        <field name="code">VF34_II</field>
                                        <field name="tax_tags_formula">vf34_ii</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_III" model="account.report.line">
                                        <field name="name">Exempt operations referred to in art. 10, n. 27-quinquies</field>
                                        <field name="code">VF34_III</field>
                                        <field name="tax_tags_formula">vf34_iii</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_IV" model="account.report.line">
                                        <field name="name">Depreciable assets and transfers interiors exempt</field>
                                        <field name="code">VF34_IV</field>
                                        <field name="tax_tags_formula">vf34_iv</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_V" model="account.report.line">
                                        <field name="name">Operations not subject</field>
                                        <field name="code">VF34_V</field>
                                        <field name="tax_tags_formula">vf34_v</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_VI" model="account.report.line">
                                        <field name="name">Operations not subject to art. 74, co. 1</field>
                                        <field name="code">VF34_VI</field>
                                        <field name="tax_tags_formula">vf34_vi</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_VII" model="account.report.line">
                                        <field name="name">Exempt operations art. 19, co. 3, letter. a-bis) and d-bis)</field>
                                        <field name="code">VF34_VII</field>
                                        <field name="tax_tags_formula">vf34_vii</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_VIII" model="account.report.line">
                                        <field name="name">Article operations 7 to 7-septies without right to deduction</field>
                                        <field name="code">VF34_VIII</field>
                                        <field name="tax_tags_formula">vf34_viii</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF34_IX" model="account.report.line">
                                        <field name="name">Operations exempted by law no. 178/2020</field>
                                        <field name="code">VF34_IX</field>
                                        <field name="tax_tags_formula">vf34_ix</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF35" model="account.report.line">
                                <field name="name">VF35 - VAT not paid on purchases and imports indicated</field>
                                <field name="code">VF35</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF35_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf35</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF36" model="account.report.line">
                                <field name="name">VF36 - VAT deductible for gold purchases made by parties other than producers and transformers pursuant to art. 19, paragraph 5 bis</field>
                                <field name="code">VF36</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF36_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf36</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF37" model="account.report.line">
                                <field name="name">VF37 - VAT allowed as deduction</field>
                                <field name="code">VF37</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF37_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">((VF27.tax + VF35.tax - VF36.tax) * (
                                            VF34_I.balance + VF34_II.balance + VF34_III.balance + VF34_IV.balance + VF34_V.balance +
                                            VF34_VI.balance + VF34_VII.balance + VF34_VIII.balance + VF34_IX.balance
                                        )) - VF35.tax + VF36.tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_passive_op_3b" model="account.report.line">
                        <field name="name">Agricultural businesses (art.34)</field>
                        <field name="code">VF_3B</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_VF38" model="account.report.line">
                                <field name="name">VF38 - Reserved for mixed agricultural enterprises</field>
                                <field name="code">VF38</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF38_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf38 Base</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF38_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf38 Tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF39" model="account.report.line">
                                <field name="name">VF39 - compensation percentage 2%</field>
                                <field name="code">VF39</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF39_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf39</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF39_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF39.balance * 0.02</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF40" model="account.report.line">
                                <field name="name">VF40 - compensation percentage 4%</field>
                                <field name="code">VF40</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF40_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf40</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF40_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF40.balance * 0.04</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF41" model="account.report.line">
                                <field name="name">VF41 - compensation percentage 6.4%</field>
                                <field name="code">VF41</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF41_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf41</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF41_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF41.balance * 0.064</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF42" model="account.report.line">
                                <field name="name">VF42 - compensation percentage 7%</field>
                                <field name="code">VF42</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF42_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf42</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF42_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF42.balance * 0.07</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF43" model="account.report.line">
                                <field name="name">VF43 - compensation percentage 7,3%</field>
                                <field name="code">VF43</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF43_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf43</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF43_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF43.balance * 0.073</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF44" model="account.report.line">
                                <field name="name">VF44 - compensation percentage 7,5%</field>
                                <field name="code">VF44</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF44_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf44</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF44_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF44.balance * 0.075</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF45" model="account.report.line">
                                <field name="name">VF45 - compensation percentage 8,3%</field>
                                <field name="code">VF45</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF45_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf45</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF45_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF45.balance * 0.083</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF46" model="account.report.line">
                                <field name="name">VF46 - compensation percentage 8,5%</field>
                                <field name="code">VF46</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF46_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf46</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF46_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF46.balance * 0.085</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF47" model="account.report.line">
                                <field name="name">VF47 - compensation percentage 9.5%</field>
                                <field name="code">VF47</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF47_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf47</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF47_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF47.balance * 0.095</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF48" model="account.report.line">
                                <field name="name">VF48 - compensation percentage 10%</field>
                                <field name="code">VF48</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF48_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf48</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF48_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF48.balance * 0.10</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF49" model="account.report.line">
                                <field name="name">VF49 - compensation percentage 12,3%</field>
                                <field name="code">VF49</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF49_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf49</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF49_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF49.balance * 0.123</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF51" model="account.report.line">
                                <field name="name">VF51 - Tax variations and roundings (indicate with the +/- sign)</field>
                                <field name="code">VF51</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF51_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf51</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF52" model="account.report.line">
                                <field name="name">VF52 - TOTALS Algebraic sum of lines from VF39 to VF51</field>
                                <field name="code">VF52</field>
                                <field name="aggregation_formula">VF39.balance + VF40.balance + VF41.balance + VF42.balance + VF43.balance +
                                                        VF44.balance + VF45.balance + VF46.balance + VF47.balance + VF48.balance + 
                                                        VF49.balance + VF51.tax</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF52_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF39.tax + VF40.tax + VF41.tax + VF42.tax + VF43.tax +
                                                              VF44.tax + VF45.tax + VF46.tax + VF47.tax + VF48.tax +
                                                              VF49.tax + VF51.tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF53" model="account.report.line">
                                <field name="name">VF53 - Deductible VAT charged to the operations referred to in line VF38</field>
                                <field name="code">VF53</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF53_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf53</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF54" model="account.report.line">
                                <field name="name">VF54 - Deductible amount for transfers, including intra-community, of agricultural products referred to in art. 34</field>
                                <field name="code">VF54</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF54_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf54</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF55" model="account.report.line">
                                <field name="name">VF55 - TOTAL VAT allowed as deduction</field>
                                <field name="code">VF55</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF55_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF52.tax + VF53.tax + VF54.tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_passive_op_3c" model="account.report.line">
                        <field name="name">Special cases</field>
                        <field name="code">VF_3C</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_VF62" model="account.report.line">
                                <field name="name">VF62 - Reserved for agricultural businesses</field>
                                <field name="code">VF62</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF62_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vf62</field>
                                    </record>
                                    <record id="tax_annual_report_line_VF62_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF62.balance * 0.5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_passive_op_4" model="account.report.line">
                        <field name="name">VAT allowed as deduction</field>
                        <field name="code">VF_4</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_VF70" model="account.report.line">
                                <field name="name">VF70 - TOTAL adjustments (indicate with the +/- sign)</field>
                                <field name="code">VF70</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF70_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VF70</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_VF71" model="account.report.line">
                                <field name="name">VF71 - VAT allowed as deduction</field>
                                <field name="code">VF71</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_VF71_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF26.tax + VF27.tax + VF37.tax + VF55.tax + VF70.tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\annual_report_sections\vh.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_annual_report_vat_vh" model="account.report">
        <field name="name">VH VAT Report</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_annual_report_vat_balance_vh" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_annual_report_line_vari_communi_perd" model="account.report.line">
                <field name="name">Changes in reporting periodicals</field>
                <field name="code">VH</field>
                <field name="children_ids">
                    <record id="tax_annual_report_line_vh1" model="account.report.line">
                        <field name="name">VH1 - January</field>
                        <field name="code">VH1</field>
                        <field name="tax_tags_formula">vh1</field>
                    </record>
                    <record id="tax_annual_report_line_vh2" model="account.report.line">
                        <field name="name">VH2 - February</field>
                        <field name="code">VH2</field>
                        <field name="tax_tags_formula">v2</field>
                    </record>
                    <record id="tax_annual_report_line_vh3" model="account.report.line">
                        <field name="name">VH3 - March</field>
                        <field name="code">VH3</field>
                        <field name="tax_tags_formula">vh3</field>
                    </record>
                    <record id="tax_annual_report_line_vh4" model="account.report.line">
                        <field name="name">VH4 - QUARTER I</field>
                        <field name="code">VH4</field>
                        <field name="aggregation_formula">VH1.balance + VH2.balance + VH3.balance</field>
                    </record>
                    <record id="tax_annual_report_line_vh5" model="account.report.line">
                        <field name="name">VH5 - April</field>
                        <field name="code">VH5</field>
                        <field name="tax_tags_formula">vh5</field>
                    </record>
                    <record id="tax_annual_report_line_vh6" model="account.report.line">
                        <field name="name">VH6 - May</field>
                        <field name="code">VH6</field>
                        <field name="tax_tags_formula">vh6</field>
                    </record>
                    <record id="tax_annual_report_line_vh7" model="account.report.line">
                        <field name="name">VH7 - June</field>
                        <field name="code">VH7</field>
                        <field name="tax_tags_formula">vh7</field>
                    </record>
                    <record id="tax_annual_report_line_vh8" model="account.report.line">
                        <field name="name">VH8 - QUARTER II</field>
                        <field name="code">VH8</field>
                        <field name="aggregation_formula">VH5.balance + VH6.balance + VH7.balance</field>
                    </record>
                    <record id="tax_annual_report_line_vh9" model="account.report.line">
                        <field name="name">VH9 - July</field>
                        <field name="code">VH9</field>
                        <field name="tax_tags_formula">vh9</field>
                    </record>
                    <record id="tax_annual_report_line_vh10" model="account.report.line">
                        <field name="name">VH10 - August</field>
                        <field name="code">VH10</field>
                        <field name="tax_tags_formula">vh10</field>
                    </record>
                    <record id="tax_annual_report_line_vh11" model="account.report.line">
                        <field name="name">VH11 - September</field>
                        <field name="code">VH11</field>
                        <field name="tax_tags_formula">vh11</field>
                    </record>
                    <record id="tax_annual_report_line_vh12" model="account.report.line">
                        <field name="name">VH12 - QUARTER III</field>
                        <field name="code">VH12</field>
                        <field name="aggregation_formula">VH9.balance + VH10.balance + VH11.balance</field>
                    </record>
                    <record id="tax_annual_report_line_vh13" model="account.report.line">
                        <field name="name">VH13 - October</field>
                        <field name="code">VH13</field>
                        <field name="tax_tags_formula">vh13</field>
                    </record>
                    <record id="tax_annual_report_line_vh14" model="account.report.line">
                        <field name="name">VH14 - November</field>
                        <field name="code">VH14</field>
                        <field name="tax_tags_formula">vh14</field>
                    </record>
                    <record id="tax_annual_report_line_vh15" model="account.report.line">
                        <field name="name">VH15 - December</field>
                        <field name="code">VH15</field>
                        <field name="tax_tags_formula">vh15</field>
                    </record>
                    <record id="tax_annual_report_line_vh16" model="account.report.line">
                        <field name="name">VH16 - QUARTER IV</field>
                        <field name="code">VH16</field>
                        <field name="aggregation_formula">VH13.balance + VH14.balance + VH15.balance</field>
                    </record>
                    <record id="tax_annual_report_line_vh17" model="account.report.line">
                        <field name="name">VH17 - Deposit due</field>
                        <field name="code">VH17</field>
                        <field name="aggregation_formula">
                            VH4.balance + VH8.balance + VH12.balance + VH16.balance
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\annual_report_sections\vj.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_annual_report_vat_vj" model="account.report">
        <field name="name">VJ VAT Report</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_annual_report_vat_balance_vj" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_annual_report_line_reverse_charge_iva" model="account.report.line">
                <field name="name">Reverse Charge</field>
                <field name="code">VJ</field>
                <field name="children_ids">
                    <record id="tax_annual_report_line_vj1" model="account.report.line">
                        <field name="name">VJ1 - Purchases of goods from Vatican City and San Marino</field>
                        <field name="code">VJ1</field>
                        <field name="tax_tags_formula">vj1</field>
                    </record>
                    <record id="tax_annual_report_line_vj2" model="account.report.line">
                        <field name="name">VJ2 - Extraction of goods from VAT warehouses</field>
                        <field name="code">VJ2</field>
                        <field name="tax_tags_formula">vj2</field>
                    </record>
                    <record id="tax_annual_report_line_vj3" model="account.report.line">
                        <field name="name">VJ3 - Purchases of goods already in Italy or services, from non-residents</field>
                        <field name="code">VJ3</field>
                        <field name="tax_tags_formula">vj3</field>
                    </record>
                    <record id="tax_annual_report_line_vj4" model="account.report.line">
                        <field name="name">VJ4 - Fees paid to resellers of travel tickets and resellers of parking documents</field>
                        <field name="code">VJ4</field>
                        <field name="tax_tags_formula">vj4</field>
                    </record>
                    <record id="tax_annual_report_line_vj5" model="account.report.line">
                        <field name="name">VJ5 - Commissions paid by travel agents to their intermediaries</field>
                        <field name="code">VJ5</field>
                        <field name="tax_tags_formula">vj5</field>
                    </record>
                    <record id="tax_annual_report_line_vj6" model="account.report.line">
                        <field name="name">VJ6 - Purchases of scrap and other recovered materials</field>
                        <field name="code">VJ6</field>
                        <field name="tax_tags_formula">vj6</field>
                    </record>
                    <record id="tax_annual_report_line_vj7" model="account.report.line">
                        <field name="name">VJ7 - Purchases of industrial gold and pure silver made in Italy</field>
                        <field name="code">VJ7</field>
                        <field name="tax_tags_formula">vj7</field>
                    </record>
                    <record id="tax_annual_report_line_vj8" model="account.report.line">
                        <field name="name">VJ8 - Investment gold purchases made in Italy</field>
                        <field name="code">VJ8</field>
                        <field name="tax_tags_formula">vj8</field>
                    </record>
                    <record id="tax_annual_report_line_vj9" model="account.report.line">
                        <field name="name">VJ9 - Intra-EU Purchases of Goods</field>
                        <field name="code">VJ9</field>
                        <field name="tax_tags_formula">vj9</field>
                    </record>
                    <record id="tax_annual_report_line_vj10" model="account.report.line">
                        <field name="name">VJ10 - Imports of scrap and other recovered materials</field>
                        <field name="code">VJ10</field>
                        <field name="tax_tags_formula">vj10</field>
                    </record>
                    <record id="tax_annual_report_line_vj11" model="account.report.line">
                        <field name="name">VJ11 - Imports of industrial gold and pure silver</field>
                        <field name="code">VJ11</field>
                        <field name="tax_tags_formula">vj11</field>
                    </record>
                    <record id="tax_annual_report_line_vj12" model="account.report.line">
                        <field name="name">VJ12 - Subcontracting of services in the construction field</field>
                        <field name="code">VJ12</field>
                        <field name="tax_tags_formula">vj12</field>
                    </record>
                    <record id="tax_annual_report_line_vj13" model="account.report.line">
                        <field name="name">VJ13 - Purchases of buildings or portions of buildings used for capital purposes</field>
                        <field name="code">VJ13</field>
                        <field name="tax_tags_formula">vj13</field>
                    </record>
                    <record id="tax_annual_report_line_vj14" model="account.report.line">
                        <field name="name">VJ14 - Purchases of cell phones</field>
                        <field name="code">VJ14</field>
                        <field name="tax_tags_formula">vj14</field>
                    </record>
                    <record id="tax_annual_report_line_vj15" model="account.report.line">
                        <field name="name">VJ15 - Purchases of electronic products</field>
                        <field name="code">VJ15</field>
                        <field name="tax_tags_formula">vj15</field>
                    </record>
                    <record id="tax_annual_report_line_vj16" model="account.report.line">
                        <field name="name">VJ16 - Provision of services in the construction field</field>
                        <field name="code">VJ16</field>
                        <field name="tax_tags_formula">vj16</field>
                    </record>
                    <record id="tax_annual_report_line_vj17" model="account.report.line">
                        <field name="name">VJ17 - Purchases of energy sector goods and services</field>
                        <field name="code">VJ17</field>
                        <field name="tax_tags_formula">vj17</field>
                    </record>
                    <record id="tax_annual_report_line_vj18" model="account.report.line">
                        <field name="name">VJ18 - Purchases made by VAT-registered public administrations</field>
                        <field name="code">VJ18</field>
                        <field name="tax_tags_formula">vj18</field>
                    </record>
                    <record id="tax_annual_report_line_vj19" model="account.report.line">
                        <field name="name">VJ19 - Total frame VJ</field>
                        <field name="code">VJ19</field>
                        <field name="aggregation_formula">VJ1.balance + VJ2.balance + VJ3.balance + VJ4.balance + VJ5.balance + VJ6.balance + VJ7.balance +
                                                VJ8.balance + VJ9.balance + VJ10.balance + VJ11.balance + VJ12.balance + VJ13.balance + VJ14.balance +
                                                VJ15.balance + VJ16.balance + VJ17.balance + VJ18.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\annual_report_sections\vl.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_annual_report_vat_vl" model="account.report">
        <field name="name">VL VAT Report</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_annual_report_vat_balance_vl" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_annual_report_line_comp_fram" model="account.report.line">
                <field name="name">Annual tax for completed schedules</field>
                <field name="code">VL</field>
                <field name="children_ids">
                    <record id="tax_annual_report_line_comp_fram_1" model="account.report.line">
                        <field name="name">Determination of VAT due or credit for the tax period</field>
                        <field name="code">VL_1</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_vl1" model="account.report.line">
                                <field name="name">VL1 - VAT payable</field>
                                <field name="code">VL1</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_vl1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VE26.balance + VJ19.balance</field>
                                        <field name="subformula">cross_report</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_vl2" model="account.report.line">
                                <field name="name">VL2 - VAT deductible</field>
                                <field name="code">VL2</field>
                                <field name="expression_ids">
                                    <record id="tax_annual_report_line_vl2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VF71.tax</field>
                                        <field name="subformula">cross_report</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_vl3" model="account.report.line">
                                <field name="name">VL3 - Tax Due</field>
                                <field name="code">VL3</field>
                                <field name="aggregation_formula">VL1.balance - VL2.balance</field>
                            </record>
                            <record id="tax_annual_report_line_vl4" model="account.report.line">
                                <field name="name">VL4 - Tax Credit</field>
                                <field name="code">VL4</field>
                                <field name="aggregation_formula">VL2.balance - VL1.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_comp_fram_2" model="account.report.line">
                        <field name="name">Previous year credit</field>
                        <field name="code">VL_2</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_vl8" model="account.report.line">
                                <field name="name">VL8 - Credit resulting from the previous year's declaration</field>
                                <field name="code">VL8</field>
                                <field name="tax_tags_formula">vl8</field>
                            </record>
                            <record id="tax_annual_report_line_vl9" model="account.report.line">
                                <field name="name">VL9 - Credit offset in the template</field>
                                <field name="code">VL9</field>
                                <field name="tax_tags_formula">vl9</field>
                            </record>
                            <record id="tax_annual_report_line_vl10" model="account.report.line">
                                <field name="name">VL10 - Non-transferable credit surplus</field>
                                <field name="code">VL10</field>
                                <field name="tax_tags_formula">vl10</field>
                            </record>
                            <record id="tax_annual_report_line_vl11" model="account.report.line">
                                <field name="name">VL11 - Credits art. 8, paragraph 6-quater, Presidential Decree. n. 322/98</field>
                                <field name="code">VL11</field>
                                <field name="tax_tags_formula">vl11</field>
                            </record>
                            <record id="tax_annual_report_line_vl12" model="account.report.line">
                                <field name="name">VL12 - Periodic payments omitted</field>
                                <field name="code">VL12</field>
                                <field name="tax_tags_formula">vl12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_annual_report_line_comp_fram_3" model="account.report.line">
                        <field name="name">Determination of debit or credit VAT relating to all activities carried out</field>
                        <field name="code">VL_3</field>
                        <field name="children_ids">
                            <record id="tax_annual_report_line_vl20" model="account.report.line">
                                <field name="name">VL20 - Interim refunds required</field>
                                <field name="code">VL20</field>
                                <field name="tax_tags_formula">vl20</field>
                            </record>
                            <record id="tax_annual_report_line_vl21" model="account.report.line">
                                <field name="name">VL21 - Amount of credits transferred</field>
                                <field name="code">VL21</field>
                                <field name="tax_tags_formula">vl21</field>
                            </record>
                            <record id="tax_annual_report_line_vl22" model="account.report.line">
                                <field name="name">VL22 - VAT credit resulting from the first 3 quarters</field>
                                <field name="code">VL22</field>
                                <field name="tax_tags_formula">vl22</field>
                            </record>
                            <record id="tax_annual_report_line_vl23" model="account.report.line">
                                <field name="name">VL23 - Interest due for quarterly payments</field>
                                <field name="code">VL23</field>
                                <field name="tax_tags_formula">vl23</field>
                            </record>
                            <record id="tax_annual_report_line_vl24" model="account.report.line">
                                <field name="name">VL24 - Previous year transfers returned by the parent company</field>
                                <field name="code">VL24</field>
                                <field name="tax_tags_formula">vl24</field>
                            </record>
                            <record id="tax_annual_report_line_vl25" model="account.report.line">
                                <field name="name">VL25 - Previous year credit surplus</field>
                                <field name="code">VL25</field>
                                <field name="aggregation_formula">VL8.balance - VL9.balance</field>
                            </record>
                            <record id="tax_annual_report_line_vl26" model="account.report.line">
                                <field name="name">VL26 - Credit requested for reimbursement in previous years computable as a deduction following refusal by the office</field>
                                <field name="code">VL26</field>
                                <field name="tax_tags_formula">vl26</field>
                            </record>
                            <record id="tax_annual_report_line_vl27" model="account.report.line">
                                <field name="name">VL27 - Tax credits used in periodic payments and for the advance payment</field>
                                <field name="code">VL27</field>
                                <field name="tax_tags_formula">vl27</field>
                            </record>
                            <record id="tax_annual_report_line_vl28" model="account.report.line">
                                <field name="name">VL28 - Credits received from savings management companies used in periodic payments and for the advance payment</field>
                                <field name="code">VL28</field>
                                <field name="tax_tags_formula">vl28</field>
                            </record>
                            <record id="tax_annual_report_line_vl29" model="account.report.line">
                                <field name="name">VL29 - EU car payments relating to sales carried out during the year</field>
                                <field name="code">VL29</field>
                                <field name="tax_tags_formula">vl29</field>
                            </record>
                            <record id="tax_annual_report_line_vl30" model="account.report.line">
                                <field name="name">VL30 - Periodic VAT amount</field>
                                <field name="code">VL30</field>
                                <field name="children_ids">
                                    <record id="tax_annual_report_line_vl30_I" model="account.report.line">
                                        <field name="name">Periodic VAT due</field>
                                        <field name="code">VL30_I</field>
                                        <field name="tax_tags_formula">vl30_i</field>
                                    </record>
                                    <record id="tax_annual_report_line_vl30_II" model="account.report.line">
                                        <field name="name">Periodic VAT paid</field>
                                        <field name="code">VL30_II</field>
                                        <field name="tax_tags_formula">vl30_ii</field>
                                    </record>
                                    <record id="tax_annual_report_line_vl30_III" model="account.report.line">
                                        <field name="name">Periodic VAT paid following notification of irregularities</field>
                                        <field name="code">VL30_III</field>
                                        <field name="tax_tags_formula">vl30_iii</field>
                                    </record>
                                    <record id="tax_annual_report_line_vl30_IV" model="account.report.line">
                                        <field name="name">Periodic VAT paid following payment orders</field>
                                        <field name="code">VL30_IV</field>
                                        <field name="tax_tags_formula">vl30_iv</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_annual_report_line_vl31" model="account.report.line">
                                <field name="name">VL31 - Amount of debts transferred</field>
                                <field name="code">VL31</field>
                                <field name="tax_tags_formula">vl31</field>
                            </record>
                            <record id="tax_annual_report_line_vl32" model="account.report.line">
                                <field name="name">VL32 - VAT DUE</field>
                                <field name="code">VL32</field>
                                <field name="aggregation_formula">(VL3.balance + VL20.balance + VL21.balance + VL22.balance + VL23.balance)
                                                    - (VL4.balance + VL11.balance + VL24.balance + VL25.balance + VL26.balance +
                                                        VL27.balance + VL28.balance + VL29.balance + VL30_I.balance + VL30_II.balance +
                                                        VL30_III.balance + VL30_IV.balance + VL31.balance)</field>
                            </record>
                            <record id="tax_annual_report_line_vl33" model="account.report.line">
                                <field name="name">VL33 - VAT CREDIT</field>
                                <field name="code">VL33</field>
                                <field name="aggregation_formula">(VL4.balance + VL11.balance + VL24.balance + VL25.balance + VL26.balance +
                                                        VL27.balance + VL28.balance + VL29.balance + VL30_I.balance + VL30_II.balance +
                                                        VL30_III.balance + VL30_IV.balance + VL31.balance) - (VL3.balance + VL20.balance +
                                                        VL21.balance + VL22.balance + VL23.balance)</field>
                            </record>
                            <record id="tax_annual_report_line_vl34" model="account.report.line">
                                <field name="name">VL34 - Tax credits used in the annual return</field>
                                <field name="code">VL34</field>
                                <field name="tax_tags_formula">vl34</field>
                            </record>
                            <record id="tax_annual_report_line_vl35" model="account.report.line">
                                <field name="name">VL35 - Credits received from asset management companies used in the annual declaration</field>
                                <field name="code">VL35</field>
                                <field name="tax_tags_formula">vl35</field>
                            </record>
                            <record id="tax_annual_report_line_vl36" model="account.report.line">
                                <field name="name">VL36 - Interest due on the annual return</field>
                                <field name="code">VL36</field>
                                <field name="tax_tags_formula">vl36</field>
                            </record>
                            <record id="tax_annual_report_line_vl37" model="account.report.line">
                                <field name="name">VL37 - Credit transferred by savings management companies pursuant to art. 8 of the legislative decree n. 351/2001</field>
                                <field name="code">VL37</field>
                                <field name="tax_tags_formula">vl37</field>
                            </record>
                            <record id="tax_annual_report_line_vl38" model="account.report.line">
                                <field name="name">VL38 - TOTAL VAT DUE</field>
                                <field name="code">VL38</field>
                                <field name="aggregation_formula">VL32.balance - VL34.balance - VL35.balance + VL36.balance</field>
                            </record>
                            <record id="tax_annual_report_line_vl39" model="account.report.line">
                                <field name="name">VL39 - TOTAL VAT INPUT</field>
                                <field name="code">VL39</field>
                                <field name="aggregation_formula">VL33.balance - VL37.balance</field>
                            </record>
                            <record id="tax_annual_report_line_vl40" model="account.report.line">
                                <field name="name">VL40 - Payments made following excess use of credit</field>
                                <field name="code">VL40</field>
                                <field name="tax_tags_formula">vl40</field>
                            </record>
                            <record id="tax_annual_report_line_vl41_I" model="account.report.line">
                                <field name="name">VL41 - Difference between periodic VAT due and periodic VAT paid</field>
                                <field name="code">VL41_I</field>
                                <field name="aggregation_formula">VL30_I.balance - (VL30_II.balance + VL30_III.balance + VL30_IV.balance)</field>
                            </record>
                            <record id="tax_annual_report_line_vl41_II" model="account.report.line">
                                <field name="name">VL41 - Difference between credit potential and actual credit</field>
                                <field name="code">VL41_II</field>
                                <field name="aggregation_formula">(VL4.balance + VL11.balance + VL12.balance + VL24.balance +
                                                        VL25.balance + VL26.balance + VL27.balance + VL28.balance +
                                                        VL29.balance + VL30_I.balance + VL31.balance) - (VL3.balance + VL20.balance +
                                                        VL21.balance + VL22.balance + VL23.balance)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-it.csv

```csv
"id","code","name","account_type","reconcile","tag_ids","name@it"
"1101","1101","Planting costs","asset_non_current","False","l10n_it.account_tag_B_ATT","Costi di impianto"
"1106","1106","Software","asset_non_current","False","l10n_it.account_tag_B_ATT","Software"
"1108","1108","Startup","asset_non_current","False","l10n_it.account_tag_B_ATT","Avviamento"
"1111","1111","Provision for depreciation of start-up costs","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento costi di impianto"
"1116","1116","Provision for software amortization","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento software"
"1118","1118","Goodwill amortization provision","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento avviamento"
"1201","1201","Buildings","asset_non_current","False","l10n_it.account_tag_B_ATT","Fabbricati"
"1202","1202","Plant and machinery","asset_non_current","False","l10n_it.account_tag_B_ATT","Impianti e macchinari"
"1204","1204","Commercial equipment","asset_non_current","False","l10n_it.account_tag_B_ATT","Attrezzature commerciali"
"1205","1205","Office machines","asset_non_current","False","l10n_it.account_tag_B_ATT","Macchine d'ufficio"
"1206","1206","Furniture","asset_non_current","False","l10n_it.account_tag_B_ATT","Arredamento"
"1207","1207","Vehicles","asset_non_current","False","l10n_it.account_tag_B_ATT","Automezzi"
"1208","1208","Durable packaging","asset_non_current","False","l10n_it.account_tag_B_ATT","Imballaggi durevoli"
"1211","1211","Building depreciation fund","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento fabbricati"
"1212","1212","Provision for depreciation of plant and machinery","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento impianti e macchinari"
"1214","1214","Allowance for depreciation of business equipment","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento attrezzature commerciali"
"1215","1215","Provision for depreciation of office machines","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento macchine d'ufficio"
"1216","1216","Furniture depreciation fund","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento arredamento"
"1217","1217","Provision for vehicle depreciation","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento automezzi"
"1218","1218","Durable packaging depreciation fund","asset_non_current","False","l10n_it.account_tag_B_ATT","Fondo ammortamento imballaggi durevoli"
"1220","1220","Fixed assets suppliers on account","asset_non_current","False","l10n_it.account_tag_B_ATT","Fornitori immobilizzazioni c/acconti"
"1301","1301","Active mortgages","asset_fixed","False","l10n_it.account_tag_B_ATT","Mutui attivi"
"1401","1401","Consumable materials","asset_current","False","l10n_it.account_tag_C_ATT","Materie di consumo"
"1404","1404","Goods","asset_current","False","l10n_it.account_tag_C_ATT","Merci"
"1410","1410","Suppliers down payments","asset_current","False","l10n_it.account_tag_C_ATT","Acconti dei fornitori"
"1501","1501","Customer receivables","asset_receivable","True","l10n_it.account_tag_C_ATT","Crediti verso i clienti"
"1502","1502","Miscellaneous trade receivables","asset_current","True","l10n_it.account_tag_C_ATT","Crediti commerciali diversi"
"1503","1503","Clients expenses in advance","asset_current","True","l10n_it.account_tag_C_ATT","Spese dei clienti in anticipo"
"1505","1505","Bills of exchange receivable","asset_current","True","l10n_it.account_tag_C_ATT","Cambiali attive"
"1506","1506","Change them at a discount","asset_current","True","l10n_it.account_tag_C_ATT","Cambiali allo sconto"
"1507","1507","Change in collection","asset_current","True","l10n_it.account_tag_C_ATT","Cambiali all'incasso"
"1508","1508","Accounts receivable (PoS)","asset_receivable","True","l10n_it.account_tag_C_ATT","Crediti v/clienti (PoS)"
"1509","1509","Invoices to be issued","asset_current","True","l10n_it.account_tag_C_ATT","Fatture da emettere"
"1510","1510","Outstanding receivables","asset_current","True","l10n_it.account_tag_C_ATT","Crediti insoluti"
"1511","1511","Outstanding bills of exchange","asset_current","True","l10n_it.account_tag_C_ATT","Cambiali insolute"
"1531","1531","Receivables to be settled","asset_current","True","l10n_it.account_tag_C_ATT","Crediti da liquidare"
"1540","1540","Allowance for doubtful accounts","asset_current","True","l10n_it.account_tag_C_ATT","Fondo svalutazione crediti"
"1541","1541","Provision for credit risks","asset_current","True","l10n_it.account_tag_C_ATT","Fondo rischi su crediti"
"1601","1601","VAT credit","asset_current","False","l10n_it.account_tag_C_ATT","Credito IVA"
"1602","1602","VAT down payment","asset_current","False","l10n_it.account_tag_C_ATT","Acconto IVA"
"1605","1605","VAT receivables","asset_current","False","l10n_it.account_tag_C_ATT","Crediti per IVA"
"1607","1607","Down payment taxes","asset_current","False","l10n_it.account_tag_C_ATT","Imposte sull'acconto"
"1608","1608","Tax credits","asset_current","False","l10n_it.account_tag_C_ATT","Crediti per imposte"
"1609","1609","Receivables for withholdings incurred","asset_current","False","l10n_it.account_tag_C_ATT","Crediti per ritenute subite"
"1610","1610","Receivables for deposits","asset_current","False","l10n_it.account_tag_C_ATT","Crediti per cauzioni"
"1620","1620","Staff down payments","asset_current","False","l10n_it.account_tag_C_ATT","Pagamenti anticipati del personale"
"1630","1630","Social security institution receivables","asset_current","False","l10n_it.account_tag_C_ATT","Crediti dell'ente previdenziale"
"1640","1640","Miscellaneous debtors","asset_current","False","l10n_it.account_tag_C_ATT","Debitori diversi"
"1901","1901","Accrued income","asset_current","False","l10n_it.account_tag_D_ATT","Ratei attivi"
"1902","1902","Prepaid expenses","asset_current","False","l10n_it.account_tag_D_ATT","Risconti attivi"
"2101","2101","Net worth","equity","False","l10n_it.account_tag_A_PASS","Patrimonio netto"
"2102","2102","Operating profit","income","False","l10n_it.account_tag_A_PASS","Utile d'esercizio"
"2103","2103","Operating loss","expense","False","l10n_it.account_tag_A_PASS","Perdita d'esercizio"
"2104","2104","Extra management withdrawals","income","False","l10n_it.account_tag_A_PASS","Prelevamenti extra gestione"
"2105","2105","Holder withholdings incurred","equity","False","l10n_it.account_tag_A_PASS","Ritenute del titolare subite"
"2201","2201","Provision for taxes","liability_current","False","l10n_it.account_tag_B_PASS","Fondo per imposte"
"2204","2204","Liability fund","liability_current","False","l10n_it.account_tag_B_PASS","Fondo responsabilità civile"
"2205","2205","Provision for future expenses","liability_current","False","l10n_it.account_tag_B_PASS","Fondo spese future"
"2211","2211","Scheduled maintenance fund","liability_current","False","l10n_it.account_tag_B_PASS","Fondo manutenzioni programmate"
"2301","2301","TFRL debts","liability_current","False","l10n_it.account_tag_C_PASS","Debiti per TFRL"
"2410","2410","Mortgages payable","liability_current","False","l10n_it.account_tag_D_PASS","Mutui passivi"
"2411","2411","Banks grants","liability_current","False","l10n_it.account_tag_D_PASS","Sovvenzioni delle banche"
"2420","2420","Banks in collection","liability_current","False","l10n_it.account_tag_D_PASS","Banche in raccolta"
"2421","2421","Banks RIBA account in collection","liability_current","False","l10n_it.account_tag_D_PASS","Conto RIBA delle banche in raccolta"
"2422","2422","Banks account bills in collection","liability_current","False","l10n_it.account_tag_D_PASS","Conti correnti bancari in incasso"
"2423","2423","Bank account advances on invoices","liability_current","False","l10n_it.account_tag_D_PASS","Anticipi bancari su fatture"
"2440","2440","Debts to other lenders","liability_current","False","l10n_it.account_tag_D_PASS","Debiti verso altri finanziatori"
"2501","2501","Accounts payable","liability_payable","True","l10n_it.account_tag_D_PASS","Debiti v/fornitori"
"2503","2503","Bills of exchange","liability_current","True","l10n_it.account_tag_D_PASS","Cambiali passive"
"2520","2520","Invoices to be received","liability_current","True","l10n_it.account_tag_D_PASS","Fatture da ricevere"
"2521","2521","Debts to be settled","liability_current","True","l10n_it.account_tag_D_PASS","Debiti da liquidare"
"2530","2530","Customers down payments","liability_current","True","l10n_it.account_tag_D_PASS","Acconti dei clienti"
"2601","2601","VAT debt","liability_current","False","l10n_it.account_tag_D_PASS","Debito IVA"
"2602","2602","Payables for withholding taxes to be paid","liability_current","False","l10n_it.account_tag_D_PASS","Debiti per ritenute da versare"
"2605","2605","Treasury VAT","liability_current","False","l10n_it.account_tag_D_PASS","Tesoro IVA"
"2606","2606","Taxes payable","liability_current","False","l10n_it.account_tag_D_PASS","Debiti per imposte"
"2607","2607","Funds with VAT Split Payment","liability_current","False","l10n_it.account_tag_D_PASS","Erario c/IVA Split Payment"
"2608","2608","VAT Split Payment","liability_current","False","l10n_it.account_tag_D_PASS","IVA c/Split Payment"
"2619","2619","Payables for deposits","liability_current","False","l10n_it.account_tag_D_PASS","Debiti per cauzioni"
"2620","2620","Staff salaries","liability_current","False","l10n_it.account_tag_D_PASS","Stipendi del personale"
"2621","2621","Staff liquidations","liability_current","False","l10n_it.account_tag_D_PASS","Liquidazioni del personale"
"2622","2622","Customers disposal","liability_current","False","l10n_it.account_tag_D_PASS","Smaltimento dei clienti"
"2630","2630","Social security debts","liability_current","False","l10n_it.account_tag_D_PASS","Debiti previdenziali"
"2640","2640","Miscellaneous creditors","liability_current","False","l10n_it.account_tag_D_PASS","Creditori diversi"
"2701","2701","Accrued expenses","liability_current","False","l10n_it.account_tag_E_PASS","Ratei passivi"
"2702","2702","Deferred income","liability_current","False","l10n_it.account_tag_E_PASS","Risconti passivi"
"2801","2801","Opening balance","liability_current","False","","Bilancio di apertura"
"2802","2802","Closing balance sheet","liability_current","False","","Bilancio di chiusura"
"2810","2810","VAT liquidations","liability_current","False","","Liquidazioni IVA"
"2811","2811","Social security institutions","liability_current","False","","Istituti previdenziali"
"2901","2901","Third-party assets","off_balance","False","l10n_it.account_tag_BENI","Beni di terzi"
"2902","2902","Depositors of goods","off_balance","False","l10n_it.account_tag_BENI","Depositanti beni"
"2911","2911","Goods to be received","off_balance","False","l10n_it.account_tag_IMPEGNI","Merci da ricevere"
"2912","2912","Supplier commitments","off_balance","False","l10n_it.account_tag_IMPEGNI","Impegni dei fornitori"
"2913","2913","Commitments for leased assets","off_balance","False","l10n_it.account_tag_IMPEGNI","Impegni per beni in leasing"
"2914","2914","Lease creditors","off_balance","False","l10n_it.account_tag_IMPEGNI","Creditori di leasing"
"2916","2916","Customer engagements","off_balance","False","l10n_it.account_tag_IMPEGNI","Impegni dei clienti"
"2917","2917","Goods to be delivered","off_balance","False","l10n_it.account_tag_IMPEGNI","Merci da consegnare"
"2921","2921","Risks for discounted effects","off_balance","False","l10n_it.account_tag_RISCHI","Rischi per effetti scontati"
"2922","2922","Banks discounted effects","off_balance","False","l10n_it.account_tag_RISCHI","Banche effetti scontati"
"2926","2926","Risks for surety bonds","off_balance","False","l10n_it.account_tag_RISCHI","Rischi per fideiussioni"
"2927","2927","Creditors for surety bonds","off_balance","False","l10n_it.account_tag_IMPEGNI","Creditori per fideiussioni"
"2931","2931","Risks for endorsements","off_balance","False","l10n_it.account_tag_RISCHI","Rischi per avalli"
"2932","2932","Creditors for endorsements","off_balance","False","l10n_it.account_tag_IMPEGNI","Creditori per avalli"
"3101","3101","Goods w/sales","income","False","l10n_it.account_tag_A_PL","Merci c/vendite"
"3103","3103","Reimbursement of sales expenses","income","False","l10n_it.account_tag_A_PL","Rimborsi spese di vendita"
"3110","3110","Returns on sales","income","False","l10n_it.account_tag_A_PL","Resi su vendite"
"3111","3111","Rebates and rebates payable","income","False","l10n_it.account_tag_A_PL","Ribassi e abbuoni passivi"
"3112","3112","Premiums on sales","income","False","l10n_it.account_tag_A_PL","Premi su vendite"
"3201","3201","Premiums on salesRent income","income","False","l10n_it.account_tag_A_PL","Fitti attivi"
"3202","3202","Miscellaneous income","income","False","l10n_it.account_tag_A_PL","Proventi vari"
"3210","3210","Active rounding","income","False","l10n_it.account_tag_A_PL","Arrotondamenti attivi"
"3220","3220","Miscellaneous ordinary capital gains","income","False","l10n_it.account_tag_A_PL","Plusvalenze ordinarie diverse"
"3230","3230","Miscellaneous ordinary contingent assets","income","False","l10n_it.account_tag_A_PL","Sopravvenienze attive ordinarie diverse"
"3240","3240","Sundry ordinary non-existent assets","income","False","l10n_it.account_tag_A_PL","Insussistenze attive ordinarie diverse"
"4101","4101","Goods purchased","expense","False","l10n_it.account_tag_B_PL","Merce acquistata"
"4102","4102","Consumables purchased","expense","False","l10n_it.account_tag_B_PL","Materiali di consumo acquistati"
"4105","4105","Goods contributions","expense","False","l10n_it.account_tag_B_PL","Contributi per le merci"
"4110","4110","Returns on purchases","expense","False","l10n_it.account_tag_B_PL","Resi su acquisti"
"4111","4111","Active rebates and discounts","expense","False","l10n_it.account_tag_B_PL","Ribassi e abbuoni attivi"
"4112","4112","Rewards on purchases","expense","False","l10n_it.account_tag_B_PL","Premi su acquisti"
"4121","4121","Goods initial existences","expense","False","l10n_it.account_tag_B_PL","Esistenze iniziali dei beni"
"4122","4122","Existing consumables","expense","False","l10n_it.account_tag_B_PL","Materiali di consumo esistenti"
"4131","4131","Goods closing inventory","expense","False","l10n_it.account_tag_B_PL","Inventario di chiusura merci"
"4132","4132","Consumables closing inventories","expense","False","l10n_it.account_tag_B_PL","Rimanenze finali di materiali di consumo"
"4201","4201","Transportation costs","expense","False","l10n_it.account_tag_B_PL","Costi di trasporto"
"4202","4202","Energy costs","expense","False","l10n_it.account_tag_B_PL","Costi per energia"
"4203","4203","Advertising costs","expense","False","l10n_it.account_tag_B_PL","Costi di pubblicità"
"4204","4204","Consulting costs","expense","False","l10n_it.account_tag_B_PL","Costi di consulenze"
"4205","4205","Postal costs","expense","False","l10n_it.account_tag_B_PL","Costi postali"
"4206","4206","Telephone costs","expense","False","l10n_it.account_tag_B_PL","Costi telefonici"
"4207","4207","Insurance costs","expense","False","l10n_it.account_tag_B_PL","Costi di assicurazione"
"4208","4208","Supervisory costs","expense","False","l10n_it.account_tag_B_PL","Costi di vigilanza"
"4209","4209","Costs for the premises","expense","False","l10n_it.account_tag_B_PL","Costi per i locali"
"4210","4210","Motor vehicle operating costs","expense","False","l10n_it.account_tag_B_PL","Costi di esercizio automezzi"
"4211","4211","Maintenance and repair costs","expense","False","l10n_it.account_tag_B_PL","Costi di manutenzione e riparazione"
"4212","4212","Commissions payable","expense","False","l10n_it.account_tag_B_PL","Provvigioni passive"
"4213","4213","Collection fees","expense","False","l10n_it.account_tag_B_PL","Spese di incasso"
"4301","4301","Rents payable","expense","False","l10n_it.account_tag_B_PL","Fitti passivi"
"4302","4302","Leasing fees","expense","False","l10n_it.account_tag_B_PL","Canoni di leasing"
"4401","4401","Wages and salaries","expense","False","l10n_it.account_tag_B_PL","Salari e stipendi"
"4402","4402","Social charges","expense","False","l10n_it.account_tag_B_PL","Oneri sociali"
"4403","4403","TFRL","expense","False","l10n_it.account_tag_B_PL","TFRL"
"4404","4404","Other personnel costs","expense","False","l10n_it.account_tag_B_PL","Altri costi per il personale"
"4501","4501","Amortization of start-up costs","expense","False","l10n_it.account_tag_B_PL","Ammortamento costi di impianto"
"4506","4506","Software amortization","expense","False","l10n_it.account_tag_B_PL","Ammortamento software"
"4508","4508","Goodwill amortization","expense","False","l10n_it.account_tag_B_PL","Ammortamento avviamento"
"4601","4601","Building depreciation","expense","False","l10n_it.account_tag_B_PL","Ammortamento fabbricati"
"4602","4602","Depreciation of plant and machinery","expense","False","l10n_it.account_tag_B_PL","Ammortamento impianti e macchinari"
"4604","4604","Commercial equipment depreciation","expense","False","l10n_it.account_tag_B_PL","Ammortamento attrezzature commerciali"
"4605","4605","Depreciation of office machines","expense","False","l10n_it.account_tag_B_PL","Ammortamento macchine d'ufficio"
"4606","4606","Furniture depreciation","expense","False","l10n_it.account_tag_B_PL","Ammortamento arredamento"
"4607","4607","Vehicle depreciation","expense","False","l10n_it.account_tag_B_PL","Ammortamento automezzi"
"4608","4608","Durable packaging depreciation","expense","False","l10n_it.account_tag_B_PL","Ammortamento imballaggi durevoli"
"4701","4701","Durable packaging depreciation","expense","False","l10n_it.account_tag_B_PL","Ammortamento imballaggi durevoli"
"4702","4702","Depreciation of tangible fixed assets","expense","False","l10n_it.account_tag_B_PL","Svalutazioni immobilizzazioni materiali"
"4706","4706","Impairment of receivables","expense","False","l10n_it.account_tag_B_PL","Svalutazione crediti"
"4814","4814","Provision for liability","expense","False","l10n_it.account_tag_B_PL","Accantonamento per responsabilità civile"
"4821","4821","Provision for future expenses","expense","False","l10n_it.account_tag_B_PL","Fondo spese future"
"4823","4823","Provision for planned maintenance","expense","False","l10n_it.account_tag_B_PL","Accantonamento per manutenzioni programmate"
"4901","4901","Miscellaneous tax charges","expense","False","l10n_it.account_tag_B_PL","Oneri fiscali diversi"
"4903","4903","Miscellaneous charges","expense","False","l10n_it.account_tag_B_PL","Oneri vari"
"4905","4905","Losses on receivables","expense","False","l10n_it.account_tag_B_PL","Perdite su crediti"
"4910","4910","Passive rounding","expense","False","l10n_it.account_tag_B_PL","Arrotondamenti passivi"
"4920","4920","Miscellaneous ordinary capital losses","expense","False","l10n_it.account_tag_B_PL","Minusvalenze ordinarie diverse"
"4930","4930","Miscellaneous ordinary contingent liabilities","expense","False","l10n_it.account_tag_B_PL","Sopravvenienze passive ordinarie diverse"
"4940","4940","Miscellaneous ordinary nonexistent liabilities","expense","False","l10n_it.account_tag_B_PL","Insussistenze passive ordinarie diverse"
"5110","5110","Interest income from customers","income_other","False","l10n_it.account_tag_C_PL","Interessi attivi v/clienti"
"5115","5115","Bank interest income","income_other","False","l10n_it.account_tag_C_PL","Interessi attivi bancari"
"5116","5116","Postal interest income","income_other","False","l10n_it.account_tag_C_PL","Interessi attivi postali"
"5140","5140","Miscellaneous financial income","income_other","False","l10n_it.account_tag_C_PL","Proventi finanziari diversi"
"5201","5201","Interest expense from/to suppliers","expense","False","l10n_it.account_tag_C_PL","Interessi passivi v/fornitori"
"5202","5202","Bank interest expense","expense","False","l10n_it.account_tag_C_PL","Interessi passivi bancari"
"5203","5203","Bank overdrafts","expense","False","l10n_it.account_tag_C_PL","Sconti passivi bancari"
"5210","5210","Interest expense on mortgages","expense","False","l10n_it.account_tag_C_PL","Interessi passivi su mutui"
"5240","5240","Miscellaneous financial charges","expense","False","l10n_it.account_tag_C_PL","Oneri finanziari diversi"
"8101","8101","Taxes for the year","expense","False","l10n_it.account_tag_E_PL","Imposte dell'esercizio"
"9101","9101","Economic performance account","expense","False","","Conto di risultato economico"
"9102","9102","État patrimonial","expense","False","","Stato patrimoniale"

```

## File: data\template\account.fiscal.position-it.csv

```csv
"id","name",sequence,"auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@it","note","note@it"
"it","Domestic","1","1","1","base.it","","","","Nazionale","",""
"extra","Import/Export","4","1","","","","22v","00ex","Importazione/Esportazione","",""
"","","","","","","","10v","00ex","","",""
"","","","","","",,"5v","00ex","","",""
"","","","","","","","4v","00ex","","",""
"","","","","","","","00v","00ex","","",""
"intra_private",EU B2C,"2","1","","","base.europe","","","EU B2C","",""
"intra","Intra-Community",3,"1","1","","base.europe","22v","00eu","Intra-Comunitario","Invoice issued in accordance with Article 17, Paragraph 2 of Presidential Decree No. 633 dated October 26, 1972, the application of VAT is the responsibility of the recipient.","Fattura emessa ai sensi dell’art. 17, comma 2 del DPR 26/10/1972 n. 633, l’applicazione dell’IVA è a carico del destinatario."
"","","","","","","","10v","00eu","","",""
"","","","","","",,"5v","00eu","","",""
"","","","","","","","4v","00eu","","",""
"","","","","","","","00v","00eu","","",""
"","","","","","","","22am","22rcm","","",""
"","","","","","","","10am","10rcm","","",""
"","","","","","","","5am","5rcm","","",""
"","","","","","","","4am","4rcm","","",""
"","","","","","","","00am","00rcm","","",""
"","","","","","","","22as","22rcs","","",""
"","","","","","","","10as","10rcs","","",""
"","","","","","","","5as","5rcs","","",""
"","","","","","","","4as","4rcs","","",""
"","","","","","","","00as","00rcs","","",""
"split_payment_fiscal_position","Split Payment","5","0","0",,,"22v","22vsp_group","Scissione dei Pagamenti","Operations subject to split payment – the seller does not collect the VAT pursuant to ex art.17-ter of the D.P.R. 633/1972, the buyer is obliged to pay the Revenue Agency.","Operazione soggetta a split payment – il cedente non incassa l’Iva ai sensi dell’ex art.17-ter del D.P.R. 633/1972, l’acquirente è obbligato al versamento all’Agenzia delle Entrate."
"","","","","","","","10v","10vsp_group","","",""
"","","","","","","","5v","5vsp_group","","",""
"","","","","","","","4v","4vsp_group","","",""
"construction_fiscal_position","Construction Subcontractors - RC","9","0","0",,,"22as","22rc_n63","Subappalto Edilizia - IC","Operation subject to Reverse Charge and exempt from stamp duty – the seller does not collect the VAT pursuant to ex art.17, comma 6, letter a, DPR 633/1972","Operazione soggetta a reverse charge ed esente da imposta da bollo – il cedente non incassa l’Iva ai sensi dell’ex art. 17, comma 6, lettera a, DPR 633/1972."
"","","","","","","","10as","10rc_n63","","",""
"","","","","","","","5as","5rc_n63","","",""
"","","","","","","","4as","4rc_n63","","",""
"","","","","","","","00v","0rc_n63","","",""

```

## File: data\template\account.tax-it.csv

```csv
"id","description","invoice_label","name","sequence","amount","amount_type","type_tax_use","tax_group_id","active","tax_scope","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@it","children_tax_ids","l10n_it_exempt_reason","l10n_it_law_reference","description@it","price_include_override"
"22v","","22%","22%","30","22.0","percent","sale","tax_group_iva_22","","","base","invoice","+02","","","22%","","","","",""
"","","","","","","","","","","","tax","invoice","+4v","2601","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","2601","","","","","","",""
"10v","","10%","10%","40","10.0","percent","sale","tax_group_iva_10","","","base","invoice","+02","","","10%","","","","",""
"","","","","","","","","","","","tax","invoice","+4v","2601","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","2601","","","","","","",""
"5v","","5%","5%","50","5.0","percent","sale","tax_group_iva_5","False","","base","invoice","+02","","","","","","","",""
"","","","","","","","","","","","tax","invoice","+4v","2601","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","2601","","","","","","",""
"4v","","4%","4%","60","4.0","percent","sale","tax_group_iva_4","False","","base","invoice","+02","","","","","","","",""
"","","","","","","","","","","","tax","invoice","+4v","2601","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","2601","","","","","","",""
"00v","","0%","0%","70","0.0","percent","sale","tax_group_fuori","","","base","invoice","","","","","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","","","","","","","","",""
"","","","","","","","","","","","base","refund","","","","","","","","",""
"","","","","","","","","","","","tax","refund","","","","","","","","",""
"22am","","22%","22% G","80","22.0","percent","purchase","tax_group_iva_22","","consu","base","invoice","+03","","","22% M","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"10am","","10%","10% G","90","10.0","percent","purchase","tax_group_iva_10","","consu","base","invoice","+03","","","10% M","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"5am","","5%","5% G","100","5.0","percent","purchase","tax_group_iva_5","False","consu","base","invoice","+03","","","5% M","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"4am","","4%","4% G","110","4.0","percent","purchase","tax_group_iva_4","False","consu","base","invoice","+03","","","4% M","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"00am","","0%","0% G","120","0.0","percent","purchase","tax_group_fuori","","consu","base","invoice","+03","","","0% M","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","","","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","","","","","","","","",""
"22as","","22%","22% S","130","22.0","percent","purchase","tax_group_iva_22","","service","base","invoice","+03","","","22% S","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"10as","","10%","10% S","140","10.0","percent","purchase","tax_group_iva_10","","service","base","invoice","+03","","","10% S","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"5as","","5%","5% S","150","5.0","percent","purchase","tax_group_iva_5","False","service","base","invoice","+03","","","5% S","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"4as","","4%","4% S","160","4.0","percent","purchase","tax_group_iva_4","False","service","base","invoice","+03","","","4% S","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"00as","","0%","0% S","170","0.0","percent","purchase","tax_group_fuori","","service","base","invoice","+03","","","0% S","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","","","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","","","","","","","","",""
"00eu","","0%","0% EU","180","0.0","percent","sale","tax_group_fuori","","","base","invoice","+02","","","0% EU","","N3.2","Art. 41, DL 331/93","",""
"","","","","","","","","","","","tax","invoice","","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","","","","","","","","",""
"00ex","","0%","0% EX","185","0.0","percent","sale","tax_group_fuori","","","base","invoice","","","","0% EX","","N2.2","Fuori dall'Unione Europea","",""
"","","","","","","","","","","","tax","invoice","","","","","","","","",""
"","","","","","","","","","","","base","refund","","","","","","","","",""
"","","","","","","","","","","","tax","refund","","","","","","","","",""
"00art15v","","0%","0% Art.15","190","0.0","percent","sale","tax_group_imp_esc_art_15","","","base","invoice","+02","","","0% Art.15","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","","","","","","","","",""
"00art15a","","00art15a","0% Art.15","200","0.0","percent","purchase","tax_group_imp_esc_art_15","","","base","invoice","+03","","","0% Art.15","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","","","","","","","","",""
"","","","","","","","","","","","base","refund","-03","","","","","","","",""
"","","","","","","","","","","","tax","refund","","","","","","","","",""
"22rcs","","22%","22% S RC","210","22.0","percent","purchase","tax_group_iva_22","","service","base","invoice","+03||+vj3","","","22% S IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rcs","","10%","10% S RC","220","10.0","percent","purchase","tax_group_iva_10","","service","base","invoice","+03||+vj3","","","10% S IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rcs","","5%","5% S RC","230","5.0","percent","purchase","tax_group_iva_5","False","service","base","invoice","+03||+vj3","","","5% S IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rcs","","4%","4% S RC","240","4.0","percent","purchase","tax_group_iva_4","False","service","base","invoice","+03||+vj3","","","4% S IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"00rcs","","0%","0% S RC","250","0.0","percent","purchase","tax_group_fuori","","service","base","invoice","+03||+vj3","","","0% S CI","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rcm","","22%","22% G RC","260","22.0","percent","purchase","tax_group_iva_22","","consu","base","invoice","+03||+vj9","","","22% M IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj9","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rcm","","10%","10% G RC","270","10.0","percent","purchase","tax_group_iva_10","","consu","base","invoice","+03||+vj9","","","10% M IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj9","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rcm","","5%","5% G RC","280","5.0","percent","purchase","tax_group_iva_5","False","consu","base","invoice","+03||+vj9","","","5% M IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj9","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rcm","","4%","4% G RC","290","4.0","percent","purchase","tax_group_iva_4","False","consu","base","invoice","+03||+vj9","","","4% M IC","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj9","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"00rcm","","0%","0% G RC","300","0.0","percent","purchase","tax_group_fuori","","consu","base","invoice","+03||+vj9","","","0% M CI","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj9","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rcd","22% G Deposit","22%","22% G D","310","22.0","percent","purchase","tax_group_iva_22","","consu","base","invoice","+03||+vj3","","","22% M Deposito","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rcd","10% G Deposit","10%","10% G D","320","10.0","percent","purchase","tax_group_iva_10","","consu","base","invoice","+03||+vj3","","","10% M Deposito","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rcd","5% G Deposit","5%","5% G D","330","5.0","percent","purchase","tax_group_iva_5","False","consu","base","invoice","+03||+vj3","","","5% M Deposito","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rcd","4% G Deposit","4%","4% G D","340","4.0","percent","purchase","tax_group_iva_4","False","consu","base","invoice","+03||+vj3","","","4% M Deposito","","","","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"00rcd","0% G Deposit","0%","0% G D","350","0.0","percent","purchase","tax_group_fuori","","service","base","invoice","+03||+vj3","","","0% M Deposito","","N1","Art. 15 DPR 633/1972","",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj3","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22vsp","22% Split Payment pos.","22%","22% SP pos.","360","22.0","percent","none","tax_group_iva_22","","","base","invoice","+02||+ve38","","","","","","","22% Scissione dei Pagamenti pos.",""
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","-02||-ve38","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"22vsp_storno","22% Split Payment neg.","22%","22% SP neg.","360","-22.0","percent","none","tax_group_split_payment","","","base","invoice","","","","","","","","22% Scissione dei Pagamenti neg.","tax_excluded"
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"22vsp_group","","22%","22% SP","360","","group","sale","tax_group_split_payment","","","","","","","","","22vsp,22vsp_storno","","","",""
"10vsp","10% Split Payment pos.","10%","10% SP pos.","360","10.0","percent","none","tax_group_iva_10","","","base","invoice","+02||+ve38","","","","","","","10% Scissione dei Pagamenti pos.",""
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","-02||-ve38","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"10vsp_storno","10% Split Payment neg.","10%","10% SP neg.","360","-10.0","percent","none","tax_group_split_payment","","","base","invoice","","","","","","","","10% Scissione dei Pagamenti neg.","tax_excluded"
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"10vsp_group","","10%","10% SP","360","","group","sale","tax_group_split_payment","","","","","","","","","10vsp,10vsp_storno","","","",""
"5vsp","5% Split Payment pos.","5%","5% SP pos.","360","5.0","percent","none","tax_group_iva_5","","","base","invoice","+02||+ve38","","","","","","","5% Scissione dei Pagamenti pos.",""
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","-02||-ve38","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"5vsp_storno","5% Split Payment neg.","5%","5% SP neg.","360","-5.0","percent","none","tax_group_split_payment","","","base","invoice","","","","","","","","5% Scissione dei Pagamenti neg.","tax_excluded"
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"5vsp_group","","5%","5% SP","360","","group","sale","tax_group_split_payment","","","","","","","","","5vsp,5vsp_storno","","","",""
"4vsp","4% Split Payment pos.","4%","4% SP pos.","360","4.0","percent","none","tax_group_iva_4","","","base","invoice","+02||+ve38","","","","","","","4% Scissione dei Pagamenti pos.",""
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","-02||-ve38","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"4vsp_storno","4% Split Payment neg.","4%","4% SP neg.","360","-4.0","percent","none","tax_group_split_payment","","","base","invoice","","","","","","","","4% Scissione dei Pagamenti neg.","tax_excluded"
"","","","","","","","","","","","tax","invoice","","2608","","","","","","",""
"","","","","","","","","","","","base","refund","","","","","","","","",""
"","","","","","","","","","","","tax","refund","","2608","","","","","","",""
"4vsp_group","","4%","4% SP","460","","group","sale","tax_group_split_payment","","","","","","","","","4vsp,4vsp_storno","","","",""
"0rc_n61","Scrap","0%","0% RC N6.1","500","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.1","","N6.1","art.74, comma 7-8, D.P.R. 633/1972","Materiali di Scarto",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n62","Gold","0%","0% RC N6.2","510","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.2","","N6.2","art.17, comma 5, D.P.R. 633/1972","Oro",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n62b","Gold Investments","0%","0% RC N6.2b","520","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.2b","","N6.2","art.17, comma 5, D.P.R. 633/1972","Investimenti in Oro",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n63","Construction subcontractors","0%","0% RC N6.3","530","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.3","","N6.3","art.17, comma 6, lett. a, D.P.R. 633/1972","Subappalto Edilizia",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n64","Investments in construction","0%","0% RC N6.4","540","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.4","","N6.4","art.17, comma 6, lett. a-bis, D.P.R. 633/1972","Investimenti in Edilizia",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n65","Cellphones","0%","0% RC N6.5","550","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.5","","N6.5","art.17, comma 6, lett. b, D.P.R. 633/1972","Cellulari",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n66","Electronics","0%","0% RC N6.6","560","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.6","","N6.6","art.17, comma 6, lett. c, D.P.R. 633/1972","Elettronica",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n67","Construction services","0%","0% RC N6.7","570","0.0","percent","sale","tax_group_fuori","True","","base","invoice","+02","","","0% IC N6.7","","N6.7","art 17, comma 6, lett. a-ter, D.P.R. 633/1972","Servizi edilizia",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"0rc_n68","Energy","0%","0% RC N6.8","580","0.0","percent","sale","tax_group_fuori","False","","base","invoice","+02","","","0% IC N6.8","","N6.8","art.17, comma 6, lett. d-bis, d-ter, d-quater, D.P.R. 633/1972","Energia",""
"","","","","","","","","","","","tax","invoice","+4v","","","","","","","",""
"","","","","","","","","","","","base","refund","-02","","","","","","","",""
"","","","","","","","","","","","tax","refund","-4v","","","","","","","",""
"4rc_n61","Scrap","4%","4% RC N6.1","590","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj6","","","4% IC N6.1","","","","Materiali di Scarto",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj6","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n62","Gold","4%","4% RC N6.2","600","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj7","","","4% IC N6.2","","","","Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj7","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n62b","Investments in gold","4%","4% RC N6.2b","610","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj8","","","4% IC N6.2b","","","","Investimenti in Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj8","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n63","Construction subcontractors","4%","4% RC N6.3","620","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj12","","","4% IC N6.3","","","","Subappalto Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj12","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n64","Investments in construction","4%","4% RC N6.4","630","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj13","","","4% IC N6.4","","","","Investimenti in Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj13","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n65","Cellphones","4%","4% RC N6.5","640","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj14","","","4% IC N6.5","","","","Cellulari",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj14","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n66","Electronics","4%","4% RC N6.6","650","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj15","","","4% IC N6.6","","","","Elettronica",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj15","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n67","Construction services","4%","4% RC N6.7","660","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj16","","","4% IC N6.7","","","","Servizi Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj16","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"4rc_n68","Energy","4%","4% RC N6.8","670","4.0","percent","purchase","tax_group_iva_4","False","","base","invoice","+03||+vj17","","","4% IC N6.8","","","","Energia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj17","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n61","Scrap","5%","5% RC N6.1","680","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj6","","","5% IC N6.1","","","","Materiali di Scarto",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj6","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n62","Gold","5%","5% RC N6.2","690","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj7","","","5% IC N6.2","","","","Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj7","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n62b","Investments in gold","5%","5% RC N6.2b","700","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj8","","","5% IC N6.2b","","","","Investimenti in Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj8","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n63","Construction subcontractors","5%","5% RC N6.3","710","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj12","","","5% IC N6.3","","","","Subappalti Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj12","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n64","Investments in construction","5%","5% RC N6.4","720","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj13","","","5% IC N6.4","","","","Investimenti in Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj13","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n65","Cellphones","5%","5% RC N6.5","730","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj14","","","5% IC N6.5","","","","Cellulari",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj14","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n66","Electronics","5%","5% RC N6.6","740","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj15","","","5% IC N6.6","","","","Elettronica",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj15","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n67","Construction services","5%","5% RC N6.7","750","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj16","","","5% IC N6.7","","","","Servizi Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj16","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"5rc_n68","Energy","5%","5% RC N6.8","760","5.0","percent","purchase","tax_group_iva_5","False","","base","invoice","+03||+vj17","","","5% IC N6.8","","","","Energia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj17","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n61","Scrap","10%","10% RC N6.1","770","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj6","","","10% IC N6.1","","","","Materiali di Scarto",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj6","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n62","Gold","10%","10% RC N6.2","780","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj7","","","10% IC N6.2","","","","Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj7","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n62b","Investments in gold","10%","10% RC N6.2b","790","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj8","","","10% IC N6.2b","","","","Investimenti in Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj8","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n63","Construction subcontractors","10%","10% RC N6.3","800","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj12","","","10% IC N6.3","","","","Subappalti Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj12","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n64","Investments in construction","10%","10% RC N6.4","810","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj13","","","10% IC N6.4","","","","Investimenti in Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj13","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n65","Cellphones","10%","10% RC N6.5","820","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj14","","","10% IC N6.5","","","","Cellulari",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj14","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n66","Electronics","10%","10% RC N6.6","830","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj15","","","10% IC N6.6","","","","Elettronica",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj15","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n67","Construction services","10%","10% RC N6.7","840","10.0","percent","purchase","tax_group_iva_10","True","","base","invoice","+03||+vj16","","","10% IC N6.7","","","","Servizi Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj16","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"10rc_n68","Energy","10%","10% RC N6.8","850","10.0","percent","purchase","tax_group_iva_10","False","","base","invoice","+03||+vj17","","","10% IC N6.8","","","","Energia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj17","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n61","Scrap","22%","22% RC N6.1","860","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj6","","","22% IC N6.1","","","","Materiali di Scarto",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj6","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n62","Gold","22%","22% RC N6.2","870","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj7","","","22% IC N6.2","","","","Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj7","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n62b","Investments in gold","22%","22% RC N6.2b","880","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj8","","","22% IC N6.2b","","","","Investimenti in Oro",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj8","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n63","Construction subcontractors","22%","22% RC N6.3","890","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj12","","","22% IC N6.3","","","","Subappalti Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj12","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n64","Investments in construction","22%","22% RC N6.4","900","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj13","","","22% IC N6.4","","","","Investimenti in Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj13","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n65","Cellphones","22%","22% RC N6.5","910","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj14","","","22% IC N6.5","","","","Cellulari",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj14","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n66","Electronics","22%","22% RC N6.6","920","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj15","","","22% IC N6.6","","","","Elettronica",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj15","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n67","Construction services","22%","22% RC N6.7","930","22.0","percent","purchase","tax_group_iva_22","True","","base","invoice","+03||+vj16","","","22% IC N6.7","","","","Servizi Edilizia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj16","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""
"22rc_n68","Energy","22%","22% RC N6.8","940","22.0","percent","purchase","tax_group_iva_22","False","","base","invoice","+03||+vj17","","","22% IC N6.8","","","","Energia",""
"","","","","","","","","","","","tax","invoice","+5v","1601","","","","","","",""
"","","","","","","","","","","","tax","invoice","-4v","2601","-100","","","","","",""
"","","","","","","","","","","","base","refund","-03||-vj17","","","","","","","",""
"","","","","","","","","","","","tax","refund","-5v","1601","","","","","","",""
"","","","","","","","","","","","tax","refund","+4v","2601","-100","","","","","",""

```

## File: data\template\account.tax.group-it.csv

```csv
"id","name","country_id","preceding_subtotal","tax_receivable_account_id","tax_payable_account_id","name@it","pos_receipt_label"
"tax_group_iva_2","2% VAT","base.it","Imponibile","2605","2605","IVA 2%",""
"tax_group_iva_4","4% VAT","base.it","Imponibile","2605","2605","IVA 4%","3"
"tax_group_iva_5","5% VAT","base.it","Imponibile","2605","2605","IVA 5%","4"
"tax_group_iva_10","10% VAT","base.it","Imponibile","2605","2605","IVA 10%","2"
"tax_group_iva_12","12% VAT","base.it","Imponibile","2605","2605","IVA 12%",""
"tax_group_iva_21","21% VAT","base.it","Imponibile","2605","2605","IVA 21%",""
"tax_group_iva_20","20% VAT","base.it","Imponibile","2605","2605","IVA 20%",""
"tax_group_iva_22","22% VAT","base.it","Imponibile","2605","2605","IVA 22%","1"
"tax_group_imp_esc_art_15","VAT Excluded Art.15","base.it","Imponibile","2605","2605","Imponibile Escluso Art.15",""
"tax_group_fuori","VAT-free","base.it","Imponibile","2605","2605","Fuori Campo IVA","10"
"tax_group_split_payment","Split Payment","base.it","Scissione dei Pagamenti Esclusa","2607","2607","Scissione dei Pagamenti",""

```

## File: migrations\0.6\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'it')], order="parent_path"):
        env['account.chart.template'].try_loading('it', company)

```

## File: migrations\0.7\post-map_carryover_to_new_report.py

```python
from odoo import api, SUPERUSER_ID
from odoo.tools import sql

def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    vat_report_id = env.ref('l10n_it.tax_report_vat').id
    monthly_vat_report_id = env.ref('l10n_it.tax_monthly_report_vat').id

    external_value_cols = [
        col
        for col in sql.table_columns(env.cr, 'account_report_external_value')
        if col not in ['id', 'carryover_origin_report_line_id', 'target_report_expression_id']
    ]

    cr.execute(f"""
        SELECT report.id AS report_id,
               expression.id AS expression_id,
               report_line.code,
               external_value.carryover_origin_report_line_id,
               {', '.join(f'external_value.{col}' for col in external_value_cols)}
          FROM account_report AS report
          JOIN account_report_line AS report_line ON report.id = report_line.report_id
          JOIN account_report_expression AS expression ON report_line.id = expression.report_line_id
                                                      AND expression.engine = 'external'
     LEFT JOIN account_report_external_value AS external_value ON expression.id = external_value.target_report_expression_id
         WHERE (
                   report.id = %s
                   AND external_value.company_id IS NOT NULL
               )
            OR report.id = %s
      ORDER BY expression_id;
    """, (vat_report_id, monthly_vat_report_id))
    report_info = cr.fetchall()

    code2expression_id = {
        report_line_code: expression_id
        for report_id, expression_id, report_line_code, *_ in report_info
        if report_id == monthly_vat_report_id
    }

    cr.execute("""
        SELECT DISTINCT old_report_line.id AS old_origin,
                        new_report_line.id AS new_origin
                   FROM account_report_external_value external_value
                   JOIN account_report_line AS old_report_line ON old_report_line.id = external_value.carryover_origin_report_line_id
                                                              AND old_report_line.report_id = %s
                   JOIN account_report_line AS new_report_line ON new_report_line.code = old_report_line.code
                                                              AND new_report_line.report_id = %s;
    """, (vat_report_id, monthly_vat_report_id))
    carryover_origin_info = cr.fetchall()
    old2new_origin = {old_origin: new_origin for old_origin, new_origin in carryover_origin_info}
    data_to_insert = [
        (code2expression_id[report_line_code], old2new_origin[carryover_origin_report_line_id], *other_external_vals)
        for report_id, _, report_line_code, carryover_origin_report_line_id, *other_external_vals in report_info
        if report_id == vat_report_id
    ]

    insert_query = f"""
        INSERT INTO account_report_external_value (
                        target_report_expression_id,
                        carryover_origin_report_line_id,
                        {', '.join(col for col in external_value_cols)}
                    )
             VALUES (%s, %s, {', '.join('%s' for _ in external_value_cols)})
        ON CONFLICT DO NOTHING
    """

    if data_to_insert:
        cr.executemany(insert_query, data_to_insert)

    # Archive the old report
    cr.execute("""
        UPDATE account_report
           SET active = FALSE
         WHERE id = %s
    """, (vat_report_id,))

```

## File: migrations\15.0.0.3\post-migrate.py

```python
# -*- coding: utf-8 -*-

def migrate(cr, version):

    cr.execute("""
        INSERT INTO account_account_account_tag
        SELECT DISTINCT account.id, template_tag.account_account_tag_id
        FROM account_account_template AS template
        JOIN account_account AS account
            ON account.code LIKE CONCAT(template.code, '%')
        JOIN account_account_template_account_tag AS template_tag
            ON template.id = template_tag.account_account_template_id
        JOIN res_company ON res_company.id = account.company_id
        JOIN res_country ON res_country.id = res_company.account_fiscal_country_id
            AND res_country.code = 'IT'
        ON CONFLICT DO NOTHING
    """)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _message_set_main_attachment_id(self, attachments, force=False, filter_xml=True):
        if self.message_main_attachment_id.mimetype == "application/pkcs7-mime":
            force = True
        super()._message_set_main_attachment_id(attachments, force, filter_xml)

```

## File: models\account_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountReportExpression(models.AbstractModel):
    _inherit = "account.report.expression"

    def _get_carryover_target_expression(self, options):
        if self.report_line_id.code == 'VP14b' and fields.Date.from_string(options['date']['date_to']).month == 12:
            # For this line, if we are between two years, we want to carry over to vp9 instead of the line set in the XML file (vp8)
            if self.report_line_id == self.env.ref('l10n_it.tax_monthly_report_line_vp14b', raise_if_not_found=False):
                return self.env.ref('l10n_it.tax_monthly_report_line_vp9_applied_carryover')
            return self.env.ref('l10n_it.tax_report_line_vp9_applied_carryover')

        return super()._get_carryover_target_expression(options)

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError, UserError

class AccountTax(models.Model):
    _inherit = "account.tax"

    l10n_it_exempt_reason = fields.Selection(
        selection=[
            ("N1", "[N1] Escluse ex art. 15"),
            ("N2", "[N2] Non soggette"),
            ("N2.1", "[N2.1] Non soggette ad IVA ai sensi degli artt. Da 7 a 7-septies del DPR 633/72"),
            ("N2.2", "[N2.2] Non soggette - altri casi"),
            ("N3", "[N3] Non imponibili"),
            ("N3.1", "[N3.1] Non imponibili - esportazioni"),
            ("N3.2", "[N3.2] Non imponibili - cessioni intracomunitarie"),
            ("N3.3", "[N3.3] Non imponibili - cessioni verso San Marino"),
            ("N3.4", "[N3.4] Non imponibili - operazioni assimilate alle cessioni all'esportazione"),
            ("N3.5", "[N3.5] Non imponibili - a seguito di dichiarazioni d'intento"),
            ("N3.6", "[N3.6] Non imponibili - altre operazioni che non concorrono alla formazione del plafond"),
            ("N4", "[N4] Esenti"),
            ("N5", "[N5] Regime del margine / IVA non esposta in fattura"),
            ("N6", "[N6] Inversione contabile (per le operazioni in reverse charge ovvero nei casi di autofatturazione per acquisti extra UE di servizi ovvero per importazioni di beni nei soli casi previsti)"),
            ("N6.1", "[N6.1] Inversione contabile - cessione di rottami e altri materiali di recupero"),
            ("N6.2", "[N6.2] Inversione contabile - cessione di oro e argento puro"),
            ("N6.3", "[N6.3] Inversione contabile - subappalto nel settore edile"),
            ("N6.4", "[N6.4] Inversione contabile - cessione di fabbricati"),
            ("N6.5", "[N6.5] Inversione contabile - cessione di telefoni cellulari"),
            ("N6.6", "[N6.6] Inversione contabile - cessione di prodotti elettronici"),
            ("N6.7", "[N6.7] Inversione contabile - prestazioni comparto edile esettori connessi"),
            ("N6.8", "[N6.8] Inversione contabile - operazioni settore energetico"),
            ("N6.9", "[N6.9] Inversione contabile - altri casi"),
            ("N7", "[N7] IVA assolta in altro stato UE (prestazione di servizi di telecomunicazioni, tele-radiodiffusione ed elettronici ex art. 7-octies, comma 1 lett. a, b, art. 74-sexies DPR 633/72)")
        ],
        string="Exoneration",
        help="Exoneration type",
    )
    l10n_it_law_reference = fields.Char(string="Law Reference", size=100)

    @api.constrains('l10n_it_exempt_reason',
                    'l10n_it_law_reference',
                    'amount',
                    'invoice_repartition_line_ids',
                    'refund_repartition_line_ids')
    def _l10n_it_edi_check_exoneration_with_no_tax(self):
        for tax in self:
            if tax.country_id.code == 'IT':
                if tax.amount_type == 'percent' and tax.amount == 0 and not (tax.l10n_it_exempt_reason and tax.l10n_it_law_reference):
                    raise ValidationError(_("If the tax amount is 0%, you must enter the exoneration code and the related law reference."))
                if tax.l10n_it_exempt_reason == 'N6' and tax._l10n_it_is_split_payment():
                    raise UserError(_("Split Payment is not compatible with exoneration of kind 'N6'"))

    def _l10n_it_get_tax_kind(self):
        if self.amount_type == 'percent' and self.amount >= 0:
            return 'vat'
        return None

    def _l10n_it_filter_kind(self, kind):
        """ Filters taxes depending on _l10n_it_get_tax_kind. """
        return self.filtered(lambda tax: tax._l10n_it_get_tax_kind() == kind)

    def _l10n_it_is_split_payment(self):
        """ Split payment means that the Public Administration buyer will pay VAT
            to the tax agency instead of the vendor
        """
        self.ensure_one()

        tax_tags = self.get_tax_tags(is_refund=False, repartition_type='tax') | self.get_tax_tags(is_refund=False, repartition_type='base')
        if not tax_tags:
            return False

        it_tax_report_ve38_lines = self.env['account.report.line'].search([
            ('report_id.country_id.code', '=', 'IT'),
            ('code', '=', 'VE38'),
        ])
        if not it_tax_report_ve38_lines:
            return False

        ve38_lines_tags = it_tax_report_ve38_lines.expression_ids._get_matching_tags()
        return bool(tax_tags & ve38_lines_tags)

```

## File: models\template_it.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('it')
    def _get_it_template_data(self):
        return {
            'property_account_receivable_id': '1501',
            'property_account_payable_id': '2501',
            'property_account_expense_categ_id': '4101',
            'property_account_income_categ_id': '3101',
        }

    @template('it', 'res.company')
    def _get_it_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.it',
                'bank_account_code_prefix': '182',
                'cash_account_code_prefix': '180',
                'transfer_account_code_prefix': '183',
                'account_default_pos_receivable_account_id': '1508',
                'income_currency_exchange_account_id': '3220',
                'expense_currency_exchange_account_id': '4920',
                'account_journal_early_pay_discount_loss_account_id': '4111',
                'account_journal_early_pay_discount_gain_account_id': '3111',
                'tax_calculation_rounding_method': 'round_globally',
                'account_sale_tax_id': '22v',
                'account_purchase_tax_id': '22am',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_it
from . import account_report
from . import account_tax
from . import account_move

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_tax_form_l10n_it" model="ir.ui.view">
        <field name="name">account.tax.form.l10n.it</field>
        <field name="model">account.tax</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//page[@name='advanced_options']" position="inside">
                <group invisible="country_code != 'IT'">
                    <group>
                        <field name="l10n_it_exempt_reason"/>
                        <field name="l10n_it_law_reference"/>
                    </group>
                </group>
            </xpath>
        </data>
        </field>
    </record>

</odoo>

```

