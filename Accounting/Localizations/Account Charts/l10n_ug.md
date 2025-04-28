# Odoo Module: l10n_ug

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    "name": "Uganda - Accounting",
    "countries": ["ug"],
    "version": "1.0.0",
    "category": "Accounting/Localizations/Account Charts",
    "license": "LGPL-3",
    "description": """
This is the basic Ugandian localisation necessary to run Odoo in UG:
================================================================================
    - Chart of accounts
    - Taxes
    - Fiscal positions
    - Default settings
    - Tax report
    """,
    "depends": [
        "account",
    ],
    'auto_install': ['account'],
    "data": [
        "data/account_tax_report_data.xml",
    ],
    "demo": [
        "demo/demo_company.xml",
    ]
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="section_CD" model="account.report">
        <field name="name">Sections C and D</field>
        <field name="country_id" ref="base.ug"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="section_CD_base" model="account.report.column">
                <field name="name">Amounts</field>
                <field name="expression_label">base</field>
            </record>
            <record id="section_CD_tax" model="account.report.column">
                <field name="name">VAT Charged</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="C" model="account.report.line">
                <field name="name">Section C - Sales (Goods and Services)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="C1" model="account.report.line">
                        <field name="name">1. Zero Rated Sales Local</field>
                        <field name="code">UG_TAX_C1</field>
                        <field name="expression_ids">
                            <record id="C1_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C1_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="C2" model="account.report.line">
                        <field name="name">2. Zero Rated Sales Export</field>
                        <field name="code">UG_TAX_C2</field>
                        <field name="expression_ids">
                            <record id="C2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C2_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="C3" model="account.report.line">
                        <field name="name">3. Exempt Local Sales</field>
                        <field name="code">UG_TAX_C3</field>
                        <field name="expression_ids">
                            <record id="C3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C3_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="C4" model="account.report.line">
                        <field name="name">4. Standard Rated Sales Charged</field>
                        <field name="code">UG_TAX_C4</field>
                        <field name="expression_ids">
                            <record id="C4_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C4_base</field>
                            </record>
                            <record id="C4_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C4_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="C5" model="account.report.line">
                        <field name="name">5. Standard Rated Sales Deemed</field>
                        <field name="code">UG_TAX_C5</field>
                        <field name="expression_ids">
                        <record id="C5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C5_base</field>
                            </record>
                            <record id="C5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C5_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="C6" model="account.report.line">
                        <field name="name">6. Capital goods sold (Business Assets) Charged</field>
                        <field name="code">UG_TAX_C6</field>
                        <field name="expression_ids">
                            <record id="C6_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C6_base</field>
                            </record>
                            <record id="C6_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C6_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="C7" model="account.report.line">
                        <field name="name">7. Capital goods sold (Business Assets) Deemed</field>
                        <field name="code">UG_TAX_C7</field>
                        <field name="expression_ids">
                            <record id="C7_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C7_base</field>
                            </record>
                            <record id="C7_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C7_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="C8" model="account.report.line">
                        <field name="name">8. Total Output tax</field>
                        <field name="code">UG_TAX_C8</field>
                        <field name="expression_ids">
                            <record id="C8_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_C4.tax + UG_TAX_C6.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="C9" model="account.report.line">
                        <field name="name">9. Adjustments to Output tax</field>
                        <field name="children_ids">
                            <record id="C9_i" model="account.report.line">
                                <field name="name">i. Imported Services</field>
                                <field name="code">UG_TAX_C9_i</field>
                                <field name="expression_ids">
                                    <record id="C9_i_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">C9_i_base</field>
                                    </record>
                                    <record id="C9_i_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">C9_i_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="C9_ii" model="account.report.line">
                                <field name="name">ii. VAT deferred at Importation</field>
                                <field name="code">UG_TAX_C9_ii</field>
                                <field name="expression_ids">
                                    <record id="C9_ii_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">C9_ii_base</field>
                                    </record>
                                    <record id="C9_ii_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">C9_ii_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="C9_iii" model="account.report.line">
                                <field name="name">iii. Tax Charge due to bad debts recovered</field>
                                <field name="code">UG_TAX_C9_iii</field>
                                <field name="expression_ids">
                                    <record id="C9_iii_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="C9_iv" model="account.report.line">
                                <field name="name">iv. Tax Charge due to change of Accounting method</field>
                                <field name="code">UG_TAX_C9_iv</field>
                                <field name="expression_ids">
                                    <record id="C9_iv_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="C9_v" model="account.report.line">
                                <field name="name">v. Tax Charge due to end of year apportionment of input tax</field>
                                <field name="code">UG_TAX_C9_v</field>
                                <field name="expression_ids">
                                    <record id="C9_v_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="C10" model="account.report.line">
                        <field name="name">10. Total Tax Charged for the Period</field>
                        <field name="code">UG_TAX_C10</field>
                        <field name="expression_ids">
                            <record id="C10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_C8.tax + UG_TAX_C9_i.tax + UG_TAX_C9_ii.tax + UG_TAX_C9_iii.tax + UG_TAX_C9_iv.tax + UG_TAX_C9_v.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="D" model="account.report.line">
                <field name="name">Section D - Purchases (Goods and Services)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="D11" model="account.report.line">
                        <field name="name">11. Zero Rated Purchases Local</field>
                        <field name="code">UG_TAX_D11</field>
                        <field name="expression_ids">
                            <record id="D11_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D11_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="D12" model="account.report.line">
                        <field name="name">12. Zero Rated Purchases Export</field>
                        <field name="code">UG_TAX_D12</field>
                        <field name="expression_ids">
                            <record id="D12_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D12_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="D13" model="account.report.line">
                        <field name="name">13. Standard Rated Purchases Local VAT Incurred</field>
                        <field name="code">UG_TAX_D13</field>
                        <field name="expression_ids">
                            <record id="D13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D13_base</field>
                            </record>
                            <record id="D13_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D13_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D14" model="account.report.line">
                        <field name="name">14. Standard Rated Purchases Local VAT Deemed</field>
                        <field name="code">UG_TAX_D14</field>
                        <field name="expression_ids">
                            <record id="D14_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D14_base</field>
                            </record>
                            <record id="D14_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D14_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D15" model="account.report.line">
                        <field name="name">15. Standard Rated Imports (Goods) VAT Incurred</field>
                        <field name="code">UG_TAX_D15</field>
                        <field name="expression_ids">
                            <record id="D15_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D15_base</field>
                            </record>
                            <record id="D15_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D15_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D16" model="account.report.line">
                        <field name="name">16. Standard Rated Imports (Goods) VAT Deferred</field>
                        <field name="code">UG_TAX_D16</field>
                        <field name="expression_ids">
                            <record id="D16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D16_base</field>
                            </record>
                            <record id="D16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D16_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D17" model="account.report.line">
                        <field name="name">17. Administrative Expenses VAT Incurred</field>
                        <field name="code">UG_TAX_D17</field>
                        <field name="expression_ids">
                            <record id="D17_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D17_base</field>
                            </record>
                            <record id="D17_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D17_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D18" model="account.report.line">
                        <field name="name">18. Capital goods bought (Business Assets) VAT Incurred</field>
                        <field name="code">UG_TAX_D18</field>
                        <field name="expression_ids">
                            <record id="D18_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D18_base</field>
                            </record>
                            <record id="D18_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D18_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D19" model="account.report.line">
                        <field name="name">19. Capital goods bought (Business Assets) VAT Deemed</field>
                        <field name="code">UG_TAX_D19</field>
                        <field name="expression_ids">
                        <record id="D19_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D19_base</field>
                            </record>
                            <record id="D19_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D19_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D20" model="account.report.line">
                        <field name="name">20. Total Input tax</field>
                        <field name="code">UG_TAX_D20</field>
                        <field name="expression_ids">
                            <record id="D20_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_D13.tax + UG_TAX_D15.tax + UG_TAX_D17.tax + UG_TAX_D18.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="D21" model="account.report.line">
                        <field name="name">21. Adjustment of input tax</field>
                        <field name="children_ids">
                            <record id="D21_i" model="account.report.line">
                                <field name="name">i. Imported Services</field>
                                <field name="code">UG_TAX_D21_i</field>
                                <field name="expression_ids">
                                    <record id="D21_i_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">D21_i_base</field>
                                    </record>
                                    <record id="D21_i_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">D21_i_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="D21_ii" model="account.report.line">
                                <field name="name">ii. Deferred VAT discharged (permitted by Commissioner General)</field>
                                <field name="code">UG_TAX_D21_ii</field>
                                <field name="expression_ids">
                                    <record id="D21_ii_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                    <record id="D21_ii_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">UG_TAX_D21_ii.base * 0.18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="D21_iii" model="account.report.line">
                                <field name="name">iii. Tax Claim due to bad debts written off</field>
                                <field name="code">UG_TAX_D21_iii</field>
                                <field name="expression_ids">
                                    <record id="D21_iii_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="D21_iv" model="account.report.line">
                                <field name="name">iv. Tax Credit due to change of Accounting method</field>
                                <field name="code">UG_TAX_D21_iv</field>
                                <field name="expression_ids">
                                    <record id="D21_iv_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="D22" model="account.report.line">
                        <field name="name">22. Total Input Tax for the Period</field>
                        <field name="code">UG_TAX_D22</field>
                        <field name="expression_ids">
                            <record id="D22_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_D20.tax + UG_TAX_D21_i.tax - UG_TAX_D21_ii.tax + UG_TAX_D21_iii.tax + UG_TAX_D21_iv.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>

    <record id="section_F" model="account.report">
        <field name="name">Section F</field>
        <field name="country_id" ref="base.ug"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="section_F_base" model="account.report.column">
                <field name="name">Input tax disallowed</field>
                <field name="expression_label">disallowed</field>
            </record>
            <record id="section_F_tax" model="account.report.column">
                <field name="name">Input tax credit Allowed</field>
                <field name="expression_label">allowed</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="F" model="account.report.line">
                <field name="name">Section F - Calculation Input tax Credit Allowed</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="FM" model="account.report.line">
                        <field name="name">Use Standard Alternative Method of Apportionment (Default is Normal method)?</field>
                        <field name="code">UG_TAX_FM</field>
                        <field name="expression_ids">
                            <record id="FM_tax" model="account.report.expression">
                                <field name="label">allowed</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable</field>
                                <field name="figure_type">boolean</field>
                            </record>
                        </field>
                    </record>
                    <record id="F29" model="account.report.line">
                        <field name="name">29. Input tax directly attributable to Taxable Sales</field>
                        <field name="code">UG_TAX_F29</field>
                        <field name="expression_ids">
                            <record id="F29_tax" model="account.report.expression">
                                <field name="label">allowed</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="F30" model="account.report.line">
                        <field name="name">30. Input tax directly attributable to Exempt Sales (Disallowed) and non-creditable input tax</field>
                        <field name="code">UG_TAX_F30</field>
                        <field name="expression_ids">
                            <record id="F30_base" model="account.report.expression">
                                <field name="label">disallowed</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="F31" model="account.report.line">
                        <field name="name">31.Input tax apportioned to Taxable Sales using selected formula</field>
                        <field name="code">UG_TAX_F31</field>
                        <field name="expression_ids">
                            <record id="F31_tax" model="account.report.expression">
                                <field name="label">allowed</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    (1 - UG_TAX_FM.allowed) * UG_TAX_F31.hidden_computation_of_F31_using_normal_formula
                                    + UG_TAX_FM.allowed * UG_TAX_F31.hidden_computation_of_F31_using_sam_formula
                                </field>
                            </record>
                            <record id="F31_normal_tax" model="account.report.expression">
                                <field name="label">hidden_computation_of_F31_using_normal_formula</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_F31.normal_formula_part1 * UG_TAX_F31.normal_formula_part2 / UG_TAX_F31.normal_formula_part3</field>
                                <field name="subformula">ignore_zero_division</field>
                            </record>
                            <record id="F31_normal_tax_part1" model="account.report.expression">
                                <field name="label">normal_formula_part1</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_D20.tax - UG_TAX_F30.disallowed</field>
                                <field name="subformula">cross_report</field>
                            </record>
                            <record id="F31_normal_tax_part2" model="account.report.expression">
                                <field name="label">normal_formula_part2</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_C1.base + UG_TAX_C2.base + UG_TAX_C4.base + UG_TAX_C5.base + UG_TAX_C6.base + UG_TAX_C7.base</field>
                                <field name="subformula">cross_report</field>
                            </record>
                            <record id="F31_normal_tax_part3" model="account.report.expression">
                                <field name="label">normal_formula_part3</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_C1.base + UG_TAX_C2.base + UG_TAX_C3.base + UG_TAX_C4.base + UG_TAX_C5.base + UG_TAX_C6.base + UG_TAX_C7.base</field>
                                <field name="subformula">cross_report</field>
                            </record>
                            <record id="F31_sam_tax" model="account.report.expression">
                                <field name="label">hidden_computation_of_F31_using_sam_formula</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_F31.sam_formula_part1 * UG_TAX_F31.sam_formula_part2 / UG_TAX_F31.sam_formula_part3</field>
                                <field name="subformula">ignore_zero_division</field>
                            </record>
                            <record id="F31_sam_tax_part1" model="account.report.expression">
                                <field name="label">sam_formula_part1</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_D20.tax - UG_TAX_F29.allowed - UG_TAX_F30.disallowed</field>
                                <field name="subformula">cross_report</field>
                            </record>
                            <record id="F31_sam_tax_part2" model="account.report.expression">
                                <field name="label">sam_formula_part2</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_C1.base + UG_TAX_C2.base + UG_TAX_C4.base + UG_TAX_C5.base + UG_TAX_C6.base + UG_TAX_C7.base</field>
                                <field name="subformula">cross_report</field>
                            </record>
                            <record id="F31_sam_tax_part3" model="account.report.expression">
                                <field name="label">sam_formula_part3</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_C1.base + UG_TAX_C2.base + UG_TAX_C3.base + UG_TAX_C4.base + UG_TAX_C5.base + UG_TAX_C6.base + UG_TAX_C7.base</field>
                                <field name="subformula">cross_report</field>
                            </record>
                        </field>
                    </record>
                    <record id="F32" model="account.report.line">
                        <field name="name">32. Input tax credit for the period</field>
                        <field name="code">UG_TAX_F32</field>
                        <field name="expression_ids">
                            <record id="F32_tax" model="account.report.expression">
                                <field name="label">allowed</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_F29.allowed + UG_TAX_F31.allowed</field>
                            </record>
                        </field>
                    </record>
                    <record id="F33" model="account.report.line">
                        <field name="name">33. Adjustments to Input tax credit</field>
                        <field name="code">UG_TAX_F33</field>
                        <field name="children_ids">
                            <record id="F33_a" model="account.report.line">
                                <field name="name">a. Adjustments to Input tax credit</field>
                                <field name="code">UG_TAX_F33_a</field>
                                <field name="expression_ids">
                                    <record id="F33_a_tax" model="account.report.expression">
                                        <field name="label">allowed</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="F34" model="account.report.line">
                        <field name="name">Total Input tax credit Allowed for the period</field>
                        <field name="expression_ids">
                            <record id="F34_tax" model="account.report.expression">
                                <field name="label">allowed</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">UG_TAX_F32.allowed + UG_TAX_F33_a.allowed + UG_TAX_D21_i.tax + UG_TAX_D21_iii.tax + UG_TAX_D21_iv.tax</field>
                                <field name="subformula">cross_report</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>

    <record id="tax_report_ug" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ug"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="use_sections" eval="True"/>
        <field name="section_report_ids" eval="[Command.set([ref('section_CD'), ref('section_F')])]"/>
    </record>
