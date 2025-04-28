# Odoo Module: l10n_bo

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
    'name': 'Bolivia - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['bo'],
    'version': '2.0',
    'description': """
Bolivian accounting chart and tax localization.

Plan contable boliviano e impuestos de acuerdo a disposiciones vigentes

    """,
    'author': 'Odoo / Cubic ERP',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account',
    ],
    'data': [
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.bo"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_bo_tax_report_code" model="account.report.column">
                <field name="name">Code</field>
                <field name="expression_label">code</field>
                <field name="figure_type">string</field>
            </record>
            <record id="l10n_bo_tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
                <field name="figure_type">integer</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_bo_tax_report_iva" model="account.report.line">
                <field name="name">Value Added Tax (IVA) - Form 200 v.5</field>
                <field name="code">l10n_bo_tax_report_iva</field>
                <field name="hierarchy_level" eval="0"/>
                <!-- Based on Form 200 v5 - see https://www.impuestos.gob.bo/page/349 -->
                <field name="children_ids">
                    <record id="l10n_bo_tax_report_iva_debit" model="account.report.line">
                        <field name="name">Determination of the Fiscal Debit</field>
                        <field name="code">l10n_bo_tax_report_iva_debit</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_iva_13" model="account.report.line">
                                <field name="name">Sales of taxed goods and/or services in the domestic market, excepting sales taxed at Zero Rate</field>
                                <field name="code">l10n_bo_tax_report_iva_13</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_13_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">13</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_13_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 13</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_14" model="account.report.line">
                                <field name="name">Export of goods and exempt operations</field>
                                <field name="code">l10n_bo_tax_report_iva_14</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_14_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">14</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_14_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 14</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_15" model="account.report.line">
                                <field name="name">Sales taxed at Zero Rate</field>
                                <field name="code">l10n_bo_tax_report_iva_15</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_15_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">15</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_15_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 15</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_505" model="account.report.line">
                                <field name="name">Untaxed sales and operations not subject to IVA</field>
                                <field name="code">l10n_bo_tax_report_iva_505</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_505_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">505</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_505_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 505</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_16" model="account.report.line">
                                <field name="name">Value attributed to retired goods and/or services and private consumption</field>
                                <field name="code">l10n_bo_tax_report_iva_16</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_16_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">16</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_16_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 16</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_17" model="account.report.line">
                                <field name="name">Returns and rescinders performed during the period</field>
                                <field name="code">l10n_bo_tax_report_iva_17</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_17_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">17</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_17_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 17</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_18" model="account.report.line">
                                <field name="name">Discounts, bonuses and rebates obtained during the period</field>
                                <field name="code">l10n_bo_tax_report_iva_18</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_18_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">18</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_18_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_39" model="account.report.line">
                                <field name="name">Tax Payable corresponding to: (C13+C16+C17+C18) * 13%</field>
                                <field name="code">l10n_bo_tax_report_iva_39</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_39_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">39</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_39_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">0.13 * (l10n_bo_tax_report_iva_13.balance + l10n_bo_tax_report_iva_16.balance + l10n_bo_tax_report_iva_17.balance + l10n_bo_tax_report_iva_18.balance)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_55" model="account.report.line">
                                <field name="name">Adjusted Tax Payable based on refunds</field>
                                <field name="code">l10n_bo_tax_report_iva_debit_donations</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_55_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">55</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_55_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 55</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_19" model="account.report.line">
                                <field name="name">Adjusted Tax Payable based on Reconciliations</field>
                                <field name="code">l10n_bo_tax_report_iva_19</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_19_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">19</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_19_tax_amount_balance" model="account.report.expression">
                                        <field name="label">tax_amount</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 19</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_19_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_19.tax_amount - l10n_bo_tax_report_iva_39.balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_1002" model="account.report.line">
                                <field name="name">Total Tax Payable of the period (C39+C55+C19)</field>
                                <field name="code">l10n_bo_tax_report_iva_1002</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_1002_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">1002</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_1002_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_39.balance + l10n_bo_tax_report_iva_debit_donations.balance + l10n_bo_tax_report_iva_19.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_iva_credit" model="account.report.line">
                        <field name="name">Determination of the Tax Credit</field>
                        <field name="code">l10n_bo_tax_report_iva_credit</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_iva_11" model="account.report.line">
                                <field name="name">Total purchases corresponding to taxed and/or non-taxed activities</field>
                                <field name="code">l10n_bo_tax_report_iva_11</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_11_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">11</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_11_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 11</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_26" model="account.report.line">
                                <field name="name">Purchases corresponding to taxed activities</field>
                                <field name="code">l10n_bo_tax_report_iva_26</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_26_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">26</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_26_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 26</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_31" model="account.report.line">
                                <field name="name">Purchases for which it is not possible to discriminate between taxed and non-taxed activities</field>
                                <field name="code">l10n_bo_tax_report_iva_31</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_31_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">31</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_31_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 31</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_27" model="account.report.line">
                                <field name="name">Returns and rescinders received during the period</field>
                                <field name="code">l10n_bo_tax_report_iva_27</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_27_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">27</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_27_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_28" model="account.report.line">
                                <field name="name">Discounts, bonuses and rebates granted during the period</field>
                                <field name="code">l10n_bo_tax_report_iva_28</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_28_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">28</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_28_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 28</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_114" model="account.report.line">
                                <field name="name">Tax Credit: (C26+C27+C28) * 13%</field>
                                <field name="code">l10n_bo_tax_report_iva_114</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_114_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">114</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_114_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">0.13 * (l10n_bo_tax_report_iva_26.balance + l10n_bo_tax_report_iva_27.balance + l10n_bo_tax_report_iva_28.balance)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_30" model="account.report.line">
                                <field name="name">Adjusted Tax Credit based on Reconciliations</field>
                                <field name="code">l10n_bo_tax_report_iva_30</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_30_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">30</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_30_tax_amount_balance" model="account.report.expression">
                                        <field name="label">tax_amount</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IVA 30</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_30_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_30.tax_amount - l10n_bo_tax_report_iva_114.balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_1003" model="account.report.line">
                                <field name="name">Proportional tax credit corresponding to taxed activities C31*(C13+C14)/(C13+C14+C15+C505) * 13%</field>
                                <field name="code">l10n_bo_tax_report_iva_1003</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_1003_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">1003</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_1003_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">0.13 * l10n_bo_tax_report_iva_31.balance * (l10n_bo_tax_report_iva_13.balance + l10n_bo_tax_report_iva_14.balance) / (l10n_bo_tax_report_iva_13.balance + l10n_bo_tax_report_iva_14.balance + l10n_bo_tax_report_iva_15.balance + l10n_bo_tax_report_iva_505.balance)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_1004" model="account.report.line">
                                <field name="name">Total Fiscal Credit for the period (C114+C30+C1003)</field>
                                <field name="code">l10n_bo_tax_report_iva_1004</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_1004_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">1004</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_1004_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_114.balance + l10n_bo_tax_report_iva_30.balance + l10n_bo_tax_report_iva_1003.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_iva_difference" model="account.report.line">
                        <field name="name">Determination of the difference</field>
                        <field name="code">l10n_bo_tax_report_iva_difference</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_iva_693" model="account.report.line">
                                <field name="name">Difference in favor of the Taxpayer (C1004-C1002; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_693</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_693_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">693</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_693_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_1004.balance - l10n_bo_tax_report_iva_1002.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_909" model="account.report.line">
                                <field name="name">Difference in favor of the Tax Authority or determined Tax Amount (C1002-C1004; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_909</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_909_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">909</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_909_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_1002.balance - l10n_bo_tax_report_iva_1004.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_635" model="account.report.line">
                                <field name="name">Tax Credit balance to offset for the previous period (C592 of the Form for the previous period)</field>
                                <field name="code">l10n_bo_tax_report_iva_635</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_635_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">635</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_635_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_648" model="account.report.line">
                                <field name="name">Value Adjustment on the Tax Credit balance for the previous period</field>
                                <field name="code">l10n_bo_tax_report_iva_648</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_648_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">648</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_648_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_1001" model="account.report.line">
                                <field name="name">Tax Balance Determined in favor of the Tax Authority (C909-C635-C648; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_1001</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_1001_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">1001</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_1001_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_909.balance - l10n_bo_tax_report_iva_635.balance - l10n_bo_tax_report_iva_648.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <!-- Note: this tax relief provision ended in December 2021, so is no longer in use.-->
                            <record id="l10n_bo_tax_report_iva_621" model="account.report.line">
                                <field name="name">Payment to account of 50% of paid employer contributions</field>
                                <field name="code">l10n_bo_tax_report_iva_621</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_621_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">621</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_621_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_629" model="account.report.line">
                                <field name="name">Balance in favour of the Tax Authority after offsetting payments on account for employer's contributions (C1001 -C621; If > 0) ((C621 - C1001; If > 0) = 0)</field>
                                <field name="code">l10n_bo_tax_report_iva_629</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_629_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">629</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_629_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_1001.balance - l10n_bo_tax_report_iva_621.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_622" model="account.report.line">
                                <field name="name">Payments to Account already made in Declarations and/or Payment Tickets corresponding to the declared period</field>
                                <field name="code">l10n_bo_tax_report_iva_622</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_622_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">622</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_622_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_640" model="account.report.line">
                                <field name="name">Balance of Payments on Account to offset from the previous period (C747 of the Form for the previous period)</field>
                                <field name="code">l10n_bo_tax_report_iva_640</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_640_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">640</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_640_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_643" model="account.report.line">
                                <field name="name">Balance for Payments on Account in favor of the Taxpayer (C640+C622-C629; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_643</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_643_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">643</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_643_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_640.balance + l10n_bo_tax_report_iva_622.balance - l10n_bo_tax_report_iva_629.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_468" model="account.report.line">
                                <field name="name">Balance in favor of the Tax Authority after offsetting payments on account (C629-C622-C640; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_468</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_468_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">468</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_468_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_629.balance - l10n_bo_tax_report_iva_622.balance - l10n_bo_tax_report_iva_640.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <!-- Note: this fiscal regime ended in December 2021, so is no longer in use.-->
                            <record id="l10n_bo_tax_report_iva_465" model="account.report.line">
                                <field name="name">5% payment for purchases to SIETE-RG taxpayers</field>
                                <field name="code">l10n_bo_tax_report_iva_465</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_465_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">465</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_465_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_466" model="account.report.line">
                                <field name="name">Balance of Payments on Account for 5% of purchases to SIETE-RG taxpayers (C469 of the previous period)</field>
                                <field name="code">l10n_bo_tax_report_iva_466</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_466_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">466</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_466_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_467" model="account.report.line">
                                <field name="name">Balance in favor after offsetting 5% of payments on account for purchases from SIETE-RG taxpayers (C465+C466-C468; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_467</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_467_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">467</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_467_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_465.balance + l10n_bo_tax_report_iva_466.balance - l10n_bo_tax_report_iva_468.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_996" model="account.report.line">
                                <field name="name">Balance in favor of the Tax Authority (C468-C465-C466; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_996</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_996_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">996</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_996_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_468.balance - l10n_bo_tax_report_iva_465.balance - l10n_bo_tax_report_iva_466.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_iva_payable" model="account.report.line">
                        <field name="name">Determination of Tax Payable</field>
                        <field name="code">l10n_bo_tax_report_iva_payable</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_iva_924" model="account.report.line">
                                <field name="name">Unpaid Tax (C996)</field>
                                <field name="code">l10n_bo_tax_report_iva_924</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_924_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">924</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_924_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_996.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_925" model="account.report.line">
                                <field name="name">Value Adjustment on Unpaid Tax</field>
                                <field name="code">l10n_bo_tax_report_iva_925</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_925_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">925</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_925_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_938" model="account.report.line">
                                <field name="name">Interest on Adjusted Unpaid Tax</field>
                                <field name="code">l10n_bo_tax_report_iva_938</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_938_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">938</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_938_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_954" model="account.report.line">
                                <field name="name">Fine for Non-Compliance to Formal Duty (IDF) for overdue declaration</field>
                                <field name="code">l10n_bo_tax_report_iva_954</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_954_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">954</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_954_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_967" model="account.report.line">
                                <field name="name">Fine levied by the IDF for late declaration of a correction in the Tax Amount</field>
                                <field name="code">l10n_bo_tax_report_iva_967</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_967_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">967</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_967_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_955" model="account.report.line">
                                <field name="name">Total Tax Payable (C924+C925+C938+C954+C967)</field>
                                <field name="code">l10n_bo_tax_report_iva_955</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_955_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">955</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_955_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_924.balance + l10n_bo_tax_report_iva_925.balance + l10n_bo_tax_report_iva_938.balance + l10n_bo_tax_report_iva_954.balance + l10n_bo_tax_report_iva_967.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_iva_balance" model="account.report.line">
                        <field name="name">Definitive balance</field>
                        <field name="code">l10n_bo_tax_report_iva_balance</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_iva_592" model="account.report.line">
                                <field name="name">Final Tax Credit Balance in favor of the Taxpayer for the next period (C693+C635+C648-C909; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_592</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_592_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">592</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_592_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_693.balance + l10n_bo_tax_report_iva_635.balance + l10n_bo_tax_report_iva_648.balance - l10n_bo_tax_report_iva_909.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_469" model="account.report.line">
                                <field name="name">Final Balance for Payments on Account in favor of the taxpayer of the 5% for purchases to SIETE-RG contributors for the next period (C467; Yes )</field>
                                <field name="code">l10n_bo_tax_report_iva_469</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_469_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">469</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_469_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_467.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_747" model="account.report.line">
                                <field name="name">Final Balance for Payments on Account in favor of the Taxpayer for the next period (C643-C955; If>0)</field>
                                <field name="code">l10n_bo_tax_report_iva_747</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_747_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">747</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_747_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_643.balance - l10n_bo_tax_report_iva_955.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_iva_646" model="account.report.line">
                                <field name="name">Final balance in favour of the Tax Authority (C996 or (C955-C643) as appropriate; If >0)</field>
                                <field name="code">l10n_bo_tax_report_iva_646</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_iva_646_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">646</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_iva_646_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_iva_955.balance - l10n_bo_tax_report_iva_643.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bo_tax_report_it" model="account.report.line">
                <field name="name">Transaction Tax (IT) - Form 400 v.5</field>
                <field name="code">l10n_bo_tax_report_it</field>
                <field name="hierarchy_level" eval="0"/>
                <!-- Based on Form 400 v5 - see https://www.impuestos.gob.bo/page/349 -->
                <field name="children_ids">
                    <record id="l10n_bo_tax_report_it_base" model="account.report.line">
                        <field name="name">Determination of the Base Amount and the Tax Amount</field>
                        <field name="code">l10n_bo_tax_report_it_base</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_it_13" model="account.report.line">
                                <field name="name">Total gross income earned and/or received in kind (including exempt income)</field>
                                <field name="code">l10n_bo_tax_report_it_13</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_13_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">13</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_13_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_32.balance + l10n_bo_tax_report_it_24.balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_32" model="account.report.line">
                                <field name="name">Exempt income</field>
                                <field name="code">l10n_bo_tax_report_it_32</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_32_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">32</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_32_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IT 32</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_24" model="account.report.line">
                                <field name="name">Taxable base (C13 - C32)</field>
                                <field name="code">l10n_bo_tax_report_it_24</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_24_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">24</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_24_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IT 24</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_909" model="account.report.line">
                                <field name="name">Determined tax (C24 * 3%)</field>
                                <field name="code">l10n_bo_tax_report_it_909</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_909_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">909</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_909_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_24.balance * 0.03</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_it_balance" model="account.report.line">
                        <field name="name">Determination of the Balance</field>
                        <field name="code">l10n_bo_tax_report_it_balance</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_it_664" model="account.report.line">
                                <field name="name">Paid IUE to offset (1st Instance or “Balance of IUE to offset” of Form 400, C619 of the previous period)</field>
                                <field name="code">l10n_bo_tax_report_it_664</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_664_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">664</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_664_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_1001" model="account.report.line">
                                <field name="name">Tax Balance Determined in favor of the Tax Authority (C909 - C664; If >0)</field>
                                <field name="code">l10n_bo_tax_report_it_1001</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_1001_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">1001</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_1001_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_909.balance - l10n_bo_tax_report_it_664.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_622" model="account.report.line">
                                <field name="name">Payments to Account already made in Declarations and/or Payment Tickets corresponding to the declared period</field>
                                <field name="code">l10n_bo_tax_report_it_622</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_622_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">622</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_622_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_640" model="account.report.line">
                                <field name="name">Balance of Payments on Account to offset from the previous period (C747 of Form 400 of the previous period)</field>
                                <field name="code">l10n_bo_tax_report_it_640</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_640_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">640</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_640_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_643" model="account.report.line">
                                <field name="name">Balance for Payments on Account in favor of the Taxpayer (C622+C640-C1001; If >0)</field>
                                <field name="code">l10n_bo_tax_report_it_643</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_643_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">643</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_643_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_622.balance + l10n_bo_tax_report_it_640.balance - l10n_bo_tax_report_it_1001.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_996" model="account.report.line">
                                <field name="name">Balance in favor of the Tax Authority (C1001-C622-C640; If >0)</field>
                                <field name="code">l10n_bo_tax_report_it_996</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_996_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">996</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_996_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_1001.balance - l10n_bo_tax_report_it_622.balance - l10n_bo_tax_report_it_640.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_it_payable" model="account.report.line">
                        <field name="name">Determination of Tax Payable</field>
                        <field name="code">l10n_bo_tax_report_it_payable</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_it_924" model="account.report.line">
                                <field name="name">Unpaid Tax (C996)</field>
                                <field name="code">l10n_bo_tax_report_it_924</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_924_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">924</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_924_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_996.balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_925" model="account.report.line">
                                <field name="name">Value Adjustment on Unpaid Tax</field>
                                <field name="code">l10n_bo_tax_report_it_925</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_925_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">925</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_925_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_938" model="account.report.line">
                                <field name="name">Interest on Adjusted Unpaid Tax</field>
                                <field name="code">l10n_bo_tax_report_it_938</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_938_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">938</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_938_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_954" model="account.report.line">
                                <field name="name">Fine for Non-Compliance to Formal Duty (IDF) for overdue declaration</field>
                                <field name="code">l10n_bo_tax_report_it_954</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_954_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">954</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_954_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_967" model="account.report.line">
                                <field name="name">Fine levied by the IDF for late declaration of a correction in the Tax Amount</field>
                                <field name="code">l10n_bo_tax_report_it_967</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_967_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">967</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_967_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_955" model="account.report.line">
                                <field name="name">Total Tax Payable (C924+C925+C938+C954+C967)</field>
                                <field name="code">l10n_bo_tax_report_it_955</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_955_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">955</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_955_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_924.balance + l10n_bo_tax_report_it_925.balance + l10n_bo_tax_report_it_938.balance + l10n_bo_tax_report_it_954.balance + l10n_bo_tax_report_it_967.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_it_final_balance" model="account.report.line">
                        <field name="name">Definitive balance</field>
                        <field name="code">l10n_bo_tax_report_it_final_balance</field>
                        <field name="children_ids">
                            <record id="l10n_bo_tax_report_it_619" model="account.report.line">
                                <field name="name">Final balance of IUE to offset for the next period (C664-C909; If >0)</field>
                                <field name="code">l10n_bo_tax_report_it_619</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_619_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">619</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_619_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_664.balance - l10n_bo_tax_report_it_909.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_747" model="account.report.line">
                                <field name="name">Final Balance for Payments on Account in favor of the Taxpayer for the next period (C643-C955; If >0)</field>
                                <field name="code">l10n_bo_tax_report_it_747</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_747_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">747</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_747_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_643.balance - l10n_bo_tax_report_it_955.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bo_tax_report_it_646" model="account.report.line">
                                <field name="name">Final balance in favour of the Tax Authority (C996 or (C955-C643) as appropriate; If >0)</field>
                                <field name="code">l10n_bo_tax_report_it_646</field>
                                <field name="expression_ids">
                                    <record id="l10n_bo_tax_report_it_646_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">646</field>
                                    </record>
                                    <record id="l10n_bo_tax_report_it_646_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_bo_tax_report_it_955.balance - l10n_bo_tax_report_it_643.balance</field>
                                        <field name="subformula">if_above(BOB(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bo_tax_report_whh_it" model="account.report.line">
                <field name="name">IT Withholdings - Form 410</field>
                <field name="code">l10n_bo_tax_report_whh_it</field>
                <field name="hierarchy_level" eval="0"/>
                <!-- Based on Form 410 - see https://www.impuestos.gob.bo/page/349 -->
                <field name="children_ids">
                    <record id="l10n_bo_tax_report_whh_it_13" model="account.report.line">
                        <field name="name">Retained tax</field>
                        <field name="code">l10n_bo_tax_report_whh_it_13</field>
                        <field name="hierarchy_level" eval="5"/>
                        <field name="expression_ids">
                            <record id="l10n_bo_tax_report_whh_it_13_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">13</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_it_13_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Ret IT 13</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bo_tax_report_whh_iue" model="account.report.line">
                <field name="name">IUE Withholdings - Form 570</field>
                <field name="code">l10n_bo_tax_report_whh_iue</field>
                <field name="hierarchy_level" eval="0"/>
                <!-- Based on Form 570 - see https://www.impuestos.gob.bo/page/349 -->
                <field name="children_ids">
                    <record id="l10n_bo_tax_report_whh_iue_13" model="account.report.line">
                        <field name="name">Amount Paid for Services</field>
                        <field name="code">l10n_bo_tax_report_whh_iue_13</field>
                        <field name="hierarchy_level" eval="5"/>
                        <field name="expression_ids">
                            <record id="l10n_bo_tax_report_whh_iue_13_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">13</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_iue_13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Ret IUE 13</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_iue_13_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_bo_tax_report_whh_iue_13.base / 0.845</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_whh_iue_26" model="account.report.line">
                        <field name="name">Amount Paid for Acquisitions</field>
                        <field name="code">l10n_bo_tax_report_whh_iue_26</field>
                        <field name="hierarchy_level" eval="5"/>
                        <field name="expression_ids">
                            <record id="l10n_bo_tax_report_whh_iue_26_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">26</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_iue_26_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Ret IUE 26</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_iue_26_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_bo_tax_report_whh_iue_26.base / 0.92</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_whh_iue_1001" model="account.report.line">
                        <field name="name">Amount Subject to Tax (C13 * 0.5)</field>
                        <field name="code">l10n_bo_tax_report_whh_iue_1001</field>
                        <field name="hierarchy_level" eval="5"/>
                        <field name="expression_ids">
                            <record id="l10n_bo_tax_report_whh_iue_1001_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">1001</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_iue_1001_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_bo_tax_report_whh_iue_13.balance * 0.5</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_whh_iue_1002" model="account.report.line">
                        <field name="name">Amount Subject to Tax (C26 * 0.2)</field>
                        <field name="code">l10n_bo_tax_report_whh_iue_1002</field>
                        <field name="hierarchy_level" eval="5"/>
                        <field name="expression_ids">
                            <record id="l10n_bo_tax_report_whh_iue_1002_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">1002</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_iue_1002_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_bo_tax_report_whh_iue_26.balance * 0.5</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bo_tax_report_whh_iue_909" model="account.report.line">
                        <field name="name">Determined tax (0.25 * (C1001 + C1002))</field>
                        <field name="code">l10n_bo_tax_report_whh_iue_909</field>
                        <field name="hierarchy_level" eval="5"/>
                        <field name="expression_ids">
                            <record id="l10n_bo_tax_report_whh_iue_909_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">909</field>
                            </record>
                            <record id="l10n_bo_tax_report_whh_iue_909_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">0.25 * (l10n_bo_tax_report_whh_iue_1001.balance + l10n_bo_tax_report_whh_iue_1002.balance)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-bo.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@es"
"l10n_bo_1112","Petty Cash","1112","asset_cash","","False","Fondo fijo"
"l10n_bo_1114","Temporary Investments","1114","asset_cash","","False","Inversiones Temporales"
"l10n_bo_1121","Commercial Accounts Receivable","1121","asset_receivable","account.account_tag_operating","True","Cuentas por Cobrar Comerciales"
"l10n_bo_11211","Commercial Accounts Receivable - PoS","11211","asset_receivable","account.account_tag_operating","True","Cuentas por Cobrar Comerciales - PoS"
"l10n_bo_1122","Other Accounts Receivable","1122","asset_receivable","account.account_tag_operating","True","Otras Cuentas por Cobrar"
"l10n_bo_1123","Accounts Receivable from Related Companies","1123","asset_receivable","account.account_tag_operating","True","Cuentas por Cobrar a Empresas Relacionadas"
"l10n_bo_1124","Advances to Suppliers","1124","asset_prepayments","account.account_tag_operating","False","Anticipo a Proveedores"
"l10n_bo_1125","Provision for Bad Accounts","1125","asset_receivable","account.account_tag_operating","True","Provisión para Cuentas Incobrables"
"l10n_bo_1131","Inventory - Finished Products","1131","asset_current","account.account_tag_operating","False","Inventarios de Productos Terminados"
"l10n_bo_1132","Inventory - Work in Process","1132","asset_current","account.account_tag_operating","False","Inventarios de Productos en Proceso"
"l10n_bo_1133","Inventory - Raw Materials","1133","asset_current","account.account_tag_operating","False","Inventarios de Materia Prima"
"l10n_bo_11341","Stock Interim (Received)","11341","asset_current","account.account_tag_operating","False","Inventario en Tránsito Entrante"
"l10n_bo_11342","Stock Interim (Delivered)","11342","asset_current","account.account_tag_operating","False","Inventario en Tránsito Saliente"
"l10n_bo_1135","Inventory - Provision for Obsolescence","1135","asset_current","account.account_tag_operating","False","Inventario - Provisión para Obsolescencias"
"l10n_bo_1141","IVA Tax Credit","1141","asset_prepayments","account.account_tag_operating","False","Crédito Fiscal IVA"
"l10n_bo_1142","Taxes to Recover","1142","asset_prepayments","account.account_tag_operating","False","Impuestos por Recuperar"
"l10n_bo_1143","Advance Payments","1143","asset_current","account.account_tag_operating","False","Pagos Anticipados"
"l10n_bo_1144","Other Current Assets","1144","asset_current","account.account_tag_investing","False","Otros Activos Corrientes"
"l10n_bo_1211","Commercial Accounts Receivable - Long Term","1211","asset_receivable","account.account_tag_operating","True","Cuentas por Cobrar Comerciales - Largo Plazo"
"l10n_bo_1212","Other Accounts Receivable - Long Term","1212","asset_receivable","account.account_tag_operating","True","Otras Cuentas por Cobrar - Largo Plazo"
"l10n_bo_1213","Long-Term Accounts Receivable from Related and/or Linked Companies","1213","asset_receivable","account.account_tag_operating","True","Cuentas por Cobrar Empresas Relacionadas y/o Vinculadas Largo Plazo"
"l10n_bo_1221","Inventory - Spare Parts","1221","asset_non_current","account.account_tag_investing","True","Inventarios de Repuestos"
"l10n_bo_1222","Other Inventory","1222","asset_non_current","account.account_tag_investing","True","Otros Inventarios"
"l10n_bo_1231","Land","1231","asset_fixed","account.account_tag_investing","False","Terrenos"
"l10n_bo_1232","Buildings","1232","asset_fixed","account.account_tag_investing","False","Edificios"
"l10n_bo_12321","Accumulated Depreciation Buildings","12321","asset_fixed","account.account_tag_investing","False","Depreciación Acumulada Edificios"
"l10n_bo_1233","Machinery","1233","asset_fixed","account.account_tag_investing","False","Maquinaria"
"l10n_bo_12331","Accumulated Depreciation Machinery","12331","asset_fixed","account.account_tag_investing","False","Depreciación Acumulada Maquinaria"
"l10n_bo_1234","Vehicles","1234","asset_fixed","account.account_tag_investing","False","Vehículos"
"l10n_bo_12341","Accumulated Depreciation Vehicles","12341","asset_fixed","account.account_tag_investing","False","Depreciación Acumulada Vehículos"
"l10n_bo_1235","Furniture and Fixtures","1235","asset_fixed","account.account_tag_investing","False","Muebles y Enseres"
"l10n_bo_12351","Accumulated Depreciation Furniture and Fixtures","12351","asset_fixed","account.account_tag_investing","False","Depreciación Acumulada Muebles y Enseres"
"l10n_bo_1236","Computer Equipment","1236","asset_fixed","account.account_tag_investing","False","Equipos de Computación"
"l10n_bo_12361","Accumulated Depreciation Computer Equipment","12361","asset_fixed","account.account_tag_investing","False","Depreciación Acumulada Equipos de Computación"
"l10n_bo_124","Investment Properties","124","asset_non_current","account.account_tag_investing","False","Propriedades de Inversión"
"l10n_bo_1251","Patents and Trademarks","1251","asset_non_current","account.account_tag_investing","False","Patentes y Marcas"
"l10n_bo_12511","Accumulated Amortization Patents and Trademarks","12511","asset_non_current","account.account_tag_investing","False","Amortización Acumulada Patentes y Marcas"
"l10n_bo_1252","Rights of Use","1252","asset_non_current","account.account_tag_investing","False","Derechos de Llave"
"l10n_bo_12521","Accumulated Amortization Rights of Use","12521","asset_non_current","account.account_tag_investing","False","Amortización Acumulada Derechos de Llave"
"l10n_bo_126","Permanent Investments","126","asset_non_current","account.account_tag_investing","False","Inversiones Permanentes"
"l10n_bo_127","Other Non-Current Assets","127","asset_non_current","account.account_tag_operating","False","Otros Activos No Corrientes"
"l10n_bo_2111","Bank loans","2111","liability_current","account.account_tag_financing","False","Préstamos Bancarios"
"l10n_bo_2112","Other Financial Liabilities","2112","liability_current","account.account_tag_financing","False","Otros Pasivos Financieros"
"l10n_bo_2113","Interest Payable","2113","liability_current","account.account_tag_financing","False","Intereses por Pagar"
"l10n_bo_2121","Commercial Accounts Payable","2121","liability_payable","account.account_tag_operating","True","Cuentas por Pagar Comerciales"
"l10n_bo_2122","Notes Payable Short Term","2122","liability_payable","account.account_tag_operating","True","Documentos por Pagar Corto Plazo"
"l10n_bo_2123","Accounts Payable to Related Companies","2123","liability_payable","account.account_tag_operating","True","Cuentas por Pagar a Empresas Relacionadas"
"l10n_bo_2131","Short-term Contributions and Withholdings","2131","liability_current","account.account_tag_operating","False","Aportes y Retenciones a Corto Plazo"
"l10n_bo_2132","Wages Payable","2132","liability_current","account.account_tag_operating","False","Sueldos por Pagar"
"l10n_bo_2133","Social Benefits Payable","2133","liability_current","account.account_tag_operating","False","Beneficios Sociales por Pagar"
"l10n_bo_2134","Social Charges","2134","liability_current","account.account_tag_operating","False","Cargos Sociales"
"l10n_bo_2135","IVA Tax Debit","2135","liability_current","account.account_tag_operating","True","Débito Fiscal IVA"
"l10n_bo_2136","Transaction Tax Payable","2136","liability_current","account.account_tag_operating","True","Impuesto a las Transacciones por Pagar"
"l10n_bo_2137","IT Withholdings Payable","2137","liability_current","account.account_tag_operating","False","Retenciones IT por Pagar"
"l10n_bo_2138","IUE Withholdings Payable","2138","liability_current","account.account_tag_operating","False","Retenciones IUE por Pagar"
"l10n_bo_2139","Other Tax Payable","2139","liability_current","account.account_tag_operating","False","Otros Impuestos por Pagar"
"l10n_bo_214","Provisions","214","liability_current","account.account_tag_operating","False","Provisiones"
"l10n_bo_215","Deferred Income","215","liability_current","account.account_tag_financing","False","Ingresos Diferidos"
"l10n_bo_216","Other Current Liabilities","216","liability_current","account.account_tag_financing","False","Otros Pasivos Corrientes"
"l10n_bo_2211","Bank loans - Long Term","2211","liability_current","account.account_tag_operating","False","Préstamos Bancarios a Largo Plazo"
"l10n_bo_2212","Other Long-Term Financial Liabilities","2212","liability_current","account.account_tag_operating","False","Otros Pasivos Financieros a Largo Plazo"
"l10n_bo_2221","Notes Payable Long Term","2221","liability_payable","account.account_tag_operating","True","Documentos por Pagar a Largo Plazo"
"l10n_bo_2222","Long-term Accounts Payable to Related Companies","2222","liability_non_current","account.account_tag_operating","False","Cuentas por Pagar a Empresas Relacionadas a Largo Plazo"
"l10n_bo_223","Provision for Social Benefits (Personnel Compensation)","223","liability_non_current","account.account_tag_operating","False","Provisión para Beneficios Sociales (Indemnizaciones al Personal)"
"l10n_bo_224","Other Non-Current Liabilities","224","liability_non_current","account.account_tag_operating","False","Otros Pasivos No Corrientes"
"l10n_bo_3101","Paid Share Capital","3101","equity","account.account_tag_financing","False","Capital Social Pagado"
"l10n_bo_3102","Contributions to Capitalize","3102","equity","account.account_tag_financing","False","Aportes por Capitalizar"
"l10n_bo_3103","Capital Adjustment","3103","equity","account.account_tag_financing","False","Ajuste de Capital"
"l10n_bo_3201","Legal Reserve","3201","equity","account.account_tag_financing","False","Reserva Legal"
"l10n_bo_3202","Other Reserves","3202","equity","account.account_tag_financing","False","Otras Reservas"
"l10n_bo_3203","Adjustment of Equity Reserves","3203","equity","account.account_tag_financing","False","Ajuste de Reservas Patrimoniales"
"l10n_bo_331","Retained Earnings","331","equity","account.account_tag_financing","False","Resultados Acumulados"
"l10n_bo_332","Current Year Earnings","332","equity_unaffected","account.account_tag_financing","False","Resultados de la Gestión"
"l10n_bo_4101","Sales","4101","income","account.account_tag_operating","False","Ventas"
"l10n_bo_4102","Returns, Rebates and Discounts of Goods and/or Services","4102","income","account.account_tag_operating","False","Devoluciones, Rebajas y Descuetos de Bienes y/o Servicios"
"l10n_bo_4201","Interest on Bank Deposits","4201","income","account.account_tag_investing","False","Intereses Sobre Depósitos Bancarios"
"l10n_bo_4202","Interest on Temporary Investments","4202","income","account.account_tag_investing","False","Intereses de Inversiones Temporales"
"l10n_bo_4203","Other Financial Income","4203","income","account.account_tag_investing","False","Otros Ingresos Financieros"
"l10n_bo_4301","Adjustment for Inflation and Property Ownership","4301","income","account.account_tag_operating","False","Ajuste por Inflación y Tenencie de Bienes"
"l10n_bo_4302","Income from Sale of Securities","4302","income","account.account_tag_operating","False","Ingresos por Venta de Valores"
"l10n_bo_4303","Exchange Gain","4303","income","account.account_tag_operating","False","Ganancia Cambiaria"
"l10n_bo_4304","Other Income","4304","income_other","account.account_tag_operating","False","Otros Ingresos"
"l10n_bo_5101","Product Costs","5101","expense","account.account_tag_operating","False","Costos de Productos"
"l10n_bo_5102","Freight and Transportation of Products","5102","expense","account.account_tag_operating","False","Fletes y Transportes de Productos"
"l10n_bo_5103","Returns on Purchases","5103","expense","account.account_tag_operating","False","Devolución en Compras"
"l10n_bo_5104","Discounts on Purchases","5104","expense","account.account_tag_operating","False","Descuentos Sobre Compras"
"l10n_bo_5105","Cost of Damaged Products","5105","expense","account.account_tag_operating","False","Costo de Productos Dañados"
"l10n_bo_5201","Costs of Sale - Wages and Salaries","5201","expense","account.account_tag_operating","False","Gastos de Comercialización - Sueldos y Salarios"
"l10n_bo_5202","Costs of Sale - Social Benefits","5202","expense","account.account_tag_operating","False","Gastos de Comercialización - Beneficios Sociales"
"l10n_bo_5203","Costs of Sale - Sales Commissions","5203","expense","account.account_tag_operating","False","Gastos de Comercialización - Comisiones Sobre Ventas"
"l10n_bo_5204","Costs of Sale - Per diems","5204","expense","account.account_tag_operating","False","Gastos de Comercialización - Víaticos"
"l10n_bo_5205","Costs of Sale - Tickets","5205","expense","account.account_tag_operating","False","Gastos de Comercialización - Pasajes"
"l10n_bo_5206","Costs of Sale - Advertising","5206","expense","account.account_tag_operating","False","Gastos de Comercialización - Publicidad"
"l10n_bo_5207","Costs of Sale - Depreciation of Fixed Assets","5207","expense","account.account_tag_operating","False","Gastos de Comercialización - Depreciación de Bienes de Uso"
"l10n_bo_5208","Costs of Sale - Loss on Bad Accounts","5208","expense","account.account_tag_operating","False","Gastos de Comercialización - Pérdida en Cuentas Incobrabres"
"l10n_bo_5209","Other Costs of Sale","5209","expense","account.account_tag_operating","False","Otros Gastos de Comercialización"
"l10n_bo_52091","Transaction Tax","52091","expense","account.account_tag_operating","False","Impuesto a las Transacciones"
"l10n_bo_52092","IT Withholdings","52092","expense","account.account_tag_operating","False","Retenciones IT"
"l10n_bo_52093","IUE Withholdings","52093","expense","account.account_tag_operating","False","Retenciones IUE"
"l10n_bo_53001","Administration Expenses - Wages and Salaries","53001","expense","account.account_tag_operating","False","Gastos de Administración - Sueldos y Salarios"
"l10n_bo_53002","Administration Expenses - Social Benefits","53002","expense","account.account_tag_operating","False","Gastos de Administración - Beneficios Sociales"
"l10n_bo_53003","Administrative Expenses - Provision for Christmas Bonuses","53003","expense","account.account_tag_operating","False","Gastos de Administración - Provisión Aguinaldos"
"l10n_bo_53004","Administration Expenses - Provision for Compensations","53004","expense","account.account_tag_operating","False","Gastos de Administración - Provisión Indemnizaciones"
"l10n_bo_53005","Administration Expenses - Per diem","53005","expense","account.account_tag_operating","False","Gastos de Administración - Víaticos"
"l10n_bo_53006","Administration Expenses - Tickets","53006","expense","account.account_tag_operating","False","Gastos de Administración - Pasajes"
"l10n_bo_53007","Administration Expenses - Basic Services","53007","expense","account.account_tag_operating","False","Gastos de Administración - Servicios Básicos"
"l10n_bo_53008","Administration Expenses - Materials and Supplies","53008","expense","account.account_tag_operating","False","Gastos de Administración - Materiales y Suministros"
"l10n_bo_53009","Administration Expenses - Freight and Transportation","53009","expense","account.account_tag_operating","False","Gastos de Administración - Fletes y Transporte"
"l10n_bo_53010","Administration Expenses - Maintenance and Repair","53010","expense","account.account_tag_operating","False","Gastos de Administración - Mantenimento y Reparación"
"l10n_bo_53011","Administration Expenses - Depreciation of Fixed Assets","53011","expense","account.account_tag_operating","False","Gastos de Administración - Depreciación de Bienes de Uso"
"l10n_bo_53012","Administration Expenses - Rentals","53012","expense","account.account_tag_operating","False","Gastos de Administración - Alquileres"
"l10n_bo_53013","Administration Expenses - Insurance","53013","expense","account.account_tag_operating","False","Gastos de Administración - Seguros"
"l10n_bo_53014","Administration Expenses - Security Service","53014","expense","account.account_tag_operating","False","Gastos de Administración - Servicio de Seguridad"
"l10n_bo_53015","Administration Expenses - General Expenses","53015","expense","account.account_tag_operating","False","Gastos de Administración - Gastos Generales"
"l10n_bo_53016","Other Administrative Expenses","53016","expense","account.account_tag_operating","False","Otros Gastos de Administración"
"l10n_bo_5401","Interest on Bank Loans","5401","expense","account.account_tag_operating","False","Intereses Sobre Préstamos Bancarios"
"l10n_bo_5402","Interest on Other Financial Obligations","5402","expense","account.account_tag_operating","False","Intereses Sobre Otras Obligaciones Financieras"
"l10n_bo_5403","Other Interests","5403","expense","account.account_tag_operating","False","Otros Intereses"
"l10n_bo_5404","Bank Fees","5404","expense","account.account_tag_operating","False","Comisiones Bancarias"
"l10n_bo_5405","Other Financial Expenses","5405","expense","account.account_tag_operating","False","Otros Gastos Financieros"
"l10n_bo_550","Other Operational Expenses","550","expense","account.account_tag_operating","False","Otros Gastos de Operación"
"l10n_bo_5601","Adjustment for Inflation and Property Ownership","5601","expense_depreciation","account.account_tag_operating","False","Ajuste por Inflación y Tenencie de Bienes"
"l10n_bo_5602","Exchange Loss","5602","expense","account.account_tag_operating","False","Pérdida Cambiaria"
"l10n_bo_5603","Other Expenses","5603","expense","account.account_tag_operating","False","Otros Gastos"
"l10n_bo_570","Corporate Income Tax","570","expense","account.account_tag_operating","False","Impuesto Sobre las Utilidades de las Empresas"

```

## File: data\template\account.fiscal.position-bo.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","name@es"
"l10n_bo_domestic","10","Domestic","1","1","base.bo","Nacional"
"l10n_bo_domestic_consumer","20","Domestic (Consumer)","1","","base.bo","Nacional (Consumidores)"
"l10n_bo_international","30","International","1","","","Internacional"

```

## File: data\template\account.group-bo.csv

```csv
"id","code_prefix_start","name","name@es"
"l10n_bo_group_1","1","Activos",""
"l10n_bo_group_11","11","Activos Corrientes",""
"l10n_bo_group_111","111","Efectivo y Equivalentes de Efectivo",""
"l10n_bo_group_112","112","Cuentas por Cobrar",""
"l10n_bo_group_113","113","Inventarios",""
"l10n_bo_group_114","114","Other Current Assets","Otros Activos Corrientes"
"l10n_bo_group_115","115","Inventarios",""
"l10n_bo_group_12","12","Activos no Corrientes",""
"l10n_bo_group_121","121","Cuentas por Cobrar a Largo Plazo",""
"l10n_bo_group_122","122","Inventarios no Corrientes",""
"l10n_bo_group_123","123","Propriedad Planta y Equipo",""
"l10n_bo_group_124","124","Investment Properties","Propriedades de Inversión"
"l10n_bo_group_125","125","Activos Intangibles",""
"l10n_bo_group_126","126","Permanent Investments","Inversiones Permanentes"
"l10n_bo_group_127","127","Otros Activos no Corrientes",""
"l10n_bo_group_2","2","Pasivos",""
"l10n_bo_group_21","21","Pasivos Corrientes",""
"l10n_bo_group_211","211","Obligaciones Bancarias y Financieras",""
"l10n_bo_group_212","212","Cuentas por Pagar",""
"l10n_bo_group_213","213","Obligaciones Sociales y Fiscales",""
"l10n_bo_group_214","214","Provisions","Provisiones"
"l10n_bo_group_215","215","Deferred Income","Ingresos Diferidos"
"l10n_bo_group_216","216","Other Current Liabilities","Otros Pasivos Corrientes"
"l10n_bo_group_22","22","Pasivos no Corrientes",""
"l10n_bo_group_221","221","Obligaciones Bancarias y Financieras a Largo Plazo",""
"l10n_bo_group_222","222","Cuentas por Pagar a Largo Plazo",""
"l10n_bo_group_223","223","Previsión para Beneficios Sociales",""
"l10n_bo_group_224","224","Otros Pasivos no Corrientes",""
"l10n_bo_group_3","3","Patrimonio",""
"l10n_bo_group_31","31","Capital",""
"l10n_bo_group_32","32","Reservas",""
"l10n_bo_group_33","33","Retained Earnings","Resultados Acumulados"
"l10n_bo_group_4","4","Ingresos",""
"l10n_bo_group_41","41","Ingresos Netos",""
"l10n_bo_group_42","42","Ingresos Financieros",""
"l10n_bo_group_43","43","Other Income","Otros Ingresos"
"l10n_bo_group_5","5","Gastos",""
"l10n_bo_group_51","51","Costo de Ventas",""
"l10n_bo_group_52","52","Gastos de Comercialización",""
"l10n_bo_group_53","53","Gastos Generales de Administración",""
"l10n_bo_group_54","54","Gastos Financieros",""
"l10n_bo_group_55","55","Other Operational Expenses","Otros Gastos de Operación"
"l10n_bo_group_56","56","Otros Gastos no Operativos",""

```

## File: data\template\account.tax-bo.csv

```csv
"id","sequence","name","description","invoice_label","amount","amount_type","type_tax_use","include_base_amount","price_include","tax_group_id","is_base_affected","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@es"
"l10n_bo_iva_13_sale","100","13%","IVA 13%","","13.0","division","sale","True","True","tax_group_iva_13","","","base","invoice","+IVA 13","",""
"","","","","","","","","","","","","","tax","invoice","+IVA 13||+IVA 19","l10n_bo_2135",""
"","","","","","","","","","","","","","base","refund","+IVA 27","",""
"","","","","","","","","","","","","","tax","refund","+IVA 27||+IVA 30","l10n_bo_2135",""
"l10n_bo_iva_13_discount_sale","105","13% D","IVA 13% - for Discounts","","13.0","division","sale","True","True","tax_group_iva_13","","","base","invoice","-IVA 28","","IVA 13% - por Descuentos"
"","","","","","","","","","","","","","tax","invoice","-IVA 28||-IVA 30","l10n_bo_2135",""
"","","","","","","","","","","","","","base","refund","-IVA 27","",""
"","","","","","","","","","","","","","tax","refund","-IVA 27||-IVA 19","l10n_bo_2135",""
"l10n_bo_iva_0_sale","110","0%","IVA 0% - Zero Rate","","0.0","percent","sale","","","tax_group_iva_0","","","base","invoice","+IVA 15","","IVA 0% - Tasa Cero"
"","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","base","refund","+IVA 27","",""
"","","","","","","","","","","","","","tax","refund","","",""
"l10n_bo_iva_0_exempt_sale","120","0% EXEMPT","0% IVA Exempt","","0.0","percent","sale","","","tax_group_iva_0_exempt","","","base","invoice","+IVA 14","","0% Exento de IVA"
"","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","base","refund","+IVA 27","",""
"","","","","","","","","","","","","","tax","refund","","",""
"l10n_bo_iva_0_untaxed_sale","130","0% N","0% Not Subject to IVA","","0.0","percent","sale","","","tax_group_iva_0_untaxed","","","base","invoice","+IVA 505","","0% No Sujeto al IVA"
"","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","base","refund","+IVA 27","",""
"","","","","","","","","","","","","","tax","refund","","",""
"l10n_bo_it_3_sale_tax","140","3% IT","IT 3% (tax line)","","3.0","percent","none","","","tax_group_it_3","True","","base","invoice","+IT 24","","IT 3% (tasa)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_2136",""
"","","","","","","","","","","","","","base","refund","-IT 24","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_2136",""
"l10n_bo_it_3_sale_expense","150","3% WH IT","IT 3% (expense line)","","-3.0","percent","none","","","tax_group_it_3","True","","base","invoice","","","IT 3% (gasto)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_52091",""
"","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_52091",""
"l10n_bo_it_3_sale","160","3% IT","IT 3%","","","group","sale","","","tax_group_it_3","","l10n_bo_it_3_sale_tax,l10n_bo_it_3_sale_expense","","","","",""
"l10n_bo_it_0_sale","170","0% N IT","0% Not Subject to IT","","0.0","percent","sale","","","tax_group_it_0","","","base","invoice","+IT 32","","0% No Sujeto al IT"
"","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","base","refund","-IT 32","",""
"","","","","","","","","","","","","","tax","refund","","",""
"l10n_bo_iva_13_purchase","300","13%","IVA 13%","","13.0","division","purchase","True","True","tax_group_iva_13","","","base","invoice","+IVA 11||+IVA 26","",""
"","","","","","","","","","","","","","tax","invoice","+IVA 11||+IVA 26||+IVA 30","l10n_bo_2135",""
"","","","","","","","","","","","","","base","refund","+IVA 17","",""
"","","","","","","","","","","","","","tax","refund","+IVA 17||+IVA 19","l10n_bo_2135",""
"l10n_bo_iva_13_discount_purchase","305","13% Disc","IVA 13% - for Discounts","","13.0","division","purchase","True","True","tax_group_iva_13","","","base","invoice","-IVA 28","","IVA 13% - por Descuentos"
"","","","","","","","","","","","","","tax","invoice","-IVA 28||-IVA 19","l10n_bo_2135",""
"","","","","","","","","","","","","","base","refund","-IVA 17","",""
"","","","","","","","","","","","","","tax","refund","-IVA 17||-IVA 30","l10n_bo_2135",""
"l10n_bo_iva_13_purchase_personal","310","13% P C","IVA 13% - for Private Consumption","","13.0","division","purchase","True","True","tax_group_iva_13","","","base","invoice","+IVA 11||+IVA 26||+IVA 16","","IVA 13% - por Consumos Particulares"
"","","","","","","","","","","","","","tax","invoice","+IVA 11||+IVA 26||+IVA 16||+IVA 30||+IVA 19","l10n_bo_2135",""
"","","","","","","","","","","","","","base","refund","+IVA 17","",""
"","","","","","","","","","","","","","tax","refund","+IVA 17","",""
"l10n_bo_iva_13_purchase_donations","320","13% D","IVA 13% - for Donations","","13.0","division","purchase","True","True","tax_group_iva_13","","","base","invoice","+IVA 11||+IVA 26","","IVA 13% - por Donaciones"
"","","","","","","","","","","","","","tax","invoice","+IVA 11||+IVA 26||+IVA 30||+IVA 19||+IVA 55","l10n_bo_2135",""
"","","","","","","","","","","","","","base","refund","+IVA 17","",""
"","","","","","","","","","","","","","tax","refund","+IVA 17","",""
"l10n_bo_iva_0_purchase","330","0%","IVA 0% - Zero Rate","","0.0","percent","purchase","","","tax_group_iva_0","","","base","invoice","+IVA 11","","IVA 0% - Tasa Cero"
"","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","base","refund","+IVA 17","",""
"","","","","","","","","","","","","","tax","refund","","",""
"l10n_bo_iva_mixed_purchase","340","13% M T U","IVA - Mixed Taxed and Untaxed","","13.0","division","purchase","True","True","tax_group_iva_mixed","","","base","invoice","+IVA 11||+IVA 31","","IVA - Mixto Gravado y no Gravado"
"","","","","","","","","","","","","","tax","invoice","+IVA 11||+IVA 31||+IVA 30","l10n_bo_2135",""
"","","","","","","","","","","","","","base","refund","+IVA 17","",""
"","","","","","","","","","","","","","tax","refund","+IVA 17||+IVA 19","l10n_bo_2135",""
"l10n_bo_iva_0_exempt_purchase","350","0% EXEMPT","0% IVA Exempt","","0.0","percent","purchase","","","tax_group_iva_0_exempt","","","base","invoice","+IVA 11","","0% Exento de IVA"
"","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","base","refund","+IVA 17","",""
"","","","","","","","","","","","","","tax","refund","","",""
"l10n_bo_iva_0_untaxed_purchase","360","0% N","0% Not Subject to IVA","","0.0","percent","purchase","","","tax_group_iva_0_untaxed","","","base","invoice","+IVA 11","","0% No Sujeto al IVA"
"","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","base","refund","+IVA 17","",""
"","","","","","","","","","","","","","tax","refund","","",""
"l10n_bo_whh_services_it_tax","370","3,5503% WHH IT S T","3% Whh. IT services gross-up (tax)","","3.5503","percent","none","","","tax_group_whh_services","","","base","invoice","","","3% Ret. IT servicios gross-up (tasa)"
"","","","","","","","","","","","","","tax","invoice","+Ret IT 13","l10n_bo_2137",""
"","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","tax","refund","-Ret IT 13","l10n_bo_2137",""
"l10n_bo_whh_services_it_expense","380","3,5503% WHH IT S","3% Whh. IT services gross-up (expense)","","-3.5503","percent","none","","","tax_group_whh_services","","","base","invoice","","","3% Ret. IT servicios gross-up (gasto)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_52092",""
"","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_52092",""
"l10n_bo_whh_services_iue_tax","390","14,7929% WHH IUE S","12.5% Whh. IUE services gross-up (tax)","","14.7929","percent","none","","","tax_group_whh_services","","","base","invoice","+Ret IUE 13","","12.5% Ret. IUE servicios gross-up (tasa)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_2138",""
"","","","","","","","","","","","","","base","refund","-Ret IUE 13","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_2138",""
"l10n_bo_whh_services_iue_expense","400","14,7929% WH WHH IUE S","12.5% Whh. IUE services gross-up (expense)","","-14.7929","percent","none","","","tax_group_whh_services","","","base","invoice","","","12.5% Ret. IUE servicios gross-up (gasto)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_52093",""
"","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_52093",""
"l10n_bo_whh_services_purchase","410","15.5% Whh. IT+IUE S","15.5% Whh. IT+IUE services gross-up","","","group","purchase","","","tax_group_whh_services","","l10n_bo_whh_services_it_tax,l10n_bo_whh_services_it_expense,l10n_bo_whh_services_iue_tax,l10n_bo_whh_services_iue_expense","","","","","15.5% Ret. IT+IUE servicios gross-up"
"l10n_bo_whh_goods_it_tax","420","3,2609%% WHH IT G","3% Whh. IT goods gross-up (tax)","","3.2609","percent","none","","","tax_group_whh_goods","","","base","invoice","","","3% Ret. IT bienes gross-up (tasa)"
"","","","","","","","","","","","","","tax","invoice","+Ret IT 13","l10n_bo_2137",""
"","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","tax","refund","-Ret IT 13","l10n_bo_2137",""
"l10n_bo_whh_goods_it_expense","430","3% WH WHH IT G","3% Whh. IT goods gross-up (expense)","","-3.2609","percent","none","","","tax_group_whh_goods","","","base","invoice","","","3% Ret. IT bienes gross-up (gasto)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_52092",""
"","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_52092",""
"l10n_bo_whh_goods_iue_tax","440","5,4348% WHH IUE G","5% Whh. IUE goods gross-up (tax)","","5.4348","percent","none","","","tax_group_whh_goods","","","base","invoice","+Ret IUE 26","","5% Ret. IUE bienes gross-up (tasa)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_2138",""
"","","","","","","","","","","","","","base","refund","-Ret IUE 26","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_2138",""
"l10n_bo_whh_goods_iue_expense","450","5,4348% WH WHH IUE G","5% Whh. IUE goods gross-up (expense)","","-5.4348","percent","none","","","tax_group_whh_goods","","","base","invoice","","","5% Ret. IUE bienes gross-up (gasto)"
"","","","","","","","","","","","","","tax","invoice","","l10n_bo_52093",""
"","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","tax","refund","","l10n_bo_52093",""
"l10n_bo_whh_goods_purchase","460","8% WHH IT+IUE G","8% Whh. IT+IUE goods gross-up","","","group","purchase","","","tax_group_whh_goods","","l10n_bo_whh_goods_it_tax,l10n_bo_whh_goods_it_expense,l10n_bo_whh_goods_iue_tax,l10n_bo_whh_goods_iue_expense","","","","","8% Ret. IT+IUE bienes gross-up"

```

## File: data\template\account.tax.group-bo.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","advance_tax_payment_account_id","name@es"
"tax_group_iva_13","IVA 13%","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142",""
"tax_group_iva_0","IVA 0%","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142",""
"tax_group_iva_mixed","IVA Mixed","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142","IVA Mixto"
"tax_group_iva_0_exempt","0% IVA Exempt","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142","0% Exento de IVA"
"tax_group_iva_0_untaxed","0% Not Subject to IVA","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142","0% No Sujeto al IVA"
"tax_group_it_3","IT 3%","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142",""
"tax_group_it_0","0% Not Subject to IT","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142","0% No Sujeto al IT"
"tax_group_whh_services","Withholdings for services","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142","Retenciones por servicios"
"tax_group_whh_goods","Withholdings for goods","base.bo","l10n_bo_1141","l10n_bo_2135","l10n_bo_1142","Retenciones por bienes"

```

## File: models\template_bo.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bo')
    def _get_bo_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'l10n_bo_1121',
            'property_account_payable_id': 'l10n_bo_2121',
            'property_account_expense_categ_id': 'l10n_bo_53008',
            'property_account_income_categ_id': 'l10n_bo_4101',
            'property_stock_account_input_categ_id': 'l10n_bo_11341',
            'property_stock_account_output_categ_id': 'l10n_bo_11342',
            'property_stock_valuation_account_id': 'l10n_bo_1131',
        }

    @template('bo', 'res.company')
    def _get_bo_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.bo',
                'bank_account_code_prefix': '11130',
                'cash_account_code_prefix': '11110',
                'transfer_account_code_prefix': '11110',
                'account_default_pos_receivable_account_id': 'l10n_bo_11211',
                'income_currency_exchange_account_id': 'l10n_bo_4303',
                'expense_currency_exchange_account_id': 'l10n_bo_5602',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_bo_5104',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_bo_4102',
                'default_cash_difference_income_account_id': 'l10n_bo_4301',
                'default_cash_difference_expense_account_id': 'l10n_bo_5601',
                'account_sale_tax_id': 'l10n_bo_iva_13_sale',
                'account_purchase_tax_id': 'l10n_bo_iva_13_purchase',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_bo

```