</odoo>

```

## File: data\template\account.account-ug.csv

```csv
id,name,code,account_type,tag_ids,reconcile
1410,Property Income,1410,income_other,,False
1420,Sale of goods and services,1420,income,,False
14211,Rent & rates - produced assets,14211,income,,False
14212,Sale of produced assets,14212,income,,False
1440,Transfers,1440,income_other,,False
14511,Premiums receivable,14511,income_other,,False
14513,Current claims receivable,14513,income_other,,False
1452,Capital Claims receivable,1452,income_other,,False
191001,Clearing Account - Discount Taken,191001,income,,False
191002,Clearing Account - Unallocated Revenue,191002,income,,False
191003,Clearing Account - Income Offset Account,191003,income,,False
2111,Wages and Salaries - Cash,2111,expense,,False
2112,Wages and salaries - in kind,2112,expense,,False
2121,Employers' Social Contributions-Actual,2121,expense,,False
2122,Employer's Social Contributions-Imputed,2122,expense,,False
2210,General use of goods and services,2210,expense,,False
221018,Exchange losses/gains,221018,expense,,False
221019,Discounts Allowed,221019,expense,,False
2220,Communications,2220,expense,,False
2230,Utility and Property Expenses,2230,expense,,False
2240,Supplies and Services,2240,expense,,False
2251,Consultancy Services- Recurrent,2251,expense,,False
2252,Consultancy Services- Capital,2252,expense,,False
2260,Insurances and Licenses,2260,expense,,False
2270,Travel and Transport,2270,expense,,False
2280,Maintenance,2280,expense,,False
2291,Net change in inventories,2291,expense,,False
2292,Sale of goods purchased for resale,2292,expense,,False
2311,Depreciation of buildings and structures,2311,expense_depreciation,,False
2312,Depreciation of machinery and equipments,2312,expense_depreciation,,False
2314,Depreciation of other assets,2314,expense_depreciation,,False
2634,Other Transfers,2634,expense,,False
2711,Social security benefits in cash,2711,expense,,False
2712,Social security benefits in kind,2712,expense,,False
2721,Social assistance benefits in cash,2721,expense,,False
2722,Social assistance benefits in kind,2722,expense,,False
2731,Employment-related social benefits in cash,2731,expense,,False
2732,Employment-related social benefits in in kind,2732,expense,,False
2811,Dividends,2811,expense,,False
2812,Withdrawals from income of quasi - corporations,2812,expense,,False
2813,Property expense for investment income disbursements,2813,expense,,False
2814,Rent,2814,expense,,False
2815,Reinvested earnings on foreign direct investment,2815,expense,,False
2821,Current transfers not elsewhere classified,2821,expense,,False
2822,Capital transfers not elsewhere classified,2822,expense,,False
2823,Tax expenditures,2823,expense,,False
2831,"Premiums, fees and current claims payable",2831,expense,,False
2832,Capital claims payable,2832,expense,,False
291001,Clearing Account - Bank Charges,291001,expense,,False
291002,Clearing Account - Exchange Losses/Gains,291002,expense,,False
291003,Clearing Account - Discounts Allowed,291003,expense,,False
291004,Clearing Account - Purchase Price Variance,291004,expense,,False
291005,Clearing Account - Bank Errors,291005,expense,,False
291006,Clearing Account - Cost of Goods Sold,291006,expense,,False
310111,Residential Buildings,310111,asset_fixed,,False
310119,Other Dwellings,310119,asset_fixed,,False
310121,Non-Residential Buildings,310121,asset_fixed,,False
310129,Other Buildings other than dwellings,310129,asset_fixed,,False
31013,Structures,31013,asset_fixed,,False
31014,Land Improvements,31014,asset_fixed,,False
31021,Transport equipment,31021,asset_fixed,,False
31022,ICT equipment,31022,asset_fixed,,False
31023,Machinery and other equipment,31023,asset_fixed,,False
31031,Classified assets,31031,asset_fixed,,False
31041,Biological assets,31041,asset_fixed,,False
31042,Intellectual property product,31042,asset_fixed,,False
31043,Intellectual property rights,31043,asset_fixed,,False
31044,Marketing assets,31044,asset_fixed,,False
320111,Inventory for materials and supplies,320111,asset_prepayments,,False
320112,Inventory for work in progress,320112,asset_prepayments,,False
320113,Inventory for finished goods,320113,asset_prepayments,,False
320114,Inventory for goods for resale,320114,asset_prepayments,,False
320115,Military Inventories,320115,asset_prepayments,,False
320119,Other Inventories,320119,asset_prepayments,,False
340111,Land,340111,asset_fixed,,False
340211,Minerals,340211,asset_fixed,,False
340212,Oil & Natural Gas,340212,asset_fixed,,False
340213,Energy resources,340213,asset_fixed,,False
340219,Other Mineral and Energy Resources,340219,asset_fixed,,False
34031,Non-Cultivated Biological resources,34031,asset_fixed,,False
34032,Water resources,34032,asset_fixed,,False
34033,Airspace resources,34033,asset_fixed,,False
34034,Non-Cultivated Non Biological,34034,asset_fixed,,False
340401,"Marketable operating leases - Contracts, leases and Permits",340401,asset_non_current,,False
340402,"Permits to use natural resources - Contracts, leases and Permits",340402,asset_non_current,,False
340403,"Permits to undertake specific activities - Contracts, leases and Permits",340403,asset_non_current,,False
3510,Monetary Gold and SDRs,3510,asset_current,,False
3522,Debt Securities,3522,asset_current,,False
3523,Loans,3523,asset_current,,False
352401,Shares in public corporations,352401,asset_non_current,,False
352402,Shares in private entities,352402,asset_non_current,,False
352501,Petroleum Revenue Investment Reserve,352501,asset_non_current,,False
352599,Other Investment Fund Shares or Units,352599,asset_non_current,,False
352701,Forwards,352701,asset_prepayments,,False
352702,Futures,352702,asset_prepayments,,False
352703,Options,352703,asset_prepayments,,False
352704,Swaps,352704,asset_prepayments,,False
3528,Account Receivable,3528,asset_receivable,,True
352802,Staff Advances,352802,asset_receivable,,True
352804,Taxes Receivable,352804,asset_current,,True
352805,Revenue receivable,352805,asset_receivable,,True
352806,Trade debtors,352806,asset_receivable,,True
352807,Sundry Debtors,352807,asset_receivable,,True
352808,Pre-payments,352808,asset_receivable,,True
352809,Deferred Expenses,352809,asset_receivable,,True
3529,Taxes owed to state,3529,asset_current,,False
4111,Currency Deposits,4111,liability_payable,,True
4112,Dept Security,411201,liability_current,,False
4113,Loans,4113,liability_current,,False
4115,Investment Fund Shares or Units,4115,liability_non_current,,False
4117,Accounts Payable,4117,liability_payable,,True
411721,Trade creditors,411721,liability_payable,,True
411722,Taxes payable,411722,liability_current,,True
411723,Taxes due to state,411723,liability_current,,False
411724,Deposits received,411724,liability_payable,,True
411725,Advances from Other Government Units,411725,liability_payable,,True
411726,Deferred Income,411726,liability_payable,,True
411798,Other Payables,411798,liability_payable,,True
4131,Provisions,4131,liability_non_current,,False
511001,Revenue Reserves,511001,equity_unaffected,,False
512001,Fixed Assets Reserves,512001,equity,,False
512201,Inventory Reserves,512201,equity,,False
512301,Financial Assets Reserves,512301,equity,,False
513001,Accumulated Fund,513001,equity,,False

```

## File: data\template\account.fiscal.position-ug.csv

```csv
"id","sequence","name","auto_apply","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_national","1","National","1","base.ug","",""
"fiscal_position_template_non_ugandian","2","International","1","","sale_zero_rated","sale_export_0"
"","","","","","sale_exempt","sale_export_0"
"","","","","","sale_vat_18","sale_export_0"
"","","","","","sale_deemed_vat_18","sale_export_0"
"","","","","","sale_capital_goods_18","sale_export_0"
"","","","","","sale_deemed_capital_goods_18","sale_export_0"
"","","","","","purchase_zero_rated","purchase_import_vat_18"
"","","","","","purchase_vat_18","purchase_import_vat_18"
"","","","","","purchase_deemed_vat_18","purchase_import_vat_18"
"","","","","","purchase_capital_goods_18","purchase_import_vat_18"
"","","","","","purchase_deemed_capital_goods_18","purchase_import_vat_18"
"","","","","","purchase_admin_exp_18","purchase_import_vat_18"

```

## File: data\template\account.tax-ug.csv

```csv
"id","name","description","amount","type_tax_use",tax_scope,"tax_group_id","repartition_line_ids/document_type","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","active"
"sale_zero_rated","0%","Zero-rated supplies.","0","sale",,"consumption_taxes","invoice","","base",,"+C1_base","True"
,,,,,,,"invoice","","tax",,,
,,,,,,,"refund","","base",,"-C1_base",
,,,,,,,"refund","","tax",,,
"purchase_zero_rated","0%","Zero-rated supplies.","0","purchase",,"consumption_taxes","invoice","","base",,"+D11_base","True"
,,,,,,,"invoice","","tax",,,
,,,,,,,"refund","","base",,"-D11_base",
,,,,,,,"refund","","tax",,,
"sale_export_0","0% EX","Exported supplies attract a zero rate of tax.","0","sale",,"consumption_taxes","invoice","","base",,"+C2_base","True"
,,,,,,,"invoice","","tax",,,
,,,,,,,"refund","","base",,"-C2_base",
,,,,,,,"refund","","tax",,,
"sale_exempt","Tax Exempt","Supplies exempts of tax.","0","sale",,"consumption_taxes","invoice","","base",,"+C3_base","True"
,,,,,,,"invoice","","tax",,,
,,,,,,,"refund","","base",,"-C3_base",
,,,,,,,"refund","","tax",,,
"sale_vat_18","18%","VAT on all supplies made by taxable persons.","18","sale",,"consumption_taxes","invoice","","base",,"+C4_base","True"
,,,,,,,"invoice","","tax","411722","+C4_tax",
,,,,,,,"refund","","base",,"-C4_base",
,,,,,,,"refund","","tax","411722","-C4_tax",
"purchase_vat_18","18%","VAT on all supplies made by taxable persons.","18","purchase",,"consumption_taxes","invoice","","base",,"+D13_base","True"
,,,,,,,"invoice","","tax","352804","+D13_tax",
,,,,,,,"refund","","base",,"-D13_base",
,,,,,,,"refund","","tax","352804","-D13_tax",
"sale_deemed_vat_18","18% Deemed","Deemed VAT on all supplies made by taxable persons.","18","sale",,"consumption_taxes","invoice","","base",,"+C5_base","False"
,,,,,,,"invoice","","tax","411722","+C5_tax",
,,,,,,,"invoice","-100","tax","411722",,
,,,,,,,"refund","","base",,"-C5_base",
,,,,,,,"refund","","tax","411722","-C5_tax",
,,,,,,,"refund","-100","tax","411722",,
"purchase_deemed_vat_18","18% Deemed","Deemed VAT on all supplies made by taxable persons.","18","purchase",,"consumption_taxes","invoice","","base",,"+D14_base","False"
,,,,,,,"invoice","","tax","352804","+D14_tax",
,,,,,,,"invoice","-100","tax","352804",,
,,,,,,,"refund","","base",,"-D14_base",
,,,,,,,"refund","","tax","352804","-D14_tax",
,,,,,,,"refund","-100","tax","352804",,
"sale_capital_goods_18","18% CG","Tax on all capital goods (business assets) sold.","18","sale",consu,"consumption_taxes","invoice","","base",,"+C6_base","True"
,,,,,,,"invoice","","tax","411722","+C6_tax",
,,,,,,,"refund","","base",,"-C6_base",
,,,,,,,"refund","","tax","411722","-C6_tax",
"purchase_capital_goods_18","18% CG","Tax on all capital goods (business assets) purchased.","18","purchase",consu,"consumption_taxes","invoice","","base",,"+D18_base","True"
,,,,,,,"invoice","","tax","352804","+D18_tax",
,,,,,,,"refund","","base",,"-D18_base",
,,,,,,,"refund","","tax","352804","-D18_tax",
"sale_deemed_capital_goods_18","18% CG Deemed","Deemed tax on all capital goods (business assets) sold.","18","sale",consu,"consumption_taxes","invoice","","base",,"+C7_base","False"
,,,,,,,"invoice","","tax","411722","+C7_tax",
,,,,,,,"invoice","-100","tax","411722",,
,,,,,,,"refund","","base",,"-C7_base",
,,,,,,,"refund","","tax","411722","-C7_tax",
,,,,,,,"refund","-100","tax","411722",,
"purchase_deemed_capital_goods_18","18% CG Deemed","Deemed tax on all capital goods (business assets) purchased.","18","purchase",consu,"consumption_taxes","invoice","","base",,"+D19_base","False"
,,,,,,,"invoice","","tax","352804","+D19_tax",
,,,,,,,"invoice","-100","tax","352804",,
,,,,,,,"refund","","base",,"-D19_base",
,,,,,,,"refund","","tax","352804","-D19_tax",
,,,,,,,"refund","-100","tax","352804",,
"sale_import_services_18","18% IS","Tax on imported services sold.","18","sale",service,"consumption_taxes","invoice","","base",,"+C9_i_base","True"
,,,,,,,"invoice","","tax","411722","+C9_i_tax",
,,,,,,,"refund","","base",,"-C9_i_base",
,,,,,,,"refund","","tax","411722","-C9_i_tax",
"purchase_import_services_18","18% IS","Tax on imported services sold.","18","purchase",service,"consumption_taxes","invoice","","base",,"+D21_i_base","True"
,,,,,,,"invoice","","tax","352804","+D21_i_tax",
,,,,,,,"refund","","base",,"-D21_i_base",
,,,,,,,"refund","","tax","352804","-D21_i_tax",
"sale_import_deferred_vat_18","18% Import Deferred","VAT deferred at importation","18","sale",,"consumption_taxes","invoice","","base",,"+C9_ii_base","False"
,,,,,,,"invoice","","tax","411722","+C9_ii_tax",
,,,,,,,"refund","","base",,"-C9_ii_base",
,,,,,,,"refund","","tax","411722","-C9_ii_tax",
"purchase_import_deferred_vat_18","18% Import Deferred","VAT deferred on imported goods.","18","purchase",consu,"consumption_taxes","invoice","","base",,"+D16_base","False"
,,,,,,,"invoice","","tax","352804","+D16_tax",
,,,,,,,"refund","","base",,"-D16_base",
,,,,,,,"refund","","tax","352804","-D16_tax",
"purchase_import_vat_18","18% Import","Tax on imported goods.","18","purchase",consu,"consumption_taxes","invoice","","base",,"+D15_base","True"
,,,,,,,"invoice","","tax","352804","+D15_tax",
,,,,,,,"refund","","base",,"-D15_base",
,,,,,,,"refund","","tax","352804","-D15_tax",
"purchase_admin_exp_18","18% AE","Administrative Expenses.","18","purchase",,"consumption_taxes","invoice","","base",,"+D17_base","True"
,,,,,,,"invoice","","tax","352804","+D17_tax",
,,,,,,,"refund","","base",,"-D17_base",
,,,,,,,"refund","","tax","352804","-D17_tax",

```

## File: data\template\account.tax.group-ug.csv

```csv
id,name,country_id,tax_payable_account_id,tax_receivable_account_id
consumption_taxes,Consumption Taxes,base.ug,"411723","3529"

```

## File: models\template_ug.py

```python
# -*- coding: utf-8 -*-

from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = "account.chart.template"

    @template('ug')
    def _get_ug_template_data(self):
        return {
            'name': "Uganda Generic Chart of Accounts",
            'code_digits': 6,
            'property_account_receivable_id': '3528',
            'property_account_payable_id': '4117',
            'property_account_expense_categ_id': '2240',
            'property_account_income_categ_id': '1420',
        }

    @template('ug', 'res.company')
    def _get_ug_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.ug',
                'bank_account_code_prefix': '3528',
                'cash_account_code_prefix': '3528',
                'transfer_account_code_prefix': '3528',
                'account_default_pos_receivable_account_id': '3528',
                'income_currency_exchange_account_id': '221018',
                'expense_currency_exchange_account_id': '221018',
                'account_journal_early_pay_discount_loss_account_id': '221019',
                'account_journal_early_pay_discount_gain_account_id': '191001',
                'account_sale_tax_id': 'sale_vat_18',
                'account_purchase_tax_id': 'purchase_vat_18',
                'fiscalyear_last_day': '30',
                'fiscalyear_last_month': '6',
                'deferred_expense_account_id': '352809',
                'deferred_revenue_account_id': '411726',
            }
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import template_ug

```

