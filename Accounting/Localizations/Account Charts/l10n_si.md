# Odoo Module: l10n_si

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
    'name': 'Slovenian - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['si'],
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Chart of accounts and taxes for Slovenia.
    """,
    'depends': [
        'account',
        'base_vat',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_account_tag.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_report_ir_data.xml',
        'data/account_tax_report_pd_data.xml',
        'data/account_tax_report_pr_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="account_account_tag_l10n_si_rp_A3" model="account.account.tag">
            <field name="name">rp_a3</field>
            <field name="country_id" ref="base.si"/>
            <field name="applicability">taxes</field>
        </record>
        <record id="account_account_tag_l10n_si_rp_A4" model="account.account.tag">
            <field name="name">rp_a4</field>
            <field name="country_id" ref="base.si"/>
            <field name="applicability">taxes</field>
        </record>
        <record id="account_account_tag_l10n_si_rp_A5" model="account.account.tag">
            <field name="name">rp_a5</field>
            <field name="country_id" ref="base.si"/>
            <field name="applicability">taxes</field>
        </record>
        <record id="account_account_tag_l10n_si_rp_A6" model="account.account.tag">
            <field name="name">rp_a6</field>
            <field name="country_id" ref="base.si"/>
            <field name="applicability">taxes</field>
        </record>
        <record id="account_account_tag_l10n_si_rp_A7" model="account.account.tag">
            <field name="name">rp_a7</field>
            <field name="country_id" ref="base.si"/>
            <field name="applicability">taxes</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <!-- DDV -->
    <record id="tax_report" model="account.report">
        <field name="name">VAT Return (DDV-O)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.si"/>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_I" model="account.report.line">
                <field name="name">I. Supplies of goods and services (values excluding VAT)</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_I_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            c11.balance +
                            c11a.balance +
                            c12.balance +
                            c13.balance +
                            c14.balance +
                            c15.balance
                        </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_11" model="account.report.line">
                        <field name="name">11. Supplies of goods and services</field>
                        <field name="code">c11</field>
                        <field name="expression_ids">
                            <record id="tax_report_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_11a" model="account.report.line">
                        <field name="name">11a. Supplies of goods and services in Slovenia, of which VAT is charged by the recipient</field>
                        <field name="code">c11a</field>
                        <field name="expression_ids">
                            <record id="tax_report_11a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_12" model="account.report.line">
                        <field name="name">12. Deliveries of goods and services to other EU Member States</field>
                        <field name="code">c12</field>
                        <field name="expression_ids">
                            <record id="tax_report_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_13" model="account.report.line">
                        <field name="name">13. Sale of goods at a distance</field>
                        <field name="code">c13</field>
                        <field name="expression_ids">
                            <record id="tax_report_13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_14" model="account.report.line">
                        <field name="name">14. Assembly and installation of goods in another Member State</field>
                        <field name="code">c14</field>
                        <field name="expression_ids">
                            <record id="tax_report_14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">14</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_15" model="account.report.line">
                        <field name="name">15. Exempt supplies without the right to deduct VAT</field>
                        <field name="code">c15</field>
                        <field name="expression_ids">
                            <record id="tax_report_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_II" model="account.report.line">
                <field name="name">II. VAT charged</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_II_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            c21.balance +
                            c22.balance +
                            c22a.balance +
                            c23.balance +
                            c23a.balance +
                            c24.balance +
                            c24a.balance +
                            c24b.balance +
                            c24c.balance +
                            c25.balance +
                            c25a.balance +
                            c25b.balance +
                            c26.balance
                        </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_21" model="account.report.line">
                        <field name="name">21. At a rate of 22%</field>
                        <field name="code">c21</field>
                        <field name="expression_ids">
                            <record id="tax_report_21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">21</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_22" model="account.report.line">
                        <field name="name">22. At a rate of 9,5%</field>
                        <field name="code">c22</field>
                        <field name="expression_ids">
                            <record id="tax_report_22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_22a" model="account.report.line">
                        <field name="name">22a. At a rate of 5%</field>
                        <field name="code">c22a</field>
                        <field name="expression_ids">
                            <record id="tax_report_22a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_23" model="account.report.line">
                        <field name="name">23. 22% of acquisitions of goods from other EU Member States</field>
                        <field name="code">c23</field>
                        <field name="expression_ids">
                            <record id="tax_report_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_23a" model="account.report.line">
                        <field name="name">23a. Of the services received from other EU Member States at a rate of 22%</field>
                        <field name="code">c23a</field>
                        <field name="expression_ids">
                            <record id="tax_report_23a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24" model="account.report.line">
                        <field name="name">24. 9,5% of acquisitions of goods from other EU Member States</field>
                        <field name="code">c24</field>
                        <field name="expression_ids">
                            <record id="tax_report_24_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24a" model="account.report.line">
                        <field name="name">24a. Of the services received from other EU Member States at the rate of 9,5%</field>
                        <field name="code">c24a</field>
                        <field name="expression_ids">
                            <record id="tax_report_24a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24b" model="account.report.line">
                        <field name="name">24b. Acquisitions of goods from other EU Member States at the rate of 5%</field>
                        <field name="code">c24b</field>
                        <field name="expression_ids">
                            <record id="tax_report_24b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24c" model="account.report.line">
                        <field name="name">24c. Of the services received from other EU Member States at the rate of 5%</field>
                        <field name="code">c24c</field>
                        <field name="expression_ids">
                            <record id="tax_report_24c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24c</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_25" model="account.report.line">
                        <field name="name">25. On the basis of self-assessment as a recipient of goods and services at a rate of 22%</field>
                        <field name="code">c25</field>
                        <field name="expression_ids">
                            <record id="tax_report_25_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_25a" model="account.report.line">
                        <field name="name">25a. On the basis of self-assessment as a recipient of goods and services at a rate of 9,5%</field>
                        <field name="code">c25a</field>
                        <field name="expression_ids">
                            <record id="tax_report_25a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_25b" model="account.report.line">
                        <field name="name">25b. On the basis of self-assessment as a recipient of goods and services at a rate of 5%</field>
                        <field name="code">c25b</field>
                        <field name="expression_ids">
                            <record id="tax_report_25b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_26" model="account.report.line">
                        <field name="name">26. On the basis of self-assessment of imports</field>
                        <field name="code">c26</field>
                        <field name="expression_ids">
                            <record id="tax_report_26_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">26</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_III" model="account.report.line">
                <field name="name">III. Purchases of goods and services (values excluding VAT)</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_III_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            c31.balance +
                            c31a.balance +
                            c32.balance +
                            c32a.balance +
                            c33.balance +
                            c34.balance +
                            c35.balance
                        </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_31" model="account.report.line">
                        <field name="name">31. Purchases of goods and services</field>
                        <field name="code">c31</field>
                        <field name="expression_ids">
                            <record id="tax_report_31_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_31a" model="account.report.line">
                        <field name="name">31a. Purchases of goods and services in Slovenia, of which the recipient charges VAT</field>
                        <field name="code">c31a</field>
                        <field name="expression_ids">
                            <record id="tax_report_31a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_32" model="account.report.line">
                        <field name="name">32. Acquisitions of goods from other EU Member States</field>
                        <field name="code">c32</field>
                        <field name="expression_ids">
                            <record id="tax_report_32_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_32a" model="account.report.line">
                        <field name="name">32a. Services received from other EU Member States</field>
                        <field name="code">c32a</field>
                        <field name="expression_ids">
                            <record id="tax_report_32a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_33" model="account.report.line">
                        <field name="name">33. Exempt purchases of goods and services and exempt acquisitions of goods</field>
                        <field name="code">c33</field>
                        <field name="expression_ids">
                            <record id="tax_report_33_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">33</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_34" model="account.report.line">
                        <field name="name">34. Purchase value of real estate</field>
                        <field name="code">c34</field>
                        <field name="expression_ids">
                            <record id="tax_report_34_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">34</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_35" model="account.report.line">
                        <field name="name">35. Cost of other fixed assets</field>
                        <field name="code">c35</field>
                        <field name="expression_ids">
                            <record id="tax_report_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">35</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_IV" model="account.report.line">
                <field name="name">IV. VAT deduction</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_IV_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c41.balance + c42.balance + c42a.balance + c43.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_41" model="account.report.line">
                        <field name="name">41. From purchases of goods and services, acquisition of goods and services received from other EU Member States and from imports at a rate of 22%</field>
                        <field name="code">c41</field>
                        <field name="expression_ids">
                            <record id="tax_report_41_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">41</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_42" model="account.report.line">
                        <field name="name">42. From purchases of goods and services, acquisition of goods and services received from other EU Member States and from imports at a rate of 9,5%</field>
                        <field name="code">c42</field>
                        <field name="expression_ids">
                            <record id="tax_report_42_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_42a" model="account.report.line">
                        <field name="name">42a. From purchases of goods and services, acquisition of goods and services received from other EU Member States and from imports at a rate of 5%</field>
                        <field name="code">c42a</field>
                        <field name="expression_ids">
                            <record id="tax_report_42a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_43" model="account.report.line">
                        <field name="name">43. Of the flat-rate compensation at the rate of 8%</field>
                        <field name="code">c43</field>
                        <field name="expression_ids">
                            <record id="tax_report_43_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">43</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_51" model="account.report.line">
                <field name="name">51. VAT liability</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_51_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            c21.balance +
                            c22.balance +
                            c22a.balance +
                            c23.balance +
                            c23a.balance +
                            c24.balance +
                            c24a.balance +
                            c24b.balance +
                            c25.balance +
                            c25a.balance +
                            c25b.balance +
                            c26.balance -
                            c41.balance -
                            c42.balance -
                            c42a.balance -
                            c43.balance
                        </field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_52" model="account.report.line">
                <field name="name">52. VAT surplus</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_52_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            c41.balance +
                            c42.balance +
                            c42a.balance +
                            c43.balance -
                            c21.balance -
                            c22.balance -
                            c22a.balance -
                            c23.balance -
                            c23a.balance -
                            c24.balance -
                            c24a.balance -
                            c24b.balance -
                            c25.balance -
                            c25a.balance -
                            c25b.balance -
                            c26.balance
                        </field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_report_ir_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <!-- IR -->
    <record id="tax_report_ir" model="account.report">
        <field name="name">Payable VAT (IR)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.si"/>
        <field name="column_ids">
            <record id="tax_report_ir_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_ir_7" model="account.report.line">
                <field name="name">7. SLO Value excluding VAT</field>
                <field name="code">ir_7</field>
                <field name="expression_ids">
                    <record id="tax_report_ir_7_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">ir_7</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ir_8" model="account.report.line">
                <field name="name">8. Self-taxation of supplies Article 76a</field>
                <field name="code">ir_8</field>
                <field name="expression_ids">
                    <record id="tax_report_ir_8_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">ir_8</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ir_exempt_turnover_eu" model="account.report.line">
                <field name="name">EXEMPTED TURNOVER - EU</field>
                <field name="expression_ids">
                    <record id="tax_report_ir_exempt_turnover_eu_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            ir_9.balance +
                            ir_10a.balance +
                            ir_10b.balance +
                            ir_10c.balance +
                            ir_10d.balance +
                            ir_11.balance +
                            ir_12.balance +
                            ir_13.balance
                        </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_ir_9" model="account.report.line">
                        <field name="name">9. No right to deduct VAT</field>
                        <field name="code">ir_9</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_9</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_10a" model="account.report.line">
                        <field name="name">10a. Supplies of goods</field>
                        <field name="code">ir_10a</field>
                        <field name="sequence">3200</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_10a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_10a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_10b" model="account.report.line">
                        <field name="name">10b. Custom posts 42 and 43</field>
                        <field name="code">ir_10b</field>
                        <field name="sequence">3300</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_10b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_10b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_10c" model="account.report.line">
                        <field name="name">10c. Deliveries of bl. after storage on recall</field>
                        <field name="code">ir_10c</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_10c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_10c</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_10d" model="account.report.line">
                        <field name="name">10d. Passed st.</field>
                        <field name="code">ir_10d</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_10d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_10d</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_11" model="account.report.line">
                        <field name="name">11. Three-way delivery</field>
                        <field name="code">ir_11</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_12" model="account.report.line">
                        <field name="name">12. Distance selling</field>
                        <field name="code">ir_12</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_13" model="account.report.line">
                        <field name="name">13. Assembly</field>
                        <field name="code">ir_13</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_13</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ir_taxed_turnover" model="account.report.line">
                <field name="name">TAXED TURNOVER</field>
                <field name="expression_ids">
                    <record id="tax_report_ir_taxed_turnover_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            ir_14.balance +
                            ir_15a.balance +
                            ir_15b.balance +
                            ir_16.balance +
                            ir_17.balance +
                            ir_18a.balance +
                            ir_19a.balance +
                            ir_18b.balance +
                            ir_19b.balance +
                            ir_20.balance +
                            ir_21a.balance +
                            ir_21b.balance +
                            ir_22.balance +
                            ir_22a.balance +
                            ir_22b.balance
                        </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_ir_transport_slovenia" model="account.report.line">
                        <field name="name">Transport in Slovenia</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_transport_slovenia_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    ir_14.balance +
                                    ir_15a.balance +
                                    ir_15b.balance
                                </field>
                            </record>
                        </field>
                         <field name="children_ids">
                            <record id="tax_report_ir_14" model="account.report.line">
                                <field name="name">14. VAT 22%</field>
                                <field name="code">ir_14</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_14</field>
                                    </record>
                                </field>
                            </record>
                             <record id="tax_report_ir_15a" model="account.report.line">
                                <field name="name">15a. VAT 9,5%</field>
                                <field name="code">ir_15a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_15a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_15a</field>
                                    </record>
                                </field>
                            </record>
                             <record id="tax_report_ir_15b" model="account.report.line">
                                <field name="name">15b. VAT 5%</field>
                                <field name="code">ir_15b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_15b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_15b</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_self_taxation_eu" model="account.report.line">
                        <field name="name">Self-taxation - EU</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_self_taxation_eu_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    ir_16.balance +
                                    ir_17.balance +
                                    ir_18a.balance +
                                    ir_18b.balance +
                                    ir_19a.balance +
                                    ir_19b.balance
                                </field>
                            </record>
                        </field>
                         <field name="children_ids">
                            <record id="tax_report_ir_16" model="account.report.line">
                                <field name="name">16. VAT BL 22%</field>
                                <field name="code">ir_16</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_16</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_ir_17" model="account.report.line">
                                <field name="name">17. VAT ST 22%</field>
                                <field name="code">ir_17</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_17</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_ir_18a" model="account.report.line">
                                <field name="name">18a. VAT BL 9.5%</field>
                                <field name="code">ir_18a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_18a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_18a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_ir_19a" model="account.report.line">
                                <field name="name">19a. VAT ST 9.5%</field>
                                <field name="code">ir_19a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_19a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_19a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_ir_18b" model="account.report.line">
                                <field name="name">18b. VAT BL 5%</field>
                                <field name="code">ir_18b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_18b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_18b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_ir_19b" model="account.report.line">
                                <field name="name">19b. VAT ST 5%</field>
                                <field name="code">ir_19b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_19b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_19b</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_self_taxation_slo_under_article_761" model="account.report.line">
                        <field name="name">Self-taxation - SLO under Article 76a</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_self_taxation_slo_under_article_761_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">ir_20.balance + ir_21a.balance + ir_21b.balance</field>
                            </record>
                        </field>
                         <field name="children_ids">
                             <record id="tax_report_ir_20" model="account.report.line">
                                <field name="name">20. VAT 22%</field>
                                <field name="code">ir_20</field>
                                <field name="sequence">4310</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_20</field>
                                    </record>
                                </field>
                             </record>
                             <record id="tax_report_ir_21a" model="account.report.line">
                                <field name="name">21a. VAT 9,5%</field>
                                <field name="code">ir_21a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_21a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_21a</field>
                                    </record>
                                </field>
                            </record>
                             <record id="tax_report_ir_21b" model="account.report.line">
                                <field name="name">21b. VAT 5%</field>
                                <field name="code">ir_21b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_21b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_21b</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_self_taxation_3rd_world_imports" model="account.report.line">
                        <field name="name">Self-taxation - 3rd world imports</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_self_taxation_3rd_world_imports_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">ir_22.balance + ir_22a.balance + ir_22b.balance</field>
                            </record>
                        </field>
                         <field name="children_ids">
                             <record id="tax_report_ir_22" model="account.report.line">
                                <field name="name">22. VAT 22%</field>
                                <field name="code">ir_22</field>
                                <field name="sequence">4410</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_22</field>
                                    </record>
                                </field>
                             </record>
                             <record id="tax_report_ir_22a" model="account.report.line">
                                <field name="name">22a. VAT 9,5%</field>
                                <field name="code">ir_22a</field>
                                <field name="sequence">4420</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_22a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_22a</field>
                                    </record>
                                </field>
                            </record>
                             <record id="tax_report_ir_22b" model="account.report.line">
                                <field name="name">22b. VAT 5%</field>
                                <field name="code">ir_22b</field>
                                <field name="sequence">4430</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ir_22b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ir_22b</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ir_3rd_world_deliveries_with_right_withdrawal" model="account.report.line">
                <field name="name">3rd world deliveries with right of withdrawal VAT</field>
                <field name="expression_ids">
                    <record id="tax_report_ir_23_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">ir_23</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ir_reverse_charge_base" model="account.report.line">
                <field name="name">REVERSE CHARGE - BASE</field>
                <field name="expression_ids">
                    <record id="tax_report_ir_reverse_charge_base_base" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ir_25a.balance + ir_25b.balance + ir_25c.balance + ir_25d.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_ir_25a" model="account.report.line">
                        <field name="name">25a. Acquisition of EU goods</field>
                        <field name="code">ir_25a</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_25a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_25a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_25b" model="account.report.line">
                        <field name="name">25b. EU services received</field>
                        <field name="code">ir_25b</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_25b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_25b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_25c" model="account.report.line">
                        <field name="name">25c. Acquisition SLO Article 76a</field>
                        <field name="code">ir_25c</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_25c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_25c</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_25d" model="account.report.line">
                        <field name="name">25d. Purchases other for self-taxation</field>
                        <field name="code">ir_25d</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_25d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ir_25d</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_report_pd_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <!-- PD -->
    <record id="tax_report_pd" model="account.report">
        <field name="name">Slovenia VAT RC (PD-O)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.si"/>
        <field name="column_ids">
            <record id="tax_report_pd_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_pd_a3" model="account.report.line">
                <field name="name">a3. Value of supplies of goods and services</field>
                <field name="code">pd_a3</field>
                <field name="expression_ids">
                    <record id="tax_report_pd_a3_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">pd_a3</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_report_pr_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <!-- PR -->
    <record id="tax_report_pr" model="account.report">
        <field name="name">Receivable VAT (PR) </field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.si"/>
        <field name="column_ids">
            <record id="tax_report_pr_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_pr_value_taxes_purchases_base" model="account.report.line">
                <field name="name">Value of taxed purchases - base</field>
                <field name="expression_ids">
                    <record id="tax_report_pr_value_taxes_purchases_base_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">
                            pr_8a.balance +
                            pr_8b.balance +
                            pr_9.balance +
                            pr_10.balance +
                            pr_11.balance +
                            pr_12.balance +
                            pr_13.balance
                        </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_pr_8a" model="account.report.line">
                        <field name="name">8a. Goods and services SLO (except for self employed)</field>
                        <field name="code">pr_8a</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_8a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_8a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_8b" model="account.report.line">
                        <field name="name">8b. Goods and services 3rd world</field>
                        <field name="code">pr_8b</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_8b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_8b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_9" model="account.report.line">
                        <field name="name">9. SLO acquisitions (Art. 76a only)</field>
                        <field name="code">pr_9</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_9</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_10" model="account.report.line">
                        <field name="name">10. Acquisitions of goods from the EU</field>
                        <field name="code">pr_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_11" model="account.report.line">
                        <field name="name">11. Services received from the EU</field>
                        <field name="code">pr_11</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_12" model="account.report.line">
                        <field name="name">12. Real estate (Part 8 or 9)</field>
                        <field name="code">pr_12</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_13" model="account.report.line">
                        <field name="name">13. Other OS (Part 8 or 10)</field>
                        <field name="code">pr_13</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_13</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pr_value_exempt_purchases_base" model="account.report.line">
                <field name="name">Value of exempt purchases - base</field>
                <field name="expression_ids">
                    <record id="tax_report_pr_value_exempt_purchases_base_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">pr_14.balance + pr_15.balance + pr_16.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_pr_14" model="account.report.line">
                        <field name="name">14. Purchases and acquisitions</field>
                        <field name="code">pr_14</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_14</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_15" model="account.report.line">
                        <field name="name">15. Real estate (Part 14)</field>
                        <field name="code">pr_15</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_15</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_16" model="account.report.line">
                        <field name="name">16. Other OS (Part 14)</field>
                        <field name="code">pr_16</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_16</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pr_non_deductible_vat" model="account.report.line">
                <field name="name">Non-deductible VAT</field>
                <field name="expression_ids">
                    <record id="tax_report_pr_non_deductible_vat_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">pr_17a.balance + pr_17b.balance + pr_17c.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_pr_17a" model="account.report.line">
                        <field name="name">17a. 22%</field>
                        <field name="code">pr_17a</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_17a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_17a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_17b" model="account.report.line">
                        <field name="name">17b. 9,5%</field>
                        <field name="code">pr_17b</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_17b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_17b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_17c" model="account.report.line">
                        <field name="name">17c. 8% and 5%</field>
                        <field name="code">pr_17c</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_17c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_17c</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pr_deductible_vat" model="account.report.line">
                <field name="name">Deductible VAT</field>
                <field name="expression_ids">
                    <record id="tax_report_pr_deductible_vat_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">pr_18.balance + pr_19.balance + pr_19a.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_pr_18" model="account.report.line">
                        <field name="name">18. 22%</field>
                        <field name="code">pr_18</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_18</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_19" model="account.report.line">
                        <field name="name">19. 9,5%</field>
                        <field name="code">pr_19</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_19_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_19</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pr_19a" model="account.report.line">
                        <field name="name">19a. 5%</field>
                        <field name="code">pr_19a</field>
                        <field name="expression_ids">
                            <record id="tax_report_pr_19a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">pr_19a</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pr_20" model="account.report.line">
                <field name="name">20. Flat-rate compensation 8%</field>
                <field name="expression_ids">
                    <record id="tax_report_pr_20_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">pr_20</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-si.csv

```csv
"id","code","account_type","name","reconcile","name@sl"
"gd_acc_000000","000000","asset_non_current","Good name","False","Dobro ime"
"gd_acc_002000","002000","asset_non_current","Deferred development costs","False","Odloženi stroški razvijanja"
"gd_acc_003000","003000","asset_non_current","Property and other rights","False","Premoženjske in druge pravice"
"gd_acc_005000","005000","asset_fixed","Other intangible assets","False","Druga neopredmetena sredstva"
"gd_acc_007000","007000","asset_non_current","Long-term active accruals","False","Dolgoročne aktivne časovne razmejitve"
"gd_acc_008000","008000","asset_non_current","Amortisation adjustment to intangible assets","False","Popravek vrednosti neopredmetenih sredstev zaradi amortiziranja"
"gd_acc_009000","009000","asset_non_current","Impairment of intangible assets","False","Oslabitev vrednosti neopredmetenih sredstev"
"gd_acc_010000","010000","asset_non_current","Investment property valued according to the cost model","False","Naložbene nepremičnine, vrednotene po modelu nabavne vrednosti"
"gd_acc_011000","011000","asset_non_current","Investment property valued using the fair value model","False","Naložbene nepremičnine, vrednotene po modelu poštene vrednosti"
"gd_acc_015000","015000","asset_non_current","Allowance for depreciation of investment property","False","Popravek vrednosti naložbenih nepremičnin zaradi amortiziranja"
"gd_acc_017000","017000","asset_non_current","Investment property under construction or development","False","Naložbene nepremičnine v gradnji oziroma izdelavi"
"gd_acc_019000","019000","asset_non_current","Impairment of investment property","False","Oslabitev vrednosti naložbenih nepremičnin"
"gd_acc_020000","020000","asset_fixed","Land valued using the cost model","False","Zemljišča, vrednotena po modelu nabavne vrednosti"
"gd_acc_021000","021000","asset_fixed","Buildings valued using the cost model","False","Zgradbe, vrednotene po modelu nabavne vrednosti"
"gd_acc_021100","021100","asset_fixed","Buildings - leasing","False","Zgradbe - leasing"
"gd_acc_022000","022000","asset_fixed","Land valued using the revaluation model","False","Zemljišča, vrednotena po modelu revaloriziranja"
"gd_acc_022100","022100","asset_fixed","Land valued according to the revaluation model","False","Zemljišča, vrednotena po modelu prevrednotenja"
"gd_acc_023000","023000","asset_fixed","Buildings valued according to the revaluation model","False","Zgradbe, vrednotene po modelu revaloriziranja"
"gd_acc_026000","026000","asset_fixed","Investing in foreign-owned real estate","False","Vlaganja v nepremičnine  v tuji lasti"
"gd_acc_027000","027000","asset_fixed","Real estate under construction or construction","False","Nepremičnine v gradnji oziroma izdelavi"
"gd_acc_031000","031000","asset_fixed","Impairment of land value","False","Oslabitev vrednosti zemljišč"
"gd_acc_032000","032000","asset_fixed","Correction of the value of land (quarries, waste disposal sites) due to depreciation","False","Popravek vrednosti zemljišč (kamnolomov, odlagališč odpadkov) zaradi amortiziranja"
"gd_acc_035000","035000","asset_fixed","Correction of the value of buildings due to depreciation","False","Popravek vrednosti zgradb zaradi amortiziranja"
"gd_acc_035100","035100","asset_fixed","Correction of the value of buildings - leasing","False","Popravek vrednosti zgradb - leasing"
"gd_acc_036000","036000","asset_fixed","Correction of the value of investments in foreign-owned real estate","False","Popravek vrednosti vlaganj v nepremičnine v tuji lasti"
"gd_acc_039000","039000","asset_fixed","Impairment of value of buildings","False","Oslabitev vrednosti zgradb"
"gd_acc_040000","040000","asset_fixed","Equipment and spare parts valued according to the cost model","False","Oprema in nadomestni deli, vrednoteni po modelu nabavne vrednosti"
"gd_acc_040100","040100","asset_fixed","Equipment and spare parts - leasing","False","Oprema in nadomestni deli - leasing"
"gd_acc_041000","041000","asset_fixed","Small inventory","False","Drobni inventar"
"gd_acc_043000","043000","asset_fixed","Biological means - perennial plantations","False","Biološka sredstva - večletni nasadi"
"gd_acc_044000","044000","asset_fixed","Biological means - basic herd","False","Biološka sredstva - osnovna čreda"
"gd_acc_045000","045000","asset_fixed","Other tangible fixed assets","False","Druga opredmetena osnovna sredstva"
"gd_acc_045100","045100","asset_fixed","Other tangible fixed assets - leasing","False","Druga opredmetena osnovna sredstva - leasing"
"gd_acc_046000","046000","asset_fixed","Investments in foreign-owned tangible assets","False","Vlaganja v opredmetena sredstva v tuji lasti"
"gd_acc_047000","047000","asset_fixed","Equipment and other tangible fixed assets in construction or production","False","Oprema in druga opredmetena osnovna sredstva v gradnji oziroma izdelavi"
"gd_acc_048000","048000","asset_fixed","Works of art and other objects of cultural and/or historical value that are not depreciated","False","Umetniška dela in drugi predmeti kulturne in/ali zgodovinske vrednosti, ki se ne amortizirajo"
"gd_acc_050000","050000","asset_fixed","Correction of the value of equipment and spare parts due to depreciation","False","Popravek vrednosti opreme in nadomestnih delov zaradi amortiziranja"
"gd_acc_050100","050100","asset_fixed","Correction of the value of equipment and spare parts - leasing","False","Popravek vrednosti opreme in nadomestnih delov - leasing"
"gd_acc_051000","051000","asset_fixed","Adjustment of value of petty inventory due to depreciation","False","Popravek vrednosti drobnega inventarja zaradi amortiziranja"
"gd_acc_052000","052000","asset_fixed","Depreciation of equipment and spare parts","False","Oslabitev vrednosti opreme in nadomestnih delov"
"gd_acc_053000","053000","asset_fixed","Correction of the value of biological assets - perennial plantations due to depreciation","False","Popravek vrednosti bioloških sredstev - večletnih nasadov zaradi amortiziranja"
"gd_acc_054000","054000","asset_fixed","Correction of the value of biological assets - basic herd due to depreciation","False","Popravek vrednosti bioloških sredstev - osnovne črede zaradi amortiziranja"
"gd_acc_055000","055000","asset_fixed","Correction of the value of other tangible fixed assets due to depreciation","False","Popravek vrednosti drugih opredmetrenih osnovnih sredstev  zaradi amortiziranja"
"gd_acc_055100","055100","asset_fixed","Correction of the value of other tangible fixed assets - leasing","False","Popravek vrednosti drugih opredmetrenih osnovnih sredstev - leasing"
"gd_acc_056000","056000","asset_fixed","Correction of the value of investments in tangible fixed assets in foreign ownership","False","Popravek vrednosti vlaganj v opredmetena osnovna sredstva v tuji lasti"
"gd_acc_058000","058000","asset_fixed","Impairment of the value of biological assets","False","Oslabitev vrednosti bioloških sredstev"
"gd_acc_059000","059000","asset_fixed","Impairment of value of other tangible fixed assets","False","Oslabitev vrednosti drugih opredmetenih osnovnih sredstev"
"gd_acc_060000","060000","asset_non_current","Long-term financial investments in shares and shares of organizations in the group, distributed and measured by purchase value","False","Dolgoročne finančne naložbe v delnice in deleže organizacij v skupini, razporejene in izmerjene po nabavni vrednosti"
"gd_acc_063000","063000","asset_non_current","Long-term financial investments in shares and shares of affiliated organizations and jointly controlled organizations, distributed and measured at cost","False","Dolgoročne finančne naložbe v delnice in deleže pridruženih organizacij in skupaj obvladovanih organizacij, razporejene in izmerjene po nabavni vrednosti"
"gd_acc_066000","066000","asset_non_current","Long-term financial investments in shares and shares of affiliated organizations and jointly controlled organizations, distributed and measured at cost","False","Druge dolgoročne finančne naložbe, razporejene in izmerjene po nabavni vrednosti"
"gd_acc_067000","067000","asset_non_current","Other long-term financial investments allocated and measured at fair value through profit or loss","False","Druge dolgoročne finančne naložbe, razporejene in izmerjene po pošteni vrednosti prek poslovnega izida"
"gd_acc_068000","068000","asset_non_current","Other long-term financial investments, distributed and measured at fair value through capital or own source of funds","False","Druge dolgoročne finančne naložbe, razporejene in izmerjene po pošteni vrednosti prek kapitala oziroma lastnega vira sredstev"
"gd_acc_069000","069000","asset_non_current","Impairment of the value of long-term financial investments","False","Oslabitev vrednosti dolgoročnih finančnih naložb"
"gd_acc_070000","070000","asset_non_current","Long-term loans granted under loan agreements to group organisations, including long-term finance lease receivables","False","Dolgoročna posojila, dana na podlagi posojilnih pogodb organizacijam v skupini, vključno z dolgoročnimi terjatvami iz finančnega najema"
"gd_acc_071000","071000","asset_non_current","Long-term loans granted under loan agreements to associates and jointly controlled entities, including long-term finance lease receivables","False","Dolgoročna posojila, dana na podlagi posojilnih pogodb pridruženim organizacijam in skupaj obvladovanim organizacijam, vključno z dolgoročnimi terjatvami finančnega najema"
"gd_acc_072000","072000","asset_non_current","Long-term loans to others, including long-term receivables from finance leases","False","Dolgoročna posojila, dana drugim, vključno z dolgoročnimi terjatvami iz finančnega najema"
"gd_acc_073000","073000","asset_non_current","Long-term loans granted through the repurchase of bonds from group companies","False","Dolgoročna posojila, dana z odkupom obveznic od družb v skupini"
"gd_acc_074000","074000","asset_non_current","Long-term loans granted through the repurchase of bonds from associates and jointly controlled entities","False","Dolgoročna posojila, dana z odkupom obveznic od pridruženih organizacij in skupaj obvladovanih organizacij"
"gd_acc_075000","075000","asset_non_current","Long-term loans granted through the repurchase of bonds from others","False","Dolgoročna posojila, dana z odkupom obveznic od drugih"
"gd_acc_076000","076000","asset_non_current","Long-term receivables for uncalled capital","False","Dolgoročne terjatve za nevplačani vpoklicani kapital"
"gd_acc_077000","077000","asset_non_current","Other long-term invested assets","False","Druga dolgoročno vložena sredstva"
"gd_acc_078000","078000","asset_non_current","Long-term deposits granted","False","Dani dolgoročni depoziti"
"gd_acc_079000","079000","asset_non_current","Impairment of long-term loans granted","False","Oslabitev vrednosti danih dolgoročnih posojil"
"gd_acc_080000","080000","asset_non_current","Long-term commodity credits granted in the country","False","Dolgoročni blagovni krediti, dani v državi"
"gd_acc_081000","081000","asset_non_current","Long-term commodity credits granted abroad","False","Dolgoročni blagovni krediti, dani v tujini"
"gd_acc_082000","082000","asset_non_current","Long-term consumer credit granted","False","Dani dolgoročni potrošniški krediti"
"gd_acc_083000","083000","asset_non_current","Long-term advances granted","False","Dani dolgoročni predujmi"
"gd_acc_084000","084000","asset_non_current","Long-term securities given","False","Dane dolgoročne varščine"
"gd_acc_086000","086000","asset_non_current","Other long-term trade receivables","False","Druge dolgoročne poslovne terjatve"
"gd_acc_087000","087000","asset_non_current","Long-term trade receivables from group companies","False","Dolgoročne poslovne terjatve do družb v skupini"
"gd_acc_089000","008900","asset_non_current","Impairment of long-term trade receivables","False","Oslabitev vrednosti dolgoročnih poslovnih terjatev"
"gd_acc_090000","090000","asset_non_current","Deferred tax assets arising from deductible temporary differences","False","Odložene terjatve za davek iz odbitnih začasnih razlik"
"gd_acc_091000","091000","asset_non_current","Deferred tax assets arising from unused tax loss carryforwards","False","Odložene terjatve za davek iz neizrabljenih davčnih izgub, prenesenih v naslednja davčna obdobja"
"gd_acc_091900","091900","asset_non_current","Receivables on own account","False","Terjatve na lastniškem računu"
"gd_acc_092000","092000","asset_non_current","Deferred tax assets arising from tax credit carry-forwards","False","Odložene terjatve za davek iz davčnih dobropisov, prenesenih v naslednja davčna obdobja"
"gd_acc_100000","100000","asset_cash","Cash in hand, other than foreign currency","False","Denarna sredstva v blagajni, razen deviznih"
"gd_acc_100001","100001","asset_cash","Cash","False","Gotovina"
"gd_acc_101000","101000","asset_cash","Foreign currency in the treasury","False","Devizna sredstva v blagajni"
"gd_acc_102000","102000","asset_cash","Cheques issued (deductible item)","False","Izdani čeki (odbitna postavka)"
"gd_acc_103000","103000","asset_cash","Cheques received","False","Prejeti čeki"
"gd_acc_104000","104000","asset_cash","Risk-free readily callable debt securities","False","Netvegani takoj udenarljivi dolžniški vrednostni papirji"
"gd_acc_109000","109000","asset_cash","Money on the move","False","Denar na poti"
"gd_acc_109001","109001","asset_current","Liquidity Transfer","True","Liquidity Transfer"
"gd_acc_110000","110000","asset_cash","Cash on accounts other than foreign currency","False","Denarna sredstva na računih, razen deviznih"
"gd_acc_110001","110001","asset_current","Bank","True","Banka"
"gd_acc_110002","110002","asset_current","Bank suspense account","False","Bančni prehodni konto"
"gd_acc_110003","110003","asset_current","Outstanding Receipts","True","Outstanding Receipts"
"gd_acc_110004","110004","asset_current","Outstanding Payments","True","Outstanding Payments"
"gd_acc_110005","110005","asset_cash","Payments","False","Plačila"
"gd_acc_110900","110900","asset_cash","Transition account for unallocated payments other than foreign exchange","False","Prehodni konto za nerazporejena plačila, razen deviznih"
"gd_acc_111000","111000","asset_cash","Short-term deposits or deposits redeemable at sight, excluding foreign currency","False","Kratkoročni depoziti oziroma depoziti na odpoklic, razen deviznih"
"gd_acc_112000","112000","asset_cash","Foreign currency balances on accounts","False","Devizna sredstva na računih"
"gd_acc_112900","112900","asset_cash","Transition account for unallocated foreign exchange payments","False","Prehodni konto za nerazporejena devizna plačila"
"gd_acc_113000","113000","asset_cash","Short-term foreign currency deposits or foreign currency deposits redeemable at call","False","Kratkoročni devizni depoziti oziroma devizni depoziti na odpoklic"
"gd_acc_114000","114000","asset_cash","Cash in special accounts or for specific purposes","False","Denarna sredstva na posebnih računih oziroma za posebne namene"
"gd_acc_120000","120000","asset_receivable","Short-term receivables from customers in the country","True","Kratkoročne terjatve do kupcev v državi"
"gd_acc_121000","121000","asset_receivable","Short-term receivables from customers abroad","True","Kratkoročne terjatve do kupcev v tujini"
"gd_acc_122000","122000","asset_current","Short-term trade credits granted to customers in the country","False","Kratkoročni blagovni krediti, dani kupcem v državi"
"gd_acc_123000","123000","asset_current","Short-term trade credits to customers abroad","False","Kratkoročni blagovni krediti, dani kupcem v tujini"
"gd_acc_124000","124000","asset_current","Short-term consumer credit granted to customers in the country","False","Kratkoročni potrošniški krediti, dani kupcem v državi"
"gd_acc_125000","125000","asset_receivable","Short-term receivables from members","True","Kratkoročnet terjatve do članov"
"gd_acc_126000","126000","asset_current","Short-term receivables for unbilled goods and services","False","Kratkoročne terjatve za nezaračunano blago in storitve"
"gd_acc_127000","127000","asset_current","Short-term trade receivables from group companies","False","Kratkoročne poslovne terjatve do družb v skupini"
"gd_acc_129000","129000","asset_current","Impairment of short-term trade receivables","False","Oslabitev vrednosti kratkoročnih terjatev do kupcev"
"gd_acc_130000","130000","asset_current","Short-term advances against property, plant and equipment","False","Kratkoročni predujmi, dani za opredmetena osnovna sredstva"
"gd_acc_131000","131000","asset_current","Short-term advances made for intangible fixed assets","False","Kratkoročni predujmi, dani za neopredmetena osnovna sredstva"
"gd_acc_132000","132000","asset_current","Short-term advances against inventories of materials and goods and services not yet rendered","False","Kratkoročni predujmi, dani za zaloge materiala in blaga ter še ne opravljene storitve"
"gd_acc_133000","133000","asset_receivable","Other short-term advances and overpayments","True","Drugi dani kratkoročni predujmi in preplačila"
"gd_acc_134000","134000","asset_current","Short-term securities given","False","Dane kratkoročne varščine"
"gd_acc_139000","139000","asset_current","Impairment of short-term advances, overpayments and securities","False","Oslabitev vrednosti danih kratkoročnih predujmov, preplačil in varščin"
"gd_acc_140000","140000","asset_current","Short-term receivables from exporters","False","Kratkoročne terjatve do izvoznikov"
"gd_acc_141000","141000","asset_current","Short-term receivables from imports on foreign account","False","Kratkoročne terjatve iz uvoza za tuj račun"
"gd_acc_142000","142000","asset_current","Short-term receivables from consignment and consignment sales","False","Kratkoročne terjatve iz komisijske in konsignacijske prodaje"
"gd_acc_145000","145000","asset_current","Other short-term trade receivables due on foreign account","False","Druge kratkoročne terjatve iz poslovanja za tuj račun"
"gd_acc_149000","149000","asset_current","Impairment of short-term trade receivables due on foreign account","False","Oslabitev vrednosti kratkoročnih terjatev iz poslovanja za tuj račun"
"gd_acc_150000","150000","asset_current","Short-term interest receivable","False","Kratkoročne terjatve za obresti"
"gd_acc_151000","151000","asset_current","Short-term dividend receivable","False","Kratkoročne terjatve za dividende"
"gd_acc_152000","152000","asset_current","Short-term receivables for other profit sharing","False","Kratkoročne terjatve za druge deleže v dobičku"
"gd_acc_155000","155000","asset_current","Other short-term receivables related to financial income","False","Druge kratkoročne terjatve, povezane s finančnimi prihodki"
"gd_acc_159000","159000","asset_current","Impairment of short-term receivables related to financial income","False","Oslabitev vrednosti kratkoročnih terjatev, povezanih s finančnimi prihodki"
"gd_acc_160000","160000","asset_current","Input VAT - 22% - SLO BL ST","False","Vstopni DDV - 22% - SLO BL ST"
"gd_acc_160010","160010","asset_current","Input VAT - 9,5% - SLO BL ST","False","Vstopni DDV - 9,5% - SLO BL ST"
"gd_acc_160020","160020","asset_current","Input VAT - 5% - SLO BL ST","False","Vstopni DDV - 5% - SLO BL ST"
"gd_acc_160300","160300","asset_current","Input VAT - 22% - 3. svet BL","False","Vstopni DDV - 22% - 3. svet BL"
"gd_acc_160301","160301","asset_current","Input VAT - 22% - 3. svet ST","False","Vstopni DDV - 22% - 3. svet ST"
"gd_acc_160310","160310","asset_current","Input VAT - 9,5% - 3. svet BL","False","Vstopni DDV - 9,5% - 3. svet BL"
"gd_acc_160311","160311","asset_current","Input VAT - 9,5% - 3. svet ST","False","Vstopni DDV - 9,5% - 3. svet ST"
"gd_acc_160320","160320","asset_current","Input VAT - 5% 3. svet BL in ST","False","Vstopni DDV - 5% 3. svet BL in ST"
"gd_acc_160400","160400","asset_current","Input VAT - 22% - EU BL","False","Vstopni DDV - 22% - EU BL"
"gd_acc_160410","160410","asset_current","Input VAT - 9,5% - EU BL","False","Vstopni DDV - 9,5% - EU BL"
"gd_acc_160420","160420","asset_current","Input VAT - 5% EU BL in ST","False","Vstopni DDV - 5% EU BL in ST"
"gd_acc_160430","160430","asset_current","Input VAT - 22% - EU ST","False","Vstopni DDV - 22% - EU ST"
"gd_acc_160440","160440","asset_current","Input VAT - 9,5% - EU ST","False","Vstopni DDV - 9,5% - EU ST"
"gd_acc_160460","160460","asset_current","Input VAT - 22% - SLO - SO 76.a Article BL ST","False","Vstopni DDV - 22% - SLO - SO 76.a člen BL ST"
"gd_acc_160470","160470","asset_current","Input VAT - 9.5% - SLO - SO Article 76a BL ST","False","Vstopni DDV - 9,5% - SLO - SO 76.a člen BL ST"
"gd_acc_160500","160500","asset_current","Input VAT - 8% - SLO BL","False","Vstopni DDV - 8% - SLO BL"
"gd_acc_160700","160700","asset_current","Input VAT receivable in Slovenia after accounting","False","Terjatve za vstopni DDV v Sloveniji po obračunu"
"gd_acc_160800","160800","asset_current","Claims for the refund of the difference between input and input VAT in Slovenia","False","Terjatve za vračilo razlike med vstopnim in obračunanim DDV v Sloveniji"
"gd_acc_161000","161000","asset_current","Short-term receivables for corporation tax, including tax paid abroad","False","Kratkoročne terjatve za davek od dohodkov pravnih oseb, vključno z davkom, plačanim v tujini"
"gd_acc_161100","161100","asset_current","Short-term debit card receivables","False","Kratkoročne terjatve iz naslova plačilnih kartic"
"gd_acc_162000","162000","asset_current","Current receivables for income tax on s.p. income, including tax paid abroad","False","Kratkoročne terjatve zd dohodnino od dohodka iz dejavnosti s.p., vključno z davkom plačanim v tujini"
"gd_acc_164000","164000","asset_current","Other short-term receivables from government and other institutions","False","Druge kratkoročne terjatve do državnih in drugih inštitucij"
"gd_acc_164100","164100","asset_current","Short-term receivables for gross salary reimbursements","False","Kratkoročne terjatve za bruto refundirane plače"
"gd_acc_165000","165000","asset_current","Other short-term receivables","False","Ostale kratkoročne terjatve"
"gd_acc_165100","165100","asset_current","Receivables due from employees in respect of travel advances","False","Terjatve do zaposlenih iz naslova predujmov za potne stroške"
"gd_acc_165200","165200","asset_current","Receivables from returned goods to suppliers","False","Terjatve iz naslova vrnjenega blaga dobaviteljem"
"gd_acc_165900","165900","asset_current","Other non-current receivables","False","Ostale nerazporeje kratkoročne terjatve"
"gd_acc_166000","166000","asset_current","Short-term receivables for VAT refunded to foreigners","False","Kratkoročne terjatve za DDV, vrnjen tujcem"
"gd_acc_167000","167000","asset_current","Short-term receivables for VAT paid abroad","False","Kratkoročne terjatve za DDV, plačan v tujini"
"gd_acc_169000","169000","asset_current","Impairment of other current receivables","False","Oslabitev vrednosti drugih kratkoročnih terjatev"
"gd_acc_170000","170000","asset_current","Short-term investments in shares of group entities, classified and measured at cost","False","Kratkoročne finančne naložbe v delnice in deleže organizacij v skupini, razporejene in izmerjene po nabavni vrednosti"
"gd_acc_173000","173000","asset_current","Short-term investments in shares of associates and jointly controlled entities, classified and measured at cost","False","Kratkoročne finančne naložbe v delnice in deleže pridruženih organizacijin skupaj obvladovanih organizacij, razporejene in izmerjene po nabavni vrednosti"
"gd_acc_176000","176000","asset_current","Other short-term investments classified and measured at cost","False","Druge kratkoročne finančne naložbe, razporejene in izmerjene po nabavni vrednosti"
"gd_acc_177000","177000","asset_current","Other short-term investments designated and measured at fair value through profit or loss","False","Druge kratkoročne finančne naložbe, razporejene in izmerjene po pošteni vrednosti prek poslovnega izida"
"gd_acc_178000","178000","asset_current","Other short-term investments classified and measured at fair value through equity or own funds","False","Druge kratkoročne finančne naložbe, razporejene in izmerjene po pošteni vrednosti prek kapitala oziroma lastnega vira sredstev"
"gd_acc_179000","179000","asset_current","Impairment of short-term investments","False","Oslabitev vrednosti kratkoročnih finančnih naložb"
"gd_acc_180000","180000","asset_current","Short-term loans granted under loan agreements to group organisations","False","Kratkoročna posojila, dana na podlagi posojilnih pogodb organizacijam v skupini"
"gd_acc_181000","181000","asset_current","Short-term loans granted under loan agreements to associates and jointly controlled entities","False","Kratkoročna posojila, dana na podlagi posojilnih pogodb pridruženim organizacijam in skupaj obvladovanim družbam"
"gd_acc_182000","182000","asset_current","Short-term loans to others, including short-term receivables from finance leases (leasing)","False","Kratkoročna posojila, dana drugim, vključno s kratkoročnimi terjatvami iz finančnega najema (leasing)"
"gd_acc_183000","183000","asset_current","Short-term deposits in banks and other financial organizations","False","Kratkoročni depoziti v bankah in drugih finančnih organizacijah"
"gd_acc_184000","184000","asset_current","Short-term loans granted by buying bonds","False","Dana kratkoročna posojila z odkupom obveznic"
"gd_acc_185000","185000","asset_current","Bills of exchange received","False","Prejete menice"
"gd_acc_186000","186000","asset_current","Short-term loans granted with the purchase of other debt securities","False","Dana kratkoročna posojila z odkupom drugih dolžniških vrednostnih papirjev"
"gd_acc_187000","187000","asset_current","Short-term unpaid called-up capital","False","Kratkoročno nevplačani vpoklicani kapital"
"gd_acc_189000","189000","asset_current","Short-term unpaid called-up capital","False","Oslabitev vrednosti kratkoročnih posojil"
"gd_acc_190000","190000","asset_current","Short-term deferred costs or expenses","False","Kratkoročno odloženi stroški oziroma odhodki"
"gd_acc_190200","190200","asset_current","Subscriptions","False","Naročnine"
"gd_acc_191000","191000","asset_current","Short-term unbilled revenue","False","Kratkoročno nezaračunani prihodki"
"gd_acc_192000","192000","asset_current","Valuables","False","Vrednotnice"
"gd_acc_195000","195000","asset_current","VAT on advances received","True","DDV od prejetih predujmov"
"gd_acc_210000","210000","liability_current","Liabilities included in disposal groups","False","Obveznosti, vključene v skupine za odtujitev"
"gd_acc_220000","220000","liability_payable","Short-term liabilities (debts) to suppliers in the country","True","Kratkoročne obveznosti (dolgovi) do dobaviteljev v državi"
"gd_acc_221000","221000","liability_payable","Short-term liabilities (debts) to suppliers abroad","True","Kratkoročne obveznosti (dolgovi) do dobaviteljev v tujini"
"gd_acc_222000","222000","liability_current","Short-term commodity credits received in the country","False","Kratkoročni blagovni krediti, prejeti v državi"
"gd_acc_223000","223000","liability_current","Short-term merchandise credits received abroad","False","Kratkoročni blagovni krediti, prejeti v tujini"
"gd_acc_224000","224000","liability_current","Short-term liabilities (debts) for unpaid goods and services SLO","False","Kratkoročne obveznosti (dolgovi) za nezaračunane blago in storitve SLO"
"gd_acc_224100","224100","liability_current","Short-term liabilities (debts) for unbilled goods and services of TUJ","False","Kratkoročne obveznosti (dolgovi) za nezaračunane blago in storitve TUJ"
"gd_acc_227000","227000","liability_current","Short-term business liabilities to companies in the group","False","Kratkoročne poslovne obveznosti do družb v skupini"
"gd_acc_230000","230000","liability_payable","Received short-term advances SLO","True","Prejeti kratkoročni predujmi SLO"
"gd_acc_230100","230100","liability_payable","Received short-term advances FOREIGN","True","Prejeti kratkoročni predujmi TUJ"
"gd_acc_231000","231000","liability_current","Received short-term securities","False","Prejete kratkoročne varščine"
"gd_acc_240000","240000","liability_current","Short-term liabilities from exports for a foreign account","False","Kratkoročne obveznosti iz izvoza za tuj račun"
"gd_acc_241000","241000","liability_current","Short-term liabilities to importers","False","Kratkoročne obveznosti do uvoznikov"
"gd_acc_242000","242000","liability_current","Short-term liabilities from commission and consignment sales","False","Kratkoročne obveznosti iz komisijske in konsignacijske prodaje"
"gd_acc_245000","245000","liability_current","Other short-term liabilities from operations for a foreign account","False","Druge kratkoročne obveznosti iz poslovanja za tuj račun"
"gd_acc_250000","250000","liability_current","Short-term liabilities for accrued and unpaid wages","False","Kratkoročne obveznosti za vračunane in neobračunane plače"
"gd_acc_251000","251000","liability_current","Short-term liabilities for net wages and salary compensation","False","Kratkoročne obveznosti za neto plače in nadomestila plač"
"gd_acc_252000","252000","liability_current","Short-term liabilities for social security contributions by types of contributions","False","Kratkoročne obveznosti za prispevke za socialno varnost po vrstah prispevkov"
"gd_acc_253000","253000","liability_current","Short-term liabilities for contributions from gross wages and salary compensation","False","Kratkoročne obveznosti za prispevke iz bruto plač in nadomestil plač"
"gd_acc_253042","253042","liability_current","Obligations for employment contributions. from the gross salary","False","Obveznosti za prispevke za zaposl. iz bruto plače"
"gd_acc_253043","253043","liability_current","Contribution obligations for the parent. var. from the gross salary","False","Obveznosti za prispevke za starš. var. iz bruto plače"
"gd_acc_253044","253044","liability_current","Obligations for ZPIZ contributions from gross salary","False","Obveznosti za prispevke ZPIZ iz bruto plače"
"gd_acc_253045","253045","liability_current","Obligations for ZZZS contributions from gross salary","False","Obveznosti za prispevke ZZZS iz bruto plače"
"gd_acc_254000","254000","liability_current","Short-term liabilities for taxes from gross wages and salary compensation - income tax","False","Kratkoročne obveznosti za davke iz bruto plač in nadomestil plač - dohodnina"
"gd_acc_255000","255000","liability_current","Short-term liabilities for other employment benefits","False","Kratkoročne obveznosti za druge prejemke iz delovnega razmerja"
"gd_acc_255100","255100","liability_current","Obligations to employees for cash material costs","False","Obveznosti do zaposlenih za gotovinske materialne stroške"
"gd_acc_255200","255200","liability_current","Short-term liabilities to employees for travel expenses","False","Kratkoročne obveznosti do zaposlenih za potne stroške"
"gd_acc_255300","255300","liability_current","Short-term liabilities to employees for food allowance","False","Kratkoročne obveznosti do zaposlenih za nadomestilo za prehrano"
"gd_acc_255400","255400","liability_current","Short-term liabilities to employees for reimbursement of transportation costs to work","False","Kratkoročne obveznosti do zaposlenih za povračilo stroškov prevoza na delo"
"gd_acc_255500","255500","liability_current","Short-term liabilities for holiday pay","False","Kratkoročne obveznosti za regres"
"gd_acc_255600","255600","liability_current","Short-term liabilities for severance pay","False","Kratkoročne obveznosti za odpravnino"
"gd_acc_256000","256000","liability_current","Short-term liabilities for contributions from other benefits from the employment relationship that are not calculated together with wages","False","Kratkoročne obveznosti za prispevke iz drugih prejemkov iz delovnega razmerja, ki se ne obračunavajo skupaj s plačami"
"gd_acc_257000","257000","liability_current","Short-term tax liabilities from other employment benefits that are not calculated together with wages","False","Kratkoročne obveznosti za davek iz drugih prejemkov iz delovnega razmerja, ki se ne obračunavajo skupaj s plačami"
"gd_acc_258000","258000","liability_current","Obligations for the payer's contributions","False","Obveznosti za prispevke izplačevalca"
"gd_acc_258042","258042","liability_current","Obligations for employment contributions. on gross salary","False","Obveznosti za prispevke za zaposl. na bruto plačo"
"gd_acc_258043","258043","liability_current","Contribution obligations for the parent. var. on gross salary","False","Obveznosti za prispevke za starš. var. na bruto plačo"
"gd_acc_258044","258044","liability_current","Obligations for ZPIZ contributions on gross salary","False","Obveznosti za prispevke ZPIZ na bruto plačo"
"gd_acc_258045","258045","liability_current","Obligations for ZZZS contributions on gross salary","False","Obveznosti za prispevke ZZZS na bruto plačo"
"gd_acc_260000","260000","liability_current","Output VAT - 22% - SLO BL ST","False","Izstopni DDV - 22% - SLO BL ST"
"gd_acc_260010","260010","liability_current","Output VAT - 9,5% - SLO BL ST","False","Izstopni DDV - 9,5% - SLO BL ST"
"gd_acc_260020","260020","liability_current","Output VAT - 5% - SLO BL ST","False","Izstopni DDV - 5% - SLO BL ST"
"gd_acc_260300","260300","liability_current","Output VAT - 22% - 3. world BL","False","Izstopni DDV - 22% - 3. svet BL"
"gd_acc_260301","260301","liability_current","Output VAT - 22% - 3. world ST","False","Izstopni DDV - 22% - 3. svet ST"
"gd_acc_260310","260310","liability_current","Output VAT - 9,5% - 3. world BL","False","Izstopni DDV - 9,5% - 3. svet BL"
"gd_acc_260311","260311","liability_current","Output VAT - 9,5% - 3. world ST","False","Izstopni DDV - 9,5% - 3. svet ST"
"gd_acc_260320","260320","liability_current","Output VAT - 5% 3. world BL in ST","False","Izstopni DDV - 5% 3. svet BL in ST"
"gd_acc_260400","260400","liability_current","Output VAT - 22% - EU BL","False","Izstopni DDV - 22% - EU BL"
"gd_acc_260410","260410","liability_current","Output VAT - 9,5% - EU BL","False","Izstopni DDV - 9,5% - EU BL"
"gd_acc_260420","260420","liability_current","Output VAT - 5% EU BL in ST","False","Izstopni DDV - 5% EU BL in ST"
"gd_acc_260430","260430","liability_current","Output VAT - 22% - EU ST","False","Izstopni DDV - 22% - EU ST"
"gd_acc_260440","260440","liability_current","Output VAT - 9,5% - EU ST","False","Izstopni DDV - 9,5% - EU ST"
"gd_acc_260460","260460","liability_current","Output VAT - 22% - SLO - SO 76.a Article BL ST","False","Izstopni DDV - 22% - SLO - SO 76.a člen BL ST"
"gd_acc_260470","260470","liability_current","Output VAT - 9,5% - SLO - SO 76.a Article BL ST","False","Izstopni DDV - 9,5% - SLO - SO 76.a člen BL ST"
"gd_acc_260500","260500","liability_current","Output VAT - 8% - SLO BL","False","Izstopni DDV - 8% - SLO BL"
"gd_acc_260700","260700","liability_current","Liability for VAT in Slovenia","False","Obveznost za DDV v Sloveniji"
"gd_acc_260800","260800","liability_current","Obligation to pay the difference between calculated and input VAT for the tax period in Slovenia","False","Obveznost za plačilo razlike med obračunanim in vstopnim DDV za davčno obdobje v Sloveniji"
"gd_acc_261000","261000","liability_current","Obligations for VAT, customs and other duties on imported goods","False","Obveznosti za DDV, carino in druge dajatve od uvoženega blaga"
"gd_acc_262000","262000","liability_current","Obligations for contributions","False","Obveznosti za prispevke"
"gd_acc_263000","263000","liability_current","Obligations for advance payment of income tax on income from activities","False","Obveznosti za akontacijo dohodnine od dohodka iz dejavnosti"
"gd_acc_264000","264000","liability_current","Corporate income tax obligations - DDPO","False","Obveznosti za davek od dohodkov pravnih oseb - DDPO"
"gd_acc_265000","265000","liability_current","Obligations for withholding tax","False","Obveznosti za davčni odtegljaj"
"gd_acc_265100","265100","liability_current","Obligations for withholding tax - Author's ch.","False","Obveznost za davčni odtegljaj - Avtorska pog."
"gd_acc_265200","265200","liability_current","Obligations for withholding tax - Temporary work of pensioners","False","Obveznost za davčni odtegljaj - Začasno delo upokojencev"
"gd_acc_265300","265300","liability_current","Obligations for withholding tax - Undertaking contract","False","Obveznost za davčni odtegljaj - Podjemna pogodba"
"gd_acc_265400","265400","liability_current","Obligations for withholding tax - Ch. about the power of attorney","False","Obveznost za davčni odtegljaj - Pog. o prokuri"
"gd_acc_265500","265500","liability_current","Obligations for withholding tax - Ch. about management","False","Obveznost za davčni odtegljaj - Pog. o poslovodenju"
"gd_acc_265600","265600","liability_current","Liability for withholding tax - Payment of profit","False","Obveznost za davčni odtegljaj - Izplačilo dobička"
"gd_acc_266000","266000","liability_current","Other short-term liabilities to state and other institutions","False","Druge kratkoročne obveznosti do državnih in drugih inštitucij"
"gd_acc_266044","266044","liability_current","Obligations for ZPIZ contributions from gross salary - contract","False","Obveznosti za prispevke ZPIZ iz bruto plače - pogodbe"
"gd_acc_266045","266045","liability_current","Obligations for ZZZS contributions from gross salary - contract","False","Obveznosti za prispevke ZZZS iz bruto plače - pogodbe"
"gd_acc_266100","266100","liability_current","Obligations for contributions from contracts of natural persons","False","Obveznosti za prispevke iz pogodb fizičnih oseb"
"gd_acc_266144","266144","liability_current","Obligations for ZPIZ contributions on gross wages - contracts","False","Obveznosti za prispevke ZPIZ na bruto plače - pogodbe"
"gd_acc_266145","266145","liability_current","Obligations for ZZZS contributions on gross wages - contracts","False","Obveznosti za prispevke ZZZS na bruto plače - pogodbe"
"gd_acc_267000","267000","liability_current","Short-term liabilities for social security contributions of sole proprietors","False","Kratkoročne obveznosti za prispevke za socialno varnost samostojnih podjetnikov"
"gd_acc_267042","267042","liability_current","Obligations for employment contributions. - SP","False","Obveznosti za prispevke za zaposl. - SP"
"gd_acc_267043","267043","liability_current","Contribution obligations for the parent. var. - SP","False","Obveznosti za prispevke za starš. var. - SP"
"gd_acc_267044","267044","liability_current","Obligations for ZPIZ - SP contributions","False","Obveznosti za prispevke ZPIZ - SP"
"gd_acc_267045","267045","liability_current","Obligations for ZZZS - SP contributions","False","Obveznosti za prispevke ZZZS - SP"
"gd_acc_270000","270000","liability_current","Short-term loans obtained from organizations in the group","False","Kratkoročna posojila, dobljena pri organizacijah v skupini"
"gd_acc_271000","271000","liability_current","Short-term loans obtained from affiliates and jointly controlled entities","False","Kratkoročna posojila, dobljena pri pridruženih organizacijah in skupaj obvladovanih organizacijah"
"gd_acc_272000","272000","liability_current","Short-term loans obtained from banks and organizations in the country","False","Kratkoročna posojila, dobljena pri bankah in organizacijah v državi"
"gd_acc_272100","272100","liability_current","Short-term loans obtained from banks in the country","False","Kratkoročna posojila, dobljena pri bankah  v državi"
"gd_acc_272200","272200","liability_current","Short-term loans obtained from companies in the country","False","Kratkoročna posojila, dobljena pri  družbah v državi"
"gd_acc_273000","273000","liability_current","Short-term loans obtained from banks abroad","False","Kratkoročna posojila, dobljena pri bankah v tujini"
"gd_acc_273100","273100","liability_current","Short-term loans obtained from companies abroad","False","Kratkoročna posojila, dobljena pri družbah v tujini"
"gd_acc_274000","274000","liability_current","Short-term financial liabilities related to bonds","False","Kratkoročne finančne obveznosti v zvezi z obveznicami"
"gd_acc_275000","275000","liability_current","Short-term liabilities from financial leasing (leasing)","False","Kratkoročne obveznosti iz finančnega najema (leasing)"
"gd_acc_276000","276000","liability_current","Short-term financial obligations to natural persons (loans)","False","Kratkoročne  finančne obveznosti do fizičnih oseb (posojila)"
"gd_acc_277000","277000","liability_current","Short-term liabilities related to the distribution of profit","False","Kratkoročne obveznosti v zvezi z razdelitvijo poslovnega izida"
"gd_acc_278000","278000","liability_current","Obligations from the payment of capital until entry in the court register","False","Obveznosti iz vplačila kapitala do vpisa v sodni register"
"gd_acc_279000","279000","liability_current","Other short-term financial liabilities","False","Druge kratkoročne finančne obveznosti"
"gd_acc_279100","279100","liability_current","Installment payments","False","Obročna plačila"
"gd_acc_280000","280000","liability_current","Short-term interest liabilities","False","Kratkoročne obveznosti za obresti"
"gd_acc_281000","281000","liability_current","Short-term bills payable","False","Kratkoročne menične obveznosti"
"gd_acc_282000","282000","liability_current","Short-term liabilities related to deductions from wages and compensation of wages to employees","False","Kratkoročne obveznosti v zvezi z odtegljaji od plač in nadomestili plač zaposlenim"
"gd_acc_285000","285000","liability_current","Other short-term business liabilities","False","Ostale kratkoročne poslovne obveznosti"
"gd_acc_285100","285100","asset_cash","Obligations to the credit card issuer (card payments)","False","Obveznosti do izdajatelja kreditne kartice (kartična plačila)"
"gd_acc_285200","285200","liability_current","Current liabilities for the payment of royalties","False","Kratkoročne obveznosti za izplačilo avtorskega honorarja"
"gd_acc_285300","285300","liability_current","Short-term obligations for payment to pupils and students on internships","False","Kratkoročne obveznosti za izplačilo dijakom in študentom na praksi"
"gd_acc_285400","285400","liability_current","Short-term liabilities for the payment of management contracts","False","Kratkoročne obveznosti za izplačilo poslovodskih pogodb"
"gd_acc_285500","285500","liability_current","Short-term liabilities for the payment of contracts","False","Kratkoročne obveznosti za izplačilo podjemnih pogodb"
"gd_acc_285600","285600","liability_current","Other short-term liabilities","False","Ostale kratkoročne obveznosti"
"gd_acc_290000","290000","liability_current","Charged costs or expenses in advance","False","Vnaprej vračunani stroški oziroma odhodki"
"gd_acc_291000","291000","liability_current","Short-term deferred revenue","False","Kratkoročno odloženi prihodki"
"gd_acc_295000","295000","liability_current","VAT on advances","True","DDV od danih predujmov"
"gd_acc_300000","300000","asset_current","The value of raw materials and materials according to suppliers' invoices","False","Vrednost surovin in materiala po obračunih dobaviteljev"
"gd_acc_301000","301000","asset_current","Dependent costs of procurement of raw materials and materials","False","Odvisni stroški nabave surovin in materiala"
"gd_acc_302000","302000","asset_current","Customs and other import duties on raw materials and materials","False","Carina in druge uvozne davščine od surovin in materiala"
"gd_acc_303000","303000","asset_current","VAT and other taxes on raw materials and materials","False","DDV in druge davščine od surovin in materiala"
"gd_acc_309000","309000","asset_current","Calculation of the purchase of raw materials and materials","False","Obračun nabave surovin in materiala"
"gd_acc_310000","310000","asset_current","Stocks of raw materials and materials in the warehouse","False","Zaloge surovin in materiala v skladišču"
"gd_acc_311000","311000","asset_current","Stocks of aggregates and material in a foreign warehouse","False","Zaloge sutovin in materiala v tujem skladišču"
"gd_acc_312000","312000","asset_current","Stocks of raw materials and materials on the way","False","Zaloge surovin in materiala na poti"
"gd_acc_316000","316000","asset_current","Stocks of raw materials and materials in finishing and processing","False","Zaloge surovin in materiala v dodelavi in predelavi"
"gd_acc_319000","319000","asset_current","Deviations from the constant prices of stocks of raw materials and materials","False","Odmiki od stalnih cen zalog surovin in materiala"
"gd_acc_320000","320000","asset_current","Stocks of petty inventory in the warehouse","False","Zaloge drobnega inventarja v skladišču"
"gd_acc_320100","320100","asset_current","Stocks of packaging in the warehouse","False","Zaloge embalaže v skladišču"
"gd_acc_321000","321000","asset_current","Petty inventory stocks put to use","False","Zaloge drobnega inventarja, dane v uporabo"
"gd_acc_321100","321100","asset_current","Stocks of packaging put into use","False","Zaloge embalaže, dane v uporabo"
"gd_acc_329000","329000","asset_current","Deviations from fixed prices of small inventory and packaging","False","Odmiki od stalnih cen drobnega inventarja in embalaže"
"gd_acc_400000","400000","expense","Material costs","False","Stroški materiala"
"gd_acc_401000","401000","expense","Cost of auxiliary material","False","Stroški pomožnega materiala"
"gd_acc_401100","401100","expense","Cost of cleaning products","False","Stroški čistil"
"gd_acc_402000","402000","expense","Energy costs - electricity","False","Stroški energije - elektrika"
"gd_acc_402100","402100","expense","Energy costs - gas","False","Stroški energije - plin"
"gd_acc_402200","402200","expense","Energy costs - fuel","False","Stroški energije - gorivo"
"gd_acc_403000","403000","expense","Cost of spare parts for fixed assets and maintenance materials for fixed assets","False","Stroški nadomestnih delov za osnovna sredstva in materiala za vzdrževanje osnovnih sredstev"
"gd_acc_404000","404000","expense","Write-off of small inventories","False","Odpis drobnega inventarja"
"gd_acc_404100","404100","expense","Packaging waste","False","Odpis embalaže"
"gd_acc_405000","405000","expense","Reconciliation of material and petty inventory costs due to inventory differences","False","Uskladitev stroškov materiala in drobnega inventarja zaradi popisnih razlik"
"gd_acc_406000","406000","expense","Cost of office supplies","False","Stroški pisarniškega materiala"
"gd_acc_406100","406100","expense","Costs of professional literature and publications","False","Stroški strokovne literature in publikacij"
"gd_acc_407000","407000","expense","Other material costs","False","Drugi stroški materiala"
"gd_acc_410000","410000","expense","Cost of services in the production of goods and services","False","Stroški storitev pri ustvarjanju proizvodov in opravljanju storitev"
"gd_acc_411100","411100","expense","Transport costs - Post and other carriers (DPD, FedEx, etc.)","False","Stroški transportnih storitev - Pošta in ostali prevozniki (DPD, FedEx itd.)"
"gd_acc_412000","412000","expense","Cost of maintenance services","False","Stroški storitev v zvezi z vzdrževanjem"
"gd_acc_413000","413000","expense","Rent expense - real estate","False","Strošek najemnine - nepremičnine"
"gd_acc_413100","413100","asset_prepayments","Rental cost - equipment","False","Strošek najemnine - oprema"
"gd_acc_414000","414000","expense","Reimbursement of work-related expenses to employees - without analytics","False","Povračila stroškov  zaposlencem v zvezi z delom - brez analitike"
"gd_acc_414110","414110","expense","Daily subsistence allowance while travelling on business - SLO","False","Dnevnica na službeni poti - SLO"
"gd_acc_414120","414120","expense","Mileage on business trips - SLO","False","Kilometrina na službeni poti - SLO"
"gd_acc_414130","414130","expense","Toll and parking - SLO","False","Cestnina in parkirnina - SLO"
"gd_acc_414140","414140","expense","Transport - Other - SLO","False","Prevoz - ostalo - SLO"
"gd_acc_414160","414160","expense","Other mission expenses - SLO","False","Ostali stroški službenega potovanja - SLO"
"gd_acc_414170","414170","expense","Employee reimbursements (excluding daily subsistence allowances and mileage) - SLO","False","Povračila zaposlenemu (razen dnevnice in kilometrine) - SLO"
"gd_acc_414210","414210","expense","Daily subsistence allowance while travelling on business - TUJ","False","Dnevnica na službeni poti - TUJ"
"gd_acc_414220","414220","expense","Mileage on business trips - TUJ","False","Kilometrina na službeni poti - TUJ"
"gd_acc_414230","414230","expense","Toll and parking - TUJ","False","Cestnina in parkirnina - TUJ"
"gd_acc_414240","414240","expense","Transport - Other - TUJ","False","Prevoz - ostalo - TUJ"
"gd_acc_414260","414260","expense","Other mission expenses - TUJ","False","Ostali stroški službenega potovanja - TUJ"
"gd_acc_414270","414270","expense","Employee reimbursements (excluding daily subsistence allowances and mileage) - SLO","False","Povračila zaposlenemu (razen dnevnice in kilometrine) - SLO"
"gd_acc_415000","415000","expense","Payment and banking costs, transaction costs and insurance premiums - excluding analytics","False","Stroški plačilnega prometa in bančnih storitev, stroški posla in zavarovalne premije - brez analitike"
"gd_acc_415100","415100","expense","Cost of banking services","False","Stroški bančnih storitev"
"gd_acc_415200","415200","expense","Insurance premium costs","False","Stroški zavarovalne premije"
"gd_acc_416000","416000","expense","Cost of intellectual and personal services - other","False","Stroški intelektualnih in osebnih storitev - drugo"
"gd_acc_416100","416100","expense","Cost of accounting services","False","Stroški računovodskih storitev"
"gd_acc_416200","416200","expense","Education costs","False","Stroški izobraževanja"
"gd_acc_416300","416300","expense","Outsourcing costs - SLO","False","Stroški storitev zunanjih izvajalcev (outsourcing) - SLO"
"gd_acc_416310","416310","expense","Outsourcing costs - TUJ","False","Stroški storitev zunanjih izvajalcev (outsourcing) - TUJ"
"gd_acc_416500","416500","expense","Cost of health services","False","Stroški zdravstvenih storitev"
"gd_acc_416600","416600","expense","Cost of other intellectual services","False","Stroški ostalih intelektualnih storitev"
"gd_acc_417000","417000","expense","Costs of fairs","False","Stroški sejmov"
"gd_acc_417100","417100","expense","National team costs","False","Stroški reprezentance"
"gd_acc_417200","417200","expense","Advertising and publicity costs","False","Stroški reklame in oglaševanja"
"gd_acc_417300","417300","expense","Sponsorship costs","False","Stroški sponzorstva"
"gd_acc_418000","418000","expense","Costs of services provided by natural persons not engaged in an activity, including charges to the organisation (labour contracts, author's contracts, emoluments to employees and other persons)","False","Stroški storitev fizičnih oseb, ki ne opravljajo dejavnosti, skupaj z dajatvami, ki bremenijo organizacijo (pogodbe o delu, avtorske pogodbe, sejnine zaposlenim in drugim osebam)"
"gd_acc_418100","418100","expense","Royalties","False","Avtorski honorar"
"gd_acc_418200","418200","expense","Freelance contract","False","Podjemna pogodba"
"gd_acc_418300","418300","expense","Students via Student Services","False","Študenti prek študentskega servisa"
"gd_acc_419000","419000","expense","Cost of other services - excluding analytics","False","Stroški drugih storitev - brez analitike"
"gd_acc_419100","419100","expense","Telephone costs","False","Stroški telefonije"
"gd_acc_419200","419200","expense","Internet and other related services costs","False","Stroški interneta in drugih povezanih storitev"
"gd_acc_419300","419300","expense","Utilities costs","False","Stroški komunalnih storitev"
"gd_acc_419400","419400","expense","Cost of membership fees","False","Stroški članarin"
"gd_acc_430000","430000","expense_depreciation","Amortisation of intangible assets","False","Amortizacija neopredmetenih sredstev"
"gd_acc_431000","431000","expense_depreciation","Depreciation of buildings","False","Amortizacija zgradb"
"gd_acc_432000","432000","expense_depreciation","Depreciation of equipment and spare parts","False","Amortizacija opreme in nadomestnih delov"
"gd_acc_433000","433000","expense_depreciation","Depreciation of small inventories","False","Amortizacija drobnega inventarja"
"gd_acc_434000","434000","expense_depreciation","Depreciation of other property, plant and equipment","False","Amortizacija drugih opredmetenih osnovnih sredstev"
"gd_acc_435000","435000","expense_depreciation","Depreciation of investment property","False","Amortizacija naložbenih nepremičnin"
"gd_acc_436000","436000","expense_depreciation","Depreciation of biological assets","False","Amortizacija bioloških sredstev"
"gd_acc_440000","440000","expense","Provisions for corporate reorganisation costs","False","Rezervacije za stroške reorganizacije podjetja"
"gd_acc_441000","441000","expense","Provisions for guarantees given","False","Rezervacije za dana jamstva"
"gd_acc_442000","442000","expense","Provisions for losses on onerous contracts","False","Rezervacije za pokrivanje izgub iz kočljivih pogodb"
"gd_acc_449000","449000","expense","Rezervacije za izgube iz škodljivih pogodb","False","Rezervacije za pokrivanje drugih obveznosti iz preteklega poslovanja"
"gd_acc_450000","450000","expense","Credit interest costs","False","Stroški obresti kredita"
"gd_acc_450100","450100","expense","Interest cost of the limit","False","Stroški obresti limita"
"gd_acc_470000","470000","expense","Staff salaries","False","Plače zaposlenih"
"gd_acc_470100","470100","expense","Bonitures","False","Bonitete"
"gd_acc_471000","471000","expense","Employee salary allowances","False","Nadomestila plač zaposlencev"
"gd_acc_472000","472000","expense","Employee supplementary pension scheme costs","False","Stroški dodatnega pokojninskega zavarovanja zaposlenih"
"gd_acc_473000","473000","expense","Annual leave, bonuses, allowances and other staff emoluments - without analytics","False","Regres za letni dopust, bonitete, povračila  in drugi prejemki zaposlenih - brez analitike"
"gd_acc_473100","473100","expense","Commuting costs","False","Stroški prevoza na delo"
"gd_acc_473200","473200","expense","Meal allowance costs","False","Stroški nadomestila za prehrano"
"gd_acc_473300","473300","expense","Recourse costs","False","Stroški regresa"
"gd_acc_473400","473400","expense","Cost of severance payments","False","Stroški odpravnin"
"gd_acc_473500","473500","expense","Cost of the Jubilee Prize","False","Stroški jubilejne nagrade"
"gd_acc_474000","474000","expense","Employer's contributions on wages, salaries, wage allowances, bonuses, reimbursements and other employee benefits - without analytics","False","Delodajalčevi prispevki od plač, nadomestil plač bonitet, povračil in drugih prejemkov zaposlencev - brez analitike"
"gd_acc_474042","474042","expense","Gross cost of NA contributions - for employment","False","Stroški prispevkov NA bruto - za zaposlovanje"
"gd_acc_474043","474043","expense","Gross cost of NA contributions - for parental care","False","Stroški prispevkov NA bruto - za starševsko varstvo"
"gd_acc_474044","474044","expense","Gross cost of NA contributions - Pensions","False","Stroški prispevkov NA bruto - ZPIZ"
"gd_acc_474045","474045","expense","Gross cost of NA contributions - ZZZS","False","Stroški prispevkov NA bruto - ZZZS"
"gd_acc_475000","475000","expense","Other employers' levies on wages, salaries, bonuses, allowances and other employee benefits","False","Druge delodajalčeve dajatve od plač, bonitet, povračil in drugih prejemkov zaposlenih"
"gd_acc_476000","476000","expense","Apprentices' bonuses, including taxes charged to the company","False","Nagrade vajencem skupaj z dajatvami, ki bremenijo podjetje"
"gd_acc_477000","477000","expense","Management expenses accounted for on a basis such as employment","False","Stroški poslovodstva, obračunani na podlagi kot je delovno razmerje"
"gd_acc_478000","478000","expense","Other labour costs","False","Drugi stroški dela"
"gd_acc_479000","479000","expense","Provisions for pensions, gratuities and retirement benefits","False","Rezervacije za pokojnine, jubilejne nagrade in odpravnine ob upokojitvi"
"gd_acc_480000","480000","expense","Charges that do not depend on labour costs or other types of costs","False","Dajatve, ki niso odvisne od stroškov dela ali drugih vrst stroškov"
"gd_acc_480100","480100","expense","Cost of compensation for the use of building land","False","Stroški nadomestila za uporabo stavbnega zemljišča"
"gd_acc_481000","481000","expense","Environmental protection expenditure","False","Izdatki za varstvo okolja"
"gd_acc_482000","482000","expense","Awards to students on work placements, including taxes","False","Nagrade dijakom in študentom na delovni praksi skupaj z dajatvami"
"gd_acc_483000","483000","expense","Scholarships for pupils and students","False","Štipendije dijakom in študentom"
"gd_acc_484000","484000","expense","Social security contributions for sole traders","False","Prispevki za socialno varnost samostojnih podjetnikov"
"gd_acc_484100","484100","expense","Other costs incurred by the entrepreneur","False","Ostali stroški podjetnika"
"gd_acc_486000","486000","expense","Reimbursement of s.p. expenses","False","Povračila stroškov s.p."
"gd_acc_488000","488000","expense","Grants to other associations and legal persons (donations)","False","Dotacije drugim društvom in pravnim osebam (donacije)"
"gd_acc_489000","489000","expense","Other costs","False","Ostali stroški"
"gd_acc_489100","489100","expense","Cost of reminders","False","Stroški opominov"
"gd_acc_489200","489200","expense","Penalty costs","False","Stroški kazni"
"gd_acc_489300","489300","expense","Other non-tax deductible costs","False","Ostali davčno nepriznani stroški"
"gd_acc_490000","490000","expense","Transfer of costs to inventories","False","Prenos stroškov v zaloge"
"gd_acc_491000","491000","expense","Transferring costs directly to expenditure","False","Prenos stroškov neposredno v odhodke"
"gd_acc_500001","500001","expense","VAT only - EUL - deduction","False","Samo DDV - EUL - odbitek"
"gd_acc_500002","500002","expense","Imports of goods 3rd world","False","Uvoz blaga 3. svet"
"gd_acc_500100","500100","expense","Intermediate account of circular and chain offsets","False","Vmesni konto krožnih in verižnih kompenzacij"
"gd_acc_500300","500300","expense","Opening of stocks","False","Otvoritev zalog"
"gd_acc_599999","599999","expense","Interim account of the basis of the advance","True","Vmesni konto osnove avansa"
"gd_acc_600000","600000","asset_current","Work in progress","False","Nedokončana proizvodnja"
"gd_acc_601000","601000","asset_current","Services in progress","False","Nedokončane storitve"
"gd_acc_602000","602000","asset_current","Semi-finished products","False","Polizdelki"
"gd_acc_604000","604000","asset_current","Production in finishing and processing","False","Proizvodnja v dodelavi in predelavi"
"gd_acc_609000","609000","asset_current","Price deviations for work in progress and services","False","Odmiki od cen nedokončane proizvodnje in storitev"
"gd_acc_610000","610000","asset_current","Harvested crops at fair value","False","Pospravljeni pridelki po pošteni vrednosti"
"gd_acc_618000","618000","asset_current","Harvested crops at cost","False","Pospravljeni pridelki po nabavni vrednosti"
"gd_acc_619000","619000","asset_current","Deviations from crop prices","False","Odmiki od cen pridelkov"
"gd_acc_630000","630000","asset_current","Products in own storage","False","Proizvodi v lastnem skladišču"
"gd_acc_631000","631000","asset_current","Products in foreign storage","False","Proizvodi v tujem skladišču"
"gd_acc_632000","632000","asset_current","Products on the move","False","Proizvodi na poti"
"gd_acc_633000","633000","asset_current","Products in your own shop","False","Proizvodi v lastni prodajalni"
"gd_acc_634000","634000","asset_current","VAT accrued on products in the shop","False","Vračunani DDV od proizvodov v prodajalni"
"gd_acc_635000","635000","asset_current","Products undergoing finishing and processing","False","Proizvodi v dodelavi in predelavi"
"gd_acc_638000","638000","asset_current","Biological agents that are stocks","False","Biološka sredstva, ki so zaloge"
"gd_acc_639000","639000","asset_current","Product price deviations","False","Odmiki od cen proizvodov"
"gd_acc_650000","650000","asset_current","Value of goods as invoiced by suppliers (cost)","False","Vrednost blaga po obračunih dobaviteljev (nabavna vrednost)"
"gd_acc_651000","651000","asset_current","Dependent costs of purchase of goods","False","Odvisni stroški nabave blaga"
"gd_acc_652000","652000","asset_current","Customs duty and other import taxes on goods","False","Carina in druge uvozne davščine od blaga"
"gd_acc_653000","653000","asset_current","VAT and other taxes on goods","False","DDV in druge davščine od blaga"
"gd_acc_659000","659000","asset_current","Accounting for the purchase of goods","False","Obračun nabave blaga"
"gd_acc_660000","660000","asset_current","Goods in your own warehouse","False","Blago v lastnem skladišču"
"gd_acc_661000","661000","asset_current","Goods in a foreign warehouse","False","Blago v tujem skladišču"
"gd_acc_662000","662000","asset_current","Goods on the move","False","Blago na poti"
"gd_acc_663000","663000","asset_current","Goods in your own shop","False","Blago v lastni prodajalni"
"gd_acc_664000","664000","asset_current","VAT included in stocks of goods - 22%","False","DDV, vračunan v zalogah blaga - 22%"
"gd_acc_664010","664010","asset_current","VAT included in stocks of goods - 9.5%","False","DDV, vračunan v zalogah blaga - 9,5%"
"gd_acc_664020","664020","asset_current","VAT included in the stock of goods - 5%","False","DDV, vračunan v zalogah blaga - 5%"
"gd_acc_669000","669000","asset_current","Accumulated difference in prices of inventories of goods","False","Vračunana razlika v cenah zalog blaga"
"gd_acc_670000","670000","asset_current","Property, plant and equipment held for sale","False","Opredmetena osnovna sredstva, namenjena prodaji"
"gd_acc_671000","671000","asset_current","Investment property valued under the cost model held for sale","False","Naložbene nepremičnine, vrednotene po modelu nabavne vrednosti, namenjene prodaji"
"gd_acc_672000","672000","asset_current","Other non-current assets held for sale","False","Druga nekratkoročna sredstva, namenjena prodaji"
"gd_acc_700000","700000","expense","Value of business effects sold","False","Vrednost prodanih poslovnih učinkov"
"gd_acc_701000","701000","expense","Value of capitalised own products and services","False","Vrednost usredstvenih lastnih proizvodov in storitev"
"gd_acc_702000","702000","expense","Cost of materials and goods sold","False","Nabavna vrednost prodanih materiala in blaga"
"gd_acc_703000","703000","expense","Other operating expenses","False","Drugi poslovni odhodki"
"gd_acc_704000","704000","expense","Expenditure from the evaluation of biological centres and harvesting of agricultural crops","False","Odhodki iz vrednotenja bioloških stredstev in pospravitev kmetijskih pridelkov"
"gd_acc_710000","710000","expense","Value of business effects sold","False","Vrednost prodanih poslovnih učinkov"
"gd_acc_711000","711000","expense","Cost of materials and goods sold","False","Nabavna vrednost prodanih materiala in blaga"
"gd_acc_712000","712000","expense","Selling costs","False","Stroški prodajanja"
"gd_acc_713000","713000","expense","General operating expenses (procurement and administration)","False","Stroški splošnih dejavnosti (nabave in uprave)"
"gd_acc_714000","714000","expense","Other costs not held in inventories","False","Drugi stroški, ki se ne zadržujejo v zalogah"
"gd_acc_720000","720000","expense","Revaluation operating expenses relating to intangible assets and property, plant and equipment and investment property","False","Prevrednotovalni poslovni odhodki v zvezi z neopredmetenimi sredstvi in opredmetenimi osnovnimi sredstvi in naložbenimi nepremičninami"
"gd_acc_721000","721000","expense","Revaluation operating expenses related to inventories","False","Prevrednotovalni poslovni odhodki v zvezi z zalogami"
"gd_acc_722000","722000","expense","Revaluation operating expenses as a result of impairment revaluation related to trade receivables","False","Prevrednotovalni poslovni odhodki kot posledica prevrednotenja zaradi oslabitve v zvezi z poslovnimi terjatvami"
"gd_acc_723000","723000","expense","Revaluation operating expenses as a result of write-offs related to trade receivables","False","Prevrednotovalni poslovni odhodki kot posledica odpisov v zvezi s poslovnimi terjatvami"
"gd_acc_724000","724000","expense","Revaluation operating expenses relating to current assets other than financial investments","False","Prevrednotovalni poslovni odhodki v zvezi s kratkoročnimi sredstvi, razen finančnimi naložbami"
"gd_acc_740000","740000","expense_direct_cost","Expenditure on loans received from group organisations - net of analytics","False","Odhodki iz posojil, prejetih od organizacij v skupini - brez analitike"
"gd_acc_740100","740100","expense_direct_cost","Contractual interest - corporate loans","False","Pogodbene obresti - posojila družb"
"gd_acc_740200","740200","expense_direct_cost","Interest on leases - loans from companies","False","Obresti leasinga - posojila družb"
"gd_acc_740300","740300","expense_direct_cost","Interest on overdue loans - corporate loans","False","Zamudne obresti - posojila družb"
"gd_acc_741000","741000","expense_direct_cost","Expenditure on loans received from banks - excluding analytics","False","Odhodki iz posojil, prejetih od bank - brez analitike"
"gd_acc_741100","741100","expense_direct_cost","Contractual interest - bank loans","False","Pogodbene obresti - posojila bank"
"gd_acc_741200","741200","expense_direct_cost","Lease interest - bank loans","False","Obresti leasinga - posojila bank"
"gd_acc_741300","741300","expense_direct_cost","Interest on overdue loans - bank loans","False","Zamudne obresti - posojila bank"
"gd_acc_742000","742000","expense_direct_cost","Expenditure on bonds issued","False","Odhodki iz izdanih obveznic"
"gd_acc_743000","743000","expense_direct_cost","Expenditure on other financial liabilities","False","Odhodki iz drugih finančnih obveznosti"
"gd_acc_744000","744000","expense_direct_cost","Expenditure on payables to group organisations","False","Odhodki iz poslovnih obveznosti do organizacij v skupini"
"gd_acc_745000","745000","expense_direct_cost","Expenditure on trade payables and bills of exchange","False","Odhodki iz obveznosti do dobaviteljev in meničnih obveznosti"
"gd_acc_745100","745100","expense_direct_cost","Interest on late payment of payables to suppliers","False","Zamudne obresti iz zamudnega plačila obveznosti do dobaviteljev"
"gd_acc_746000","746000","expense_direct_cost","Expenditure on other payables, including interest expense on recalculated retirement benefits","False","Odhodki iz drugih poslovnih obveznosti, vključno za stroške obresti iz naslova preračunanih odpravnin ob upokojitvi"
"gd_acc_746100","746100","expense_direct_cost","Interest on late payment of liabilities to the State (taxes and contributions)","False","Zamudne obresti iz zamudnega plačila obveznosti do države (davki in prispevki)"
"gd_acc_746200","746200","expense","Negative exchange differences","False","Negativne tečajne razlike"
"gd_acc_747000","747000","expense_direct_cost","Expenditure on assets designated at fair value through profit or loss and expenditure on the valuation of investment property at fair value through profit or loss","False","Odhodki iz sredstev, razporejenih po pošteni vrednosti prek poslovnega izida in odhodki iz vrednotenja naložbenih nepremičnin po poštevni vrednosti"
"gd_acc_748000","748000","expense_direct_cost","Impairment losses on financial investments","False","Odhodki iz oslabitve finančnih naložb"
"gd_acc_749000","749000","expense_direct_cost","Derecognition expense on investments and investment property measured at fair value","False","Odhodki iz odprave pripoznanja finančnih naložb in naložbenih nepremičnin, izmerjenih po pošteni vrednosti"
"gd_acc_752000","752000","expense_direct_cost","Penalties not related to operating effects","False","Denarne kazni, ki niso povezane s poslovnimi učinki"
"gd_acc_753000","753000","expense_direct_cost","Compensation not related to operating effects","False","Odškodnine, ki niso povezane s poslovnimi učinki"
"gd_acc_754000","754000","expense_direct_cost","Donations","False","Donacije"
"gd_acc_755000","755000","expense_direct_cost","Subsidies, grants and similar non-operating expenditure","False","Subvencije, dotacije in podobni odhodki, ki niso povezani s poslovnimi učinki"
"gd_acc_758000","758000","expense_direct_cost","Negative euro offsets","False","Negativne evrske izravnave"
"gd_acc_759000","759000","expense_direct_cost","Other non-operating expenses","False","Ostali odhodki, ki niso povezani s poslovnimi učinki"
"gd_acc_760000","760000","income","Revenue from the sale of products and services on the domestic market","False","Prihodki od prodaje proizvodov in storitev na domačem trgu"
"gd_acc_761000","761000","income","Revenue from the sale of products and services on foreign markets - EU","False","Prihodki od prodaje proizvodov in storitev na tujem trgu - EU"
"gd_acc_761200","761200","income","Revenue from the sale of products and services on foreign markets - 3rd World","False","Prihodki od prodaje proizvodov in storitev na tujem trgu - 3. svet"
"gd_acc_762000","762000","income","Revenue from the sale of merchandise and supplies on the domestic market","False","Prihodki od prodaje trgovskega blaga in materiala na domačem trgu"
"gd_acc_763000","763000","income","Revenue from the sale of merchandise and supplies on foreign markets - EU","False","Prihodki od prodaje trgovskega blaga in materiala na tujem trgu - EU"
"gd_acc_763200","763200","income","Revenue from the sale of merchandise and supplies on foreign markets - 3rd World","False","Prihodki od prodaje trgovskega blaga in materiala na tujem trgu - 3. svet"
"gd_acc_764000","764000","income","Revenue from the valuation of biological products and the harvesting of agricultural crops","False","Prihodki iz vrednotenja bioloških sredstev in pospravitve kmetijskih pridelkov"
"gd_acc_765000","765000","income","Rental income","False","Prihodki od najemnin"
"gd_acc_766000","766000","income","Proceeds from the reversal of provisions and accrued charges or expenses","False","Prihodki od odprave rezervacij in PČR na račun vnaprej vračunanih stroškov oziroma odhodkov"
"gd_acc_767000","767000","income","Gains on business combinations (revaluation surplus - bad name)","False","Prihodki od poslovnih združitev (presežek iz prevrednotenja   slabo ime)"
"gd_acc_768000","768000","income","Other income related to operating effects (subsidies, grants, recourses, offsets, premiums, etc.)","False","Drugi prihodki, povezani s poslovnimi učinki (subvencije, dotacije, regresi, kompenzacije, premije ...)"
"gd_acc_769000","769000","income","Revaluation operating income","False","Prevrednotovalni poslovni prihodki"
"gd_acc_770000","770000","income_other","Financial income from interests in group organisations","False","Finančni prihodki iz deležev v organizacij v skupini"
"gd_acc_771000","771000","income_other","Financial income from interests in associates and jointly controlled entities","False","Finančni prihodki iz deležev v pridruženih organizacij in skupaj obvladovanih organizacij"
"gd_acc_772000","772000","income_other","Financial income from shareholdings in other organisations","False","Finančni prihodki iz deležev v drugih organizacij"
"gd_acc_773000","773000","income_other","Financial income from other investments","False","Finančni prihodki iz drugih naložb"
"gd_acc_774000","774000","income_other","Financial income from loans granted to group organisations","False","Finančni prihodki iz posojil, danih organizacij v skupini"
"gd_acc_775000","775000","income_other","Financial income from loans to others (including deposits)","False","Finančni prihodki iz posojil danih drugim( tudi od depozitov)"
"gd_acc_776000","776000","income_other","Financial income from trade receivables from group organisations","False","Finančni prihodki iz poslovnih terjatev do organizacij v skupini"
"gd_acc_777000","777000","income_other","Financial income from trade receivables due from others","False","Finančni prihodki iz poslovnih terjatev do drugih"
"gd_acc_777100","777100","income_other","Interest on creditors' claims","False","Zamudne obresti iz terjatev upnikov"
"gd_acc_777200","777200","income_other","Positive exchange rate differences","False","Pozitivne tečajne razlike"
"gd_acc_778000","778000","income_other","Financial income from financial assets designated at fair value through profit or loss","False","Finančni prihodki iz finančnih sredstev, razporejenih po pošteni vrednosti prek poslovnega izida"
"gd_acc_779000","779000","income_other","Financial income from the valuation of investment property at fair value and income from the disposal of investment property at fair value","False","Finančni prihodki iz vredotenja naložbenih nepremičnin po poštevi vrednosti in prihodki od odtujitve naložbenih nepremičnin po pošteni vrednosti"
"gd_acc_784000","784000","income_other","Donations","False","Donacije"
"gd_acc_785000","785000","income_other","Subsidies, grants and similar non-operating income","False","Subvencije, dotacije in podobni prihodki, ki niso povezani s poslovnimi učinki"
"gd_acc_786000","786000","income_other","Compensation not related to operating effects","False","Odškodnine, ki niso povezane s poslovnimi učinki"
"gd_acc_787000","787000","income_other","Penalties not related to business effects","False","Kazni, ki niso povezane s poslovnimi učinki"
"gd_acc_788000","788000","income_other","Positive euro offsets","False","Pozitivne evrske izravnave"
"gd_acc_789000","789000","income_other","Other non-operating income","False","Ostali prihodki, ki niso povezani s poslovnimi učinki"
"gd_acc_790000","790000","income","Capitalised own products and own services","False","Usredstveni lastni proizvodi in lastne storitve"
"gd_acc_794000","794000","income","Inventory surpluses","False","Inventurni presežki"
"gd_acc_800000","800000","equity_unaffected","Profit or loss before tax or surplus on income or surplus on expenses","False","Dobiček ali izguba pred obdavčitvijo oziroma presežek prihodkov ali presežek odhodkov"
"gd_acc_801000","801000","off_balance","Establishing and transferring s.p. income","False","Ugotovitev in prenos dohodka s.p."
"gd_acc_803000","803000","off_balance","Identification and transfer of a negative profit or loss","False","Ugotovitev in prenos negativnega poslovnega izida"
"gd_acc_804000","804000","off_balance","Surplus revenue from previous years to cover the surplus of the period's revenue","False","Presežek prihodkov iz prejšnjih let, namenjen pokrivanju presežka odgodkov obračunskega obdobja"
"gd_acc_805000","805000","off_balance","Surplus expenditure from previous years to be charged to the revenue surplus of the accounting period","False","Presežek odhodkov iz prejšnjih let, ki se pokriva iz presežka prihodkov obračunskega obdobja"
"gd_acc_806000","806000","off_balance","Total revenue surplus or total expenditure surplus","False","Celoten presežek prihodkov ali celotni presežek odhodkov"
"gd_acc_810000","810000","off_balance","Corporate income tax V.I.T.T.","False","Davek od dohodkov pravnih oseb DDPO"
"gd_acc_812000","812000","off_balance","Other taxes not shown under other headings","False","Drugi davki, ki niso izkazani v drugih postavkah"
"gd_acc_813000","813000","off_balance","Deferred tax income (expense)","False","Prihodki (odhodki) iz naslova odloženega davka"
"gd_acc_815000","815000","off_balance","Net profit for the year or net revenue surplus","False","Čisti dobiček poslovnega leta oziroma čisti presežek prihodkov"
"gd_acc_820000","820000","off_balance","Net profit to cover losses carried forward","False","Čisti dobiček za kritje prenesenih izgub"
"gd_acc_821000","821000","off_balance","Net profit for legal reserve formation","False","Čisti dobiček za oblikovanje zakonskih rezerv"
"gd_acc_822000","822000","off_balance","Net profit for the creation of reserves for own shares or participating interests","False","Čisti dobiček za oblikovanje rezerv za lastne delnice oziroma deleže"
"gd_acc_823000","823000","off_balance","Net profit for building up statutory reserves","False","Čisti dobiček za oblikovanje statutarnih rezerv"
"gd_acc_824000","824000","off_balance","Net profit for other profit reserves","False","Čisti dobiček za druge rezerve iz dobička"
"gd_acc_825000","825000","off_balance","Net revenue surplus earmarked for a specific purpose (fund)","False","Čisti presežek prihodkov za določen namen (sklad)"
"gd_acc_829000","829000","off_balance","Carry forward of unused part of net profit or net revenue surplus for the financial year","False","Prenos neuporabljenega dela čistega dobička oziroma čistega presežka prihodkov poslovnega leta"
"gd_acc_890000","890000","off_balance","Net loss for the year or net excess of expenses","False","Čista izguba tekočega leta oziroma čisti presežek odhodkov"
"gd_acc_899000","899000","off_balance","Carry-forward of current year losses or carry-forward of net excess expenditure","False","Prenosčiste  izgube tekočega leta oziroma prenos čistega presežka odhodkov"
"gd_acc_900000","900000","equity","Basic share capital - shares","False","Osnovni delniški kapital  - delnice"
"gd_acc_901000","901000","equity","Share capital - capital shares or capital contribution","False","Osnovni kapital - kapitalski deleži ali kapitalska vloga"
"gd_acc_902000","902000","equity","Start-up capital for s.p.","False","Začetni kapital s.p."
"gd_acc_905000","905000","equity","Incompatible cooperative capital","False","Nerazdružljivi zadružni kapital"
"gd_acc_906000","906000","equity","Cooperative members' shares","False","Deleži članov zadruge"
"gd_acc_907000","907000","equity","Ustanovitvena in kasnejše vloge","False","Ustanovitvena in kasnejše vloge"
"gd_acc_909000","909000","equity","Uncalled capital (deductible item)","False","Nevpoklicani kapital (odbitna postavka)"
"gd_acc_910000","910000","equity","Contributions in excess of the minimum issue amounts of shares (paid-in capital surplus)","False","Vplačila nad najmanjšimi emisijskimi zneski delnic oziroma deležev (vplačani presežek kapitala)"
"gd_acc_911000","911000","equity","Payments in excess of the carrying amount on disposal of temporarily redeemed own shares or interests","False","Vplačila nad knjigovodsko vrednostjo pri odtujitvi začasno odkupljenih lastnih delnic oziroma deležev"
"gd_acc_912000","912000","equity","Contributions in excess of the minimum issued amount of capital raised through the issue of convertible bonds or bonds with a share purchase option","False","Vplačila nad najmanjšim emisijskim zneskom kapitala, pridobljena z izdajo zamenljivih obveznic ali obveznic z delniško nakupno opcijo"
"gd_acc_913000","913000","equity","Contributions to acquire additional rights attached to shares or interests","False","Vplačila za pridobitev dodatnih pravic iz delnic oziroma deležev"
"gd_acc_914000","914000","equity","Other contributions of capital under the Statutes","False","Druga vplačila kapitala na podlagi statuta"
"gd_acc_915000","915000","equity","Amounts arising from simplified reduction of share capital by withdrawal of shares or interests","False","Zneski iz poenostavljenega zmanjšanja osnovnega kapitala z umikom delnic oziroma deležev"
"gd_acc_916000","916000","equity","General capital revaluation adjustment and amounts transferred from the revaluation reserve","False","Splošni prevrednotovalni popravek kapitala in zneski, preneseni iz revalorizacijske rezerve"
"gd_acc_917000","917000","equity","Amounts arising from the effects of a confirmed compulsory arrangement","False","Zneski iz učinkov potrjene prisilne poravnave"
"gd_acc_918000","918000","equity","Transfers of immovable property in the course of business","False","Prenosi stvarnega premoženja med opravljanjem dejavnosti"
"gd_acc_919000","919000","equity","Inflows and outflows between business and household","False","Pritoki in odtoki med podjetjem in gospodinjstvom"
"gd_acc_920000","920000","equity","Legal reserves","False","Zakonske rezerve"
"gd_acc_921000","921000","equity","Reserves for own shares or own interests","False","Rezerve za lastne delnice oziroma lastne poslovne deleže"
"gd_acc_922000","922000","equity","Statutory reserves","False","Statutarne rezerve"
"gd_acc_923000","923000","equity","Other profit reserves","False","Druge rezerve iz dobička"
"gd_acc_924000","924000","equity","Voluntary cooperative reserves","False","Prostovoljne zadružne rezerve"
"gd_acc_925000","925000","equity","Voluntary cooperative funds","False","Prostovoljne zadružni skladi"
"gd_acc_926000","926000","equity","Part of net revenue surplus earmarked for a specific purpose (fund)","False","Del čistega presežka prihodkov za določen namen (sklad)"
"gd_acc_929000","929000","equity","Own shares or own shares acquired (deductible item)","False","Pridobljene lastne delnice oziroma lastni poslovni deleži (odbitna postavka)"
"gd_acc_930000","930000","equity","Net profit carried forward from previous years","False","Preneseni čisti dobiček iz prejšnjih let"
"gd_acc_931000","931000","equity","Net loss carried forward from previous years","False","Prenesena čista izguba iz prejšnjih let"
"gd_acc_932000","932000","equity","Unused part of net profit for the year or unallocated net revenue surplus","False","Neuporabljeni del čistega dobička poslovnega leta oziroma nerazporejeni čisti presežek prihodkov"
"gd_acc_933000","933000","equity","Net loss for the year or net excess of expenses","False","Čista izguba poslovnega leta oziroma čisti presežek odhodkov"
"gd_acc_935000","935000","equity","Income s.p.","False","Dohodek s.p."
"gd_acc_937000","937000","equity","Negative profit/loss of s.p.","False","Negativni poslovni izid s.p."
"gd_acc_940000","940000","equity","Land revaluation reserve","False","Revalorizacijske rezerve iz prevrednotovanja zemljišč"
"gd_acc_941000","941000","equity","Revaluation reserves on revaluation of buildings","False","Revalorizacijske rezerve iz prevrednotovanja zgradb"
"gd_acc_949000","949000","equity","Adjustment to the revaluation reserve for deferred tax liabilities","False","Popravek vrednosti revaloriziacijskih rezerv za odložene obveznosri za davek"
"gd_acc_954000","954000","equity","Reserves arising from the valuation of long-term investments at fair value","False","Rezerve, nastale zaradi vrednotenja dolgoročnih finančnih naložb po pošteni vrednosti"
"gd_acc_955000","955000","equity","Reserves arising from the valuation of short-term investments at fair value","False","Rezerve, nastale zaradi vrednotenja kratkoročnih finančnih naložb po pošteni vrednosti"
"gd_acc_956000","956000","equity","Amounts of the demonstrated gain or demonstrated loss from a change in the fair value of available-for-sale financial assets that is not part of a hedging relationship","False","Zneski dokazanega dobička ali dokazane izgube iz spremembe poštene vrednosti finančnih sredstev, razpoložljivih za prodajo, ki ni del razmerja pri varovanju vrednosti"
"gd_acc_957000","957000","equity","Actuarial gains or losses on defined benefits","False","Aktuarski dobički ali izgube od določenih zaslužkov"
"gd_acc_959000","959000","equity","Fair value reserve valuation allowance for deferred tax liabilities","False","Popravek vrednosti rezerv, nastalih zaradi vrednotenja po pošteni vrednosti za odložene obveznosti za davek"
"gd_acc_960000","960000","equity","Provisions for reorganisation costs","False","Rezervacije za stroške reorganizacije organizacije"
"gd_acc_961000","961000","equity","Provisions for future costs or expenses arising from decommissioning, restoration and other similar provisions","False","Rezervacije za pokrivanje prihodnjih stroškov oziroma odhodkov zaradi razgradnje, ponovne vzpostavitve prvotnega stanja in druge podobne rezervacije"
"gd_acc_962000","962000","equity","Provisions for onerous contracts","False","Rezervacije za kočljive pogodbe"
"gd_acc_963000","963000","equity","Provisions for pensions, gratuities and retirement benefits","False","Rezervacije za pokojnine, jubilejne nagrade in odpravnine ob upokojitvi"
"gd_acc_964000","964000","equity","Provisions for guarantees given","False","Rezervacije za dana jamstva"
"gd_acc_965000","965000","equity","Other provisions for long-term accrued charges","False","Druge rezervacije iz naslova dolgoročno vnaprej vračunanih stroškov"
"gd_acc_966000","966000","income","State aid received","False","Prejete državne podpore"
"gd_acc_967000","967000","income","Donations received","False","Prejete donacije"
"gd_acc_968000","968000","income","Other non-current accrued liabilities","False","Druge dolgoročne pasivne časovne razmejitve"
"gd_acc_970000","970000","liability_non_current","Long-term loans obtained from group organisations","False","Dolgoročna posojila, dobljena pri organizacije v skupini"
"gd_acc_971000","971000","liability_non_current","Long-term loans obtained from associates","False","Dolgoročna posojila, dobljena pri pridruženih organizacije"
"gd_acc_972000","972000","liability_non_current","Long-term loans obtained from banks and organisations in the country - without analytics","False","Dolgoročna posojila, dobljena pri bankah in organizacijah v državi - brez analitike"
"gd_acc_972100","972100","liability_non_current","Long-term loans obtained from banks in the country","False","Dolgoročna posojila, dobljena pri bankah v državi"
"gd_acc_972200","972200","liability_non_current","Long-term loans obtained from state-owned companies","False","Dolgoročna posojila, dobljena pri družbah v državi"
"gd_acc_973000","973000","liability_non_current","Long-term loans obtained from banks and organisations abroad - without analytics","False","Dolgoročna posojila, dobljena pri bankah in organizacije v tujini - brez analitike"
"gd_acc_973100","973100","liability_non_current","Long-term loans from banks abroad","False","Dolgoročna posojila, dobljena pri bankah v tujini"
"gd_acc_973200","973200","liability_non_current","Long-term loans from companies abroad","False","Dolgoročna posojila, dobljena pri družbah v tujini"
"gd_acc_974000","974000","liability_non_current","Long-term financial liabilities related to bonds and notes","False","Dolgoročne finančne obveznosti v zvezi z obveznicami in menicami"
"gd_acc_975000","975000","liability_non_current","Long-term lease debts","False","Dolgoročni dolgovi iz najema (leasing)"
"gd_acc_975100","975100","liability_non_current","Interest from rent (leasing)","False","Obresti iz najema (leasing)"
"gd_acc_976000","976000","liability_non_current","Long-term financial liabilities to natural persons","False","Dolgoročne finančne obveznosti do fizičnih oseb"
"gd_acc_979000","979000","liability_non_current","Other long-term financial liabilities","False","Druge dolgoročne finančne obveznosti"
"gd_acc_980000","980000","liability_non_current","Long-term loans obtained under credit agreements from group organisations","False","Dolgoročni krediti, dobljeni na podlagi kreditnih pogodb od organizacij v skupini"
"gd_acc_981000","981000","liability_non_current","Long-term loans obtained under credit agreements from associates and jointly controlled entities","False","Dolgoročni krediti, dobljeni na podlagi kreditnih pogodb od pridruženih organizacij in skupaj obvladovanih organizacij"
"gd_acc_982000","982000","liability_non_current","Long-term credits obtained from other domestic suppliers","False","Dolgoročni krediti, dobljeni od drugih domačih dobaviteljev"
"gd_acc_983000","983000","liability_non_current","Long-term credits obtained from other foreign suppliers","False","Dolgoročni krediti, dobljeni od drugih tujih dobaviteljev"
"gd_acc_985000","985000","liability_non_current","Long-term promissory note liabilities","False","Dolgoročne menične obveznosti"
"gd_acc_986000","986000","liability_non_current","Long-term advances and securities received - without analytics","False","Dolgoročni dobljeni predujmi in varščine - brez analitike"
"gd_acc_986100","986100","liability_non_current","Long-term advances received","False","Dolgoročni prejeti predujmi"
"gd_acc_986200","986200","liability_non_current","Long-term securities received","False","Dolgoročne prejete varščine"
"gd_acc_988000","988000","liability_non_current","Deferred tax liabilities","False","Odložene obveznosti za davek"
"gd_acc_989000","989000","liability_non_current","Other non-current payables","False","Druge dolgoročne poslovne obveznosti"
"gd_acc_990000","990000","off_balance","Leased, borrowed and rented (foreign) assets","False","Najeta, izposojena in zakupljena (tuja) sredstva"
"gd_acc_991000","991000","off_balance","Bills of exchange and other securities received as security for payments","False","Menice in drugi vrednostni papirji, prejeti za zavarovanje plačil"
"gd_acc_992000","992000","off_balance","Goods received for consignment and consignment sale","False","Blago, prejeto v komisijsko in konsignacijsko prodajo"
"gd_acc_993000","993000","off_balance","Valuation vouchers issued for inter-organisational accounting","False","Vrednotnice, izdane za obračunavanje znotraj organizacije"
"gd_acc_994000","994000","off_balance","Other active off-balance sheet accounts","False","Drugi aktivni zunajbilančni konti"
"gd_acc_995000","995000","off_balance","Owners of leased, borrowed and rented assets","False","Lastniki najetih, izposojenih in zakupljenih sredstev"
"gd_acc_996000","996000","off_balance","Debtors who have secured payments with bills of exchange and other securities","False","Dolžniki, ki so zavarovali plačila z menicami in drugimi vrednostnimi papirji"
"gd_acc_997000","997000","off_balance","Liabilities arising from goods received for consignment and consignment sale","False","Obveznosti iz blaga, prejetega v komisijsko in konsignacijsko prodajo"
"gd_acc_998000","998000","off_balance","Nominal value of notes issued for intra-organisation accounting","False","Nominalna vrednost vrednotnic, izdanih za obračunavanje znotraj organizacije"
"gd_acc_999000","999000","off_balance","Other off-balance sheet passive accounts","False","Drugi pasivni zunajbilančni konti"
"gd_acc_999001","999001","off_balance","Cash Difference Loss","False","Cash Difference Loss"
"gd_acc_999002","999002","off_balance","Cash Difference Gain","False","Cash Difference Gain"
"gd_acc_999003","999003","income_other","Cash Difference Gain","False","Cash Difference Gain"
"gd_acc_999004","999004","expense","Cash Difference Loss","False","Cash Difference Loss"
"gd_acc_999900","999900","off_balance","Not defined","False","Nedefiniran"
"gd_acc_999999","999999","off_balance","Undistributed Profits/Losses","False","Undistributed Profits/Losses"

```

## File: data\template\account.fiscal.position-si.csv

```csv
"id","name","sequence","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@sl"
"gd_fp_do","Domestic partner","20","1","1","base.si","","","","","","Domači partner"
"gd_fp_eu","Partner EU","30","1","1","","base.europe","gd_taxr_3","gd_taxe_out_0_eu_goods_1","gd_acc_120000","gd_acc_121000","Partner EU"
"","","","","","","","gd_taxr_2","gd_taxe_out_0_eu_goods_1","","",""
"","","","","","","","l10n_si_vat_5_sale","gd_taxe_out_0_eu_goods_1","","",""
"","","","","","","","gd_taxp_3","gd_taxe_in_22_eu_goods_1","","",""
"","","","","","","","gd_taxp_2","gd_taxe_in_95_eu_goods_1","","",""
"","","","","","","","gd_taxe_out_22_no_service_1","gd_taxe_out_0_eu_service_1","","",""
"","","","","","","","gd_taxe_out_95_no_service_1","gd_taxe_out_0_eu_service_1","","",""
"","","","","","","","gd_taxe_out_5_no_service_1","gd_taxe_out_0_eu_service_1","","",""
"","","","","","","","gd_taxe_in_22_no_service_1","l10n_si_vat_22_purchase_services_eu","","",""
"","","","","","","","gd_taxe_in_95_no_service_1","gd_taxe_in_95_no_service_1","","",""
"","","","","","","","l10n_si_vat_5_purchase","l10n_si_vat_5_purchase_goods_eu","","",""
"","","","","","","","","","gd_acc_220000","gd_acc_221000",""
"","","","","","","","","","gd_acc_760000","gd_acc_761000",""
"","","","","","","","","","gd_acc_762000","gd_acc_763000",""
"","","","","","","","","","gd_acc_230000","gd_acc_230100",""
"gd_fp_do1","EU partner without MS","30","1","","","base.europe","","","gd_acc_120000","gd_acc_121000","Partner EU brez DŠ"
"","","","","","","","","","gd_acc_760000","gd_acc_761000",""
"","","","","","","","","","gd_acc_762000","gd_acc_763000",""
"","","","","","","","","","gd_acc_220000","gd_acc_221000",""
"","","","","","","","","","gd_acc_230000","gd_acc_230100",""
"gd_fp_ne","3. World Partner","40","1","","","","gd_taxp_3","gd_taxe_in_22_3w_goods_1","gd_acc_120000","gd_acc_121000","Partner 3. svet"
"","","","","","","","gd_taxp_2","gd_taxe_in_95_3w_goods_1","","",""
"","","","","","","","gd_taxr_3","gd_taxe_out_0_3w_goods_1","","",""
"","","","","","","","gd_taxr_2","gd_taxe_out_0_3w_goods_1","","",""
"","","","","","","","gd_taxe_out_95_no_service_1","gd_taxe_out_0_3w_service_1","","",""
"","","","","","","","gd_taxe_out_22_no_service_1","gd_taxe_out_0_3w_service_1","","",""
"","","","","","","","gd_taxe_out_5_no_service_1","gd_taxe_out_0_3w_service_1","","",""
"","","","","","","","l10n_si_vat_5_sale","gd_taxe_out_0_3w_goods_1","","",""
"","","","","","","","gd_taxe_in_22_no_service_1","gd_taxe_in_22_3w_service_1","","",""
"","","","","","","","gd_taxe_in_95_no_service_1","gd_taxe_in_95_3w_service_1","","",""
"","","","","","","","","","gd_acc_220000","gd_acc_221000",""
"","","","","","","","","","gd_acc_760000","gd_acc_761000",""
"",""," ","","","","","","","gd_acc_762000","gd_acc_763000",""
"","","","","","","","","","gd_acc_230000","gd_acc_230100",""

```

## File: data\template\account.group-si.csv

```csv
"id","code_prefix_start","name","name@sl"
"gd_acc_group_0","0","Long-term assets","Dolgoročna sredstva"
"gd_acc_group_00","00","Intangible assets and long-term accrued costs and deferred revenue","Neopredmetena sredstva in dolgoročne pasivne časovne razmejitve"
"gd_acc_group_01","01","Investment property","Naložbene nepremičnine"
"gd_acc_group_02","02","Real estate","Nepremičnina"
"gd_acc_group_03","03","Impairment and impairment of real estate","Oslabitev in slabitev nepremičnin"
"gd_acc_group_04","04","Equipment and other tangible fixed assets","Oprema in druga opredmetena osnovna sredstva"
"gd_acc_group_05","05","Adjustment and impairment of equipment and other property, plant and equipment","Prilagoditev in oslabitev opreme in drugih opredmetenih osnovnih sredstev"
"gd_acc_group_06","06","Long-term financial investments, except loans","Dolgoročne finančne naložbe, razen posojil"
"gd_acc_group_07","07","Long-term loans and receivables given for unpaid called-up capital","Dolgoročna posojila in terjatve, dana za nevplačan vpoklicani kapital"
"gd_acc_group_08","08","Long-term operating receivables","Dolgoročne poslovne terjatve"
"gd_acc_group_09","09","Deferred tax assets","Odložene terjatve za davek"
"gd_acc_group_1","1","Short-term assets, except inventories, and short-term accrued and deferred income","Kratkoročna sredstva, razen zalog, ter kratkoročne pasivne časovne razmejitve"
"gd_acc_group_10","10","Cash on hand and immediately realizable securities","Gotovina v blagajni in takoj unovčljivi vrednostni papirji"
"gd_acc_group_11","11","Balances with banks and other financial institutions","Stanja pri bankah in drugih finančnih institucijah"
"gd_acc_group_12","12","Short-term trade receivables","Kratkoročne terjatve do kupcev"
"gd_acc_group_13","13","Short-term advances and securities given","Kratkoročni predujmi in dane varščine"
"gd_acc_group_14","14","Short-term operating receivables for a foreign account","Kratkoročne poslovne terjatve za tuj račun"
"gd_acc_group_15","15","Short-term receivables related to financial income","Kratkoročne terjatve iz finančnih prihodkov"
"gd_acc_group_16","16","Other short-term receivables","Druge kratkoročne terjatve"
"gd_acc_group_17","17","Short-term financial investments, except loans","Kratkoročne finančne naložbe, razen posojil"
"gd_acc_group_18","18","Short-term loans and short-term receivables for unpaid capital","Kratkoročna posojila in kratkoročne terjatve za nevplačan kapital"
"gd_acc_group_19","19","Short-term accrued costs and deferred revenue","Kratkoročne pasivne časovne razmejitve"
"gd_acc_group_2","2","Short-term liabilities (debt) and short-term accrued and deferred income","Kratkoročne obveznosti (dolg) in kratkoročne pasivne časovne razmejitve"
"gd_acc_group_21","21","Liabilities included in disposal groups","Obveznosti, vključene v skupine za odtujitev"
"gd_acc_group_22","22","Short-term liabilities (debts) to suppliers","Kratkoročne obveznosti (dolgovi) do dobaviteljev"
"gd_acc_group_23","23","Short-term advances and securities received","Prejeti kratkoročni predujmi in varščine"
"gd_acc_group_24","24","Short-term operating liabilities for a foreign account","Kratkoročne poslovne obveznosti za tuj račun"
"gd_acc_group_25","25","Short-term wage liabilities","Kratkoročne obveznosti iz plač"
"gd_acc_group_26","26","Liabilities to state and other institutions","Obveznosti do države in drugih institucij"
"gd_acc_group_27","27","Short-term financial liabilities","Kratkoročne finančne obveznosti"
"gd_acc_group_28","28","Other current liabilities","Druge kratkoročne obveznosti"
"gd_acc_group_29","29","Shorthand passive time relations","Stenografske pasivne časovne relacije"
"gd_acc_group_3","3","Stocks of raw materials","Zaloge surovin"
"gd_acc_group_30","30","Accounting for the purchase of raw materials and supplies (including small inventory and packaging)","Obračun nabave surovin in materiala (vključno z drobnim inventarjem in embalažo)"
"gd_acc_group_31","31","Inventories of raw materials and supplies","Zaloge surovin in materiala"
"gd_acc_group_32","32","Stocks of small inventory and packaging","Zaloge drobnega inventarja in embalaže"
"gd_acc_group_4","4","Costs","Stroški"
"gd_acc_group_40","40","Material costs","Materialni stroški"
"gd_acc_group_41","41","Cost of service","Stroški storitve"
"gd_acc_group_43","43","Depreciation","Amortizacija"
"gd_acc_group_44","44","Reservations","Rezervacije"
"gd_acc_group_45","45","Interest costs","Stroški obresti"
"gd_acc_group_47","47","Labor costs","Stroški dela"
"gd_acc_group_48","48","Other costs","Drugi stroški"
"gd_acc_group_49","49","Cost transfer","Prenos stroškov"
"gd_acc_group_6","6","Inventories of products, services, goods and non-current assets (disposal groups) for sale","Zaloge proizvodov, storitev, blaga in nekratkoročnih sredstev (skupine za odtujitev) za prodajo"
"gd_acc_group_60","60","Work in progress and services","Nedokončana dela in storitve"
"gd_acc_group_61","61","Stocks of crops (harvested) from biological assets","Zaloge pridelkov (pospravljenih) iz bioloških sredstev"
"gd_acc_group_63","63","Products","Izdelki"
"gd_acc_group_65","65","Accounting for the purchase of goods","Računovodstvo nabave blaga"
"gd_acc_group_66","66","Stocks of goods","Zaloge blaga"
"gd_acc_group_67","67","Non-current assets (disposal groups) for sale","Nekratkoročna sredstva (skupine za odtujitev) za prodajo"
"gd_acc_group_7","7","Expenses and revenues","Odhodki in prihodki"
"gd_acc_group_70","70","Operating expenses (I. version of the income statement)","Poslovni odhodki (I. različica izkaza poslovnega izida)"
"gd_acc_group_71","71","Operating expenses (II. Version of the income statement)","Poslovni odhodki (II. verzija izkaza poslovnega izida)"
"gd_acc_group_72","72","Revaluation operating expenses","Prevrednotovalni poslovni odhodki"
"gd_acc_group_74","74","Financial expenses","Finančni odhodki"
"gd_acc_group_75","75","Other expenses","Drugi stroški"
"gd_acc_group_76","76","Business income","Poslovni prihodki"
"gd_acc_group_77","77","Financial income","Finančni prihodki"
"gd_acc_group_78","78","Other incomes","Drugi dohodki"
"gd_acc_group_79","79","Capitalize own products and own services","Kapitalizirajte lastne izdelke in lastne storitve"
"gd_acc_group_8","8","Profit or loss","Dobiček ali izguba"
"gd_acc_group_80","80","Profit or loss before tax","Dobiček ali izguba pred obdavčitvijo"
"gd_acc_group_81","81","Distribution of profit and / or total surplus revenue","Delitev dobička in/ali celotnega presežka prihodkov"
"gd_acc_group_82","82","Allocation of net profit for the financial year or net surplus of revenues","Razporeditev čistega dobička poslovnega leta oziroma čistega presežka prihodkov"
"gd_acc_group_89","89","Net loss or net surplus of expenses and transfer of net loss or net surplus of expenses","Čista izguba oziroma čisti presežek odhodkov in prenos čiste izgube oziroma čistega presežka odhodkov"
"gd_acc_group_9","9","Capital, long-term liabilities (debt) and long-term provisions","Kapital, dolgoročne obveznosti (dolg) in dolgoročne rezervacije"
"gd_acc_group_90","90","Called-up and initial capital and founding deposits","Vpoklicani in ustanovni kapital ter ustanovitveni vlogi"
"gd_acc_group_91","91","Capital reserves and transfers of funds","Kapitalske rezerve in prenosi sredstev"
"gd_acc_group_92","92","Profit reserves or allocated net surplus of revenues","Rezerve iz dobička oziroma razporejeni čisti presežek prihodkov"
"gd_acc_group_93","93","Net profit or net loss or unallocated net surplus of revenues or net surplus of expenses","Čisti dobiček ali čista izguba ali nerazporejeni čisti presežek prihodkov ali čisti presežek odhodkov"
"gd_acc_group_94","94","Revaluation reserves","Revalorizacijske rezerve"
"gd_acc_group_95","95","Reserves arising from fair value measurement","Rezervacije iz naslova merjenja poštene vrednosti"
"gd_acc_group_96","96","Provisions and long-term accrued costs and deferred revenue","Rezervacije in dolgoročne pasivne časovne razmejitve"
"gd_acc_group_97","97","Long-term financial liabilities","Dolgoročne finančne obveznosti"
"gd_acc_group_98","98","Long-term operating liabilities","Dolgoročne poslovne obveznosti"
"gd_acc_group_99","99","Off-balance sheet accounts","Zunajbilančni računi"

```

## File: data\template\account.tax-si.csv

```csv
"id","sequence","invoice_label","description","name","amount","amount_type","type_tax_use","tax_group_id","active","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@sl"
"gd_taxr_1","30","0%","0% output VAT - deductible","0% D","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 0% - s pravico do odbitka"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxr_2","20","9,5%","9.5% output VAT - goods","9.5% G","9.50","percent","sale","tax_group_95","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 9,5% - blago"
"","","","","","","","","","","100.00","tax","invoice","+22||+ir_15a","gd_acc_260010",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","-22||-ir_15a","gd_acc_260010",""
"gd_taxr_3","10","22%","22% Output VAT - goods","22% G","22.00","percent","sale","tax_group_22","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 22% - blago"
"","","","","","","","","","","100.00","tax","invoice","+21||+ir_14","gd_acc_260000",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","-21||-ir_14","gd_acc_260000",""
"gd_taxp_1","60","0%","0% input VAT - does not go on the books","0% Not in Books","0.00","percent","purchase","tax_group_0","","100.00","base","invoice","","","Vstopni DDV 0% - ne gre v knjigo"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxp_2","50","9,5%","Input VAT 9.5% - goods","9.5% G","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 9,5% - blago"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160010",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19","gd_acc_160010",""
"gd_taxp_3","40","22%","22% input VAT - goods","22% G","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 22% - blago"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160000",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160000",""
"gd_taxp_nr_1","70","9,5%","9.5% input VAT - non-deductible - goods","9.5% ND G","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 9,5% - neodbitni - blago"
"","","","","","","","","","","100.00","tax","invoice","+pr_17b","",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-pr_17b","",""
"gd_taxp_nr_2","80","22%","22% input VAT - non-deductible - goods","22% ND G","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 22% - neodbitni - blago"
"","","","","","","","","","","100.00","tax","invoice","+pr_17a","",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-pr_17a","",""
"gd_taxp_st_1","90","9,5%","Input VAT 9.5% - self-assessment Article 76a","9.5% Art. 76a","9.50","percent","purchase","tax_group_rc_95","","100.00","base","invoice","+31a||+ir_25c||+pr_9","","Vstopni DDV 9,5% - samoobdavčitev 76.a člen"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160470",""
"","","","","","","","","","","-100.00","tax","invoice","-25a||-ir_21a","gd_acc_260470",""
"","","","","","","","","","","100.00","base","refund","-31a||-ir_25c||-pr_9","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19","gd_acc_160470",""
"","","","","","","","","","","-100.00","tax","refund","+25a||+ir_21a","gd_acc_260470",""
"gd_taxp_st_2","100","22%","22% input VAT - self-assessment Article 76a","22% Art. 76a","22.00","percent","purchase","tax_group_rc_22","","100.00","base","invoice","+31a||+ir_25c||+pr_9","","Vstopni DDV 22% - samoobdavčitev 76.a člen"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160460",""
"","","","","","","","","","","-100.00","tax","invoice","-25||-ir_20","gd_acc_260460",""
"","","","","","","","","","","100.00","base","refund","-31a||-ir_25c||-pr_9","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160460",""
"","","","","","","","","","","-100.00","tax","refund","+25||+ir_20","gd_acc_260460",""
"gd_taxe_out_0_no_no_1","110","0%","0% output VAT - other exemptions","0% O Exempt","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 0% - druge oprostitve"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_0_no_no_2","110","0%","0% output VAT - does not go on the books","0% Not in books","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 0% - ne gre v knjigo"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_0_eu_goods_1","130","0%","0% EU output VAT  - Goods","0% EU G","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+12||+ir_10a||rp_a3","","Izstopni DDV 0% EU - blago"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-12||-ir_10a||rp_a3","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_0_eu_goods_2","140","0%","0% EU output VAT - customs procedure Articles 42 and 43 - goods","0% EU G Art. 42-43","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+12||+ir_10b||rp_a4","","Izstopni DDV 0% EU - carinski postopek 42. in 43. člen - blago"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-12||-ir_10b||rp_a4","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_0_eu_goods_3","150","0%","0% EU output VAT - triangular supplies - goods","0% EU G Tri","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+12||+ir_11||rp_a5","","Izstopni DDV 0% EU - tristranske dobave - blago"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-12||-ir_11||rp_a5","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_0_3w_goods_1","170","0%","0% Export VAT 3rd world - goods","0% EX G","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 0% 3. svet - blago"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_0_eu_service_1","180","0%","0% EU output VAT - services","0% EU S","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+12||+ir_10d||rp_a6","","Izstopni DDV 0% EU - storitev"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-12||-ir_10d||rp_a6","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_0_3w_service_1","200","0%","0% output VAT 3rd world - services","0% EX S","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+ir_23","","Izstopni DDV 0% 3. svet - storitev"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-ir_23","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_out_95_no_service_1","210","9,5%","9.5% output VAT - services","9.5% S","9.50","percent","sale","tax_group_95","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 9,5% - storitev"
"","","","","","","","","","","100.00","tax","invoice","+22||+ir_15a","gd_acc_260010",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","-22||-ir_15a","gd_acc_260010",""
"gd_taxe_out_22_no_service_1","220","22%","22% output VAT - services","22% S","22.00","percent","sale","tax_group_22","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 22% - storitev"
"","","","","","","","","","","100.00","tax","invoice","+21||+ir_14","gd_acc_260000",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","-21||-ir_14","gd_acc_260000",""
"gd_taxe_out_8_no_goods_1","230","8%","Export VAT 8%","8%","8.00","percent","sale","tax_group_8","","100.00","base","invoice","","","Izstopni DDV 8%"
"","","","","","","","","","","100.00","tax","invoice","","gd_acc_260500",""
"","","","","","","","","","","100.00","base","refund","","",""
"","","","","","","","","","","100.00","tax","refund","","gd_acc_260500",""
"gd_taxe_out_5_no_service_1","250","5%","Export VAT 5% - services","5% S","5.00","percent","sale","tax_group_5","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 5% - storitev"
"","","","","","","","","","","100.00","tax","invoice","+22a||+ir_15b","gd_acc_260020",""
"","","","","","","","","","","100.00","base","refund","-11||-ir_7","",""
"","","","","","","","","","","100.00","tax","refund","-22a||-ir_15b","gd_acc_260020",""
"gd_taxe_out_22_no_no_1","260","22%","22% output VAT - self-assessment Article 76","22% RC Art. 76","22.00","percent","sale","tax_group_rc_22","","100.00","base","invoice","+11a||+ir_8||+pd_a3","","Izstopni DDV 22% - samoobdavčitev 76. člen"
"","","","","","","","","","","","tax","invoice","","gd_acc_260460",""
"","","","","","","",,"","","100.00","base","refund","-11a||-ir_8||-pd_a3","",""
"","","","","","","","","","","","tax","refund","","gd_acc_260460",""
"gd_taxe_out_95_no_no_1","270","9,5%","Export VAT 9.5% - self-assessment Article 76","9.5% RC Art. 76","9.50","percent","sale",tax_group_rc_95,"","100.00","base","invoice","+11a||+ir_8||+pd_a3","","Izstopni DDV 9,5% - samoobdavčitev 76. člen"
"","","","","","","","","","","","tax","invoice","","gd_acc_260470",""
"","","","","","","","","","","100.00","base","refund","-11a||-ir_8||-pd_a3","",""
"","","","","","","","","","","","tax","refund","","gd_acc_260470",""
"gd_taxe_out_5_no_no_1","280","5%","Export VAT 5% - self-assessment Article 76","5% RC Art. 76","5.00","percent","sale","tax_group_rc_5","","100.00","base","invoice","+11a||+ir_8||+pd_a3","","Izstopni DDV 5% - samoobdavčitev 76. člen"
"","","","","","","","","","","","tax","invoice","","gd_acc_260020",""
"","","","","","","","","","","100.00","base","refund","-11a||-ir_8||-pd_a3","",""
"","","","","","","","","","","","tax","refund","","gd_acc_260020",""
"gd_taxe_in_95_no_service_1","290","9,5%","9.5% input VAT - services","9.5% S","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 9,5% - storitev"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160010",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19","gd_acc_160010",""
"gd_taxe_in_22_no_service_1","300","22%","22% input VAT - services","22% S","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 22% - storitev"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160000",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160000",""
"gd_taxe_in_95_no_service_2","310","9,5%","9.5% input VAT - non-deductible - services","9.5% ND S","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 9,5% - neodbitni - storitev"
"","","","","","","","","","","100.00","tax","invoice","+pr_17b","",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-pr_17b","",""
"gd_taxe_in_22_no_service_2","320","22%","22% input VAT - non-deductible - services","22% ND S","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 22% - neodbitni - storitev"
"","","","","","","","","","","100.00","tax","invoice","+pr_17a","",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-pr_17a","",""
"gd_taxe_in_8_no_no_1","330","8%","Input VAT 8% - non-deductible","8% ND","8.00","percent","purchase","tax_group_8","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 8% - neodbitni"
"","","","","","","","","","","100.00","tax","invoice","+pr_17c","",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-pr_17c","",""
"gd_taxe_in_5_no_service_1","350","5%","Input VAT 5% - non-deductible - services","5% ND S","5.00","percent","purchase","tax_group_5","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 5% - neodbitni - storitev"
"","","","","","","","","","","100.00","tax","invoice","+pr_17c","",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-pr_17c","",""
"gd_taxe_in_5_no_service_2","370","5%","Input VAT 5% - services","5% S","5.00","percent","purchase","tax_group_5","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 5% - storitev"
"","","","","","","","","","","100.00","tax","invoice","+42a||+pr_19a","gd_acc_160020",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-42a||-pr_19a","gd_acc_160020",""
"gd_taxe_in_8_no_no_2","380","8%","Input VAT 8%","8%","8.00","percent","purchase","tax_group_8","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 8%"
"","","","","","","","","","","100.00","tax","invoice","+43||+pr_20","gd_acc_160500",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-43||-pr_20","gd_acc_160500",""
"gd_taxe_in_5_no_no_1","390","5%","Input VAT 5% - self-assessment Article 76a","5% Art. 76a","5.00","percent","purchase","tax_group_rc_5","","100.00","base","invoice","+31a||+ir_25c||+pr_9","","Vstopni DDV 5% - samoobdavčitev 76.a člen"
"","","","","","","","","","","100.00","tax","invoice","+42a||+pr_19a","gd_acc_160020",""
"","","","","","","","","","","-100.00","tax","invoice","-25b||-ir_21b","gd_acc_260020",""
"","","","","","","","","","","100.00","base","refund","-31a||-ir_25c||-pr_9","",""
"","","","","","","","","","","100.00","tax","refund","-42a||-pr_19a","gd_acc_160020",""
"","","","","","","","","","","-100.00","tax","refund","+25b||+ir_21b","gd_acc_260020",""
"gd_taxe_in_0_no_no_1","400","0%","0% input VAT - exempt","0% Exempt","0.00","percent","purchase","tax_group_0","","100.00","base","invoice","+33||+pr_14","","Vstopni DDV 0% - oproščeno"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-33||-pr_14","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"gd_taxe_in_22_eu_service","410","22%","22% EU input VAT - self-assessment - non-deductible - services","22% EU ND S","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+32a||+ir_25b||+pr_11","","Vstopni DDV 22% EU - samoobdavčitev - neodbitni - storitve"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","-100.00","tax","invoice","-23a||-ir_17","gd_acc_260430",""
"","","","","","","","","","","100.00","base","refund","-32a||-ir_25b||+pr_11","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"","","","","","","","","","","-100.00","tax","refund","+23a||+ir_17","gd_acc_260430",""
"gd_taxe_in_22_eu_goods_1","420","22%","22% EU input VAT - goods","22% EU G","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+32||+ir_25a||+pr_10","","Vstopni DDV 22% EU - blago"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160400",""
"","","","","","","","","","","-100.00","tax","invoice","-23||-ir_16","gd_acc_260400",""
"","","","","","","","","","","100.00","base","refund","-32||-ir_25a||+pr_10","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160400",""
"","","","","","","","","","","-100.00","tax","refund","+23||+ir_16","gd_acc_260400",""
"gd_taxe_in_22_3w_service_1","430","22%","Input VAT 22% 3rd world - services","22% EX S","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+ir_25d||+pr_8b","","Vstopni DDV 22% 3. svet - storitve"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160301",""
"","","","","","","","","","","-100.00","tax","invoice","-25||-ir_20","gd_acc_260301",""
"","","","","","","","","","","100.00","base","refund","-31||-ir_25d||-pr_8b","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160301",""
"","","","","","","","","","","-100.00","tax","refund","+25||+ir_20","gd_acc_260301",""
"gd_taxe_in_22_3w_goods_1","440","22%","Input VAT 22% 3rd world - goods","22% EX G","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+ir_25d||+pr_8b","","Vstopni DDV 22% 3. svet - blago"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160300",""
"","","","","","","","","","","-100.00","tax","invoice","-26||-ir_22","gd_acc_260300",""
"","","","","","","","","","","100.00","base","refund","-31||-ir_25d||-pr_8b","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160300",""
"","","","","","","","","","","-100.00","tax","refund","+26||+ir_22","gd_acc_260300",""
"gd_taxe_in_95_eu_goods_1","460","9,5%","9.5% EU input VAT - goods","9.5% EU G","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+32||+ir_25a||+pr_10","","Vstopni DDV 9,5% EU - blago"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160410",""
"","","","","","","","","","","-100.00","tax","invoice","-24||-ir_18a","gd_acc_260410",""
"","","","","","","","","","","100.00","base","refund","-32||-ir_25a||-pr_10","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19a","gd_acc_160410",""
"","","","","","","","","","","-100.00","tax","refund","+24||+ir_18a","gd_acc_260410",""
"gd_taxe_in_95_3w_goods_1","470","9,5%","Entry VAT 9.5% 3rd world - goods","9.5% EX G","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+ir_25d||+pr_8b","","Vstopni DDV 9,5% 3. svet - blago"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160310",""
"","","","","","","","","","","-100.00","tax","invoice","-26||-ir_22a","gd_acc_260310",""
"","","","","","","","","","","100.00","base","refund","-31||-ir_25d||-pr_8b","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19","gd_acc_160310",""
"","","","","","","","","","","-100.00","tax","refund","+26||+ir_22a","gd_acc_260310",""
"gd_taxe_in_95_3w_service_1","480","9,5%","Input VAT 9.5% 3rd World - services","9.5% EX S","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+ir_25d||+pr_8b","","Vstopni DDV 9,5% 3. svet - storitve"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160311",""
"","","","","","","","","","","-100.00","tax","invoice","-25a||-ir_21a","gd_acc_260311",""
"","","","","","","","","","","100.00","base","refund","-31||-ir_25d||-pr_8b","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19","gd_acc_160311",""
"","","","","","","","","","","-100.00","tax","refund","+25a||+ir_21a","gd_acc_260311",""
"gd_taxe_in_5_3w_goods_1","490","5%","Entry VAT 5% 3rd world - goods","5% EX G","5.00","percent","purchase","tax_group_5","","100.00","base","invoice","+31||+ir_25d||+pr_8b","","Vstopni DDV 5% 3. svet - blago"
"","","","","","","","","","","100.00","tax","invoice","+42a||+pr_19a","gd_acc_160320",""
"","","","","","","","","","","-100.00","tax","invoice","-26||-ir_22b","gd_acc_260320",""
"","","","","","","","","","","100.00","base","refund","-31||-ir_25d||-pr_8b","",""
"","","","","","","","","","","100.00","tax","refund","-42a||-pr_19a","gd_acc_160320",""
"","","","","","","","","","","-100.00","tax","refund","+26||+ir_22b","gd_acc_260320",""
"l10n_si_vat_5_sale","510","5%","Export VAT 5% - goods","5% G","5.00","percent","sale","tax_group_5","","100.00","base","invoice","+11||+ir_7","","Izstopni DDV 5% - blago"
"","","","","","","","","","","100.00","tax","invoice","+22a||+ir_15b","gd_acc_260020",""
"","","","","","","","","","","100.00","base","refund","-11||+ir_7","",""
"","","","","","","","","","","100.00","tax","refund","-22a||-ir_15b","gd_acc_260020",""
"l10n_si_vat_22_sale_distance","550","0%","0% EU output VAT - distance selling - goods","0% EU D S G","0.00","percent","sale","tax_group_22","","100.00","base","invoice","+13||+ir_12","","Izstopni DDV 0% EU - prodaja na daljavo - blago"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-13||-ir_12","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"l10n_si_vat_22_sale_installation_eu","580","0%","0% EU output VAT - assembly - service","0% EU A S","0.00","percent","sale","tax_group_22","","100.00","base","invoice","+14||+ir_13","","Izstopni DDV 0% EU - montaža - storitev"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-14||-ir_13","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"l10n_si_vat_0_sale_no_deduction","610","0%","0% output VAT - no right to deduct","0% ND","0.00","percent","sale","tax_group_0","","100.00","base","invoice","+15||+ir_9","","Izstopni DDV 0% - brez pravice do odbitka"
"","","","","","","","","","","100.00","tax","invoice","","",""
"","","","","","","","","","","100.00","base","refund","-15||-ir_9","",""
"","","","","","","","","","","100.00","tax","refund","","",""
"l10n_si_vat_5_purchase","700","5%","Input VAT 5% - goods","5% G","5.00","percent","purchase","tax_group_5","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 5% - blago"
"","","","","","","","","","","100.00","tax","invoice","+42a||+pr_19a","gd_acc_160020",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-42a||-pr_19a","gd_acc_160020",""
"l10n_si_vat_5_purchase_non_deductible","710","5%","Input VAT 5% - non-deductible - goods","5% ND G","5.00","percent","purchase","tax_group_5","","100.00","base","invoice","+31||+pr_8a","","Vstopni DDV 5% - neodbitni - blago"
"","","","","","","","","","","100.00","tax","invoice","+pr_17c","",""
"","","","","","","","","","","100.00","base","refund","-31||-pr_8a","",""
"","","","","","","","","","","100.00","tax","refund","-pr_17c","",""
"l10n_si_vat_5_purchase_goods_eu","720","5%","5% EU input VAT - goods","5% EU G","5.00","percent","purchase","tax_group_5","","100.00","base","invoice","+32||+ir_25a||+pr_10","","Vstopni DDV 5% EU - blago"
"","","","","","","","","","","100.00","tax","invoice","+42a||+pr_19","gd_acc_160420",""
"","","","","","","","","","","-100.00","tax","invoice","-24b||-ir_18b","gd_acc_260420",""
"","","","","","","","","","","100.00","base","refund","-32||-ir_25a||-pr_10","",""
"","","","","","","","","","","100.00","tax","refund","-42a||-pr_19a","gd_acc_160420",""
"","","","","","","","","","","-100.00","tax","refund","+24b||+ir_18b","gd_acc_260420",""
"l10n_si_vat_22_purchase_services_eu","730","22%","22% EU input VAT - services","22% EU S","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+32a||+ir_25b||+pr_11","","Vstopni DDV 22% EU - storitve"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160430",""
"","","","","","","","","","","-100.00","tax","invoice","-23a||-ir_17","gd_acc_260430",""
"","","","","","","","","","","100.00","base","refund","-32a||-ir_25b||-pr_11","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160430",""
"","","","","","","","","","","-100.00","tax","refund","+23a||+ir_17","gd_acc_260430",""
"l10n_si_vat_9_purchase_services_eu","740","9,5%","9.5% EU input VAT - services","9.5% EU S","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+32a||+ir_25b||+pr_11","","Vstopni DDV 9,5% EU - storitve"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19a","gd_acc_160440",""
"","","","","","","","","","","-100.00","tax","invoice","-24a||-ir_19a","gd_acc_260440",""
"","","","","","","","","","","100.00","base","refund","-32a||-ir_25b||-pr_11","",""
"","","","","","","","","","","100.00","tax","refund","-42||+pr_19a","gd_acc_160440",""
"","","","","","","","","","","-100.00","tax","refund","+24a||-ir_19a","gd_acc_260440",""
"l10n_si_vat_22_purchase_estate","760","22%","22% input VAT - real estate","22% RE","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+34||+pr_8a||+pr_12","","Vstopni DDV 22% - nepremičnine"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160000",""
"","","","","","","","","","","100.00","base","refund","-31||-34||-pr_8a||-pr_12","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160000",""
"l10n_si_vat_9_purchase_estate","770","9,5%","9.5% input VAT - real estate","9.5% RE","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+34||+pr_8a||+pr_12","","Vstopni DDV 9,5% - nepremičnine"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160000",""
"","","","","","","","","","","100.00","base","refund","-31||-34||-pr_8a||-pr_12","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19","gd_acc_160000",""
"l10n_si_vat_22_purchase_fixed","820","22%","22% input VAT - fixed assets","22% FA","22.00","percent","purchase","tax_group_22","","100.00","base","invoice","+31||+35||+pr_8a||+pr_13","","Vstopni DDV 22% - osnovna sredstva"
"","","","","","","","","","","100.00","tax","invoice","+41||+pr_18","gd_acc_160000",""
"","","","","","","","","","","100.00","base","refund","-31||-35||-pr_8a||-pr_13","",""
"","","","","","","","","","","100.00","tax","refund","-41||-pr_18","gd_acc_160000",""
"l10n_si_vat_9_purchase_fixed","830","9,5%","9.5% input VAT - fixed assets","9.5% FA","9.50","percent","purchase","tax_group_95","","100.00","base","invoice","+31||+35||+pr_8a||+pr_13","","Vstopni DDV 9,5% - osnovna sredstva"
"","","","","","","","","","","100.00","tax","invoice","+42||+pr_19","gd_acc_160000",""
"","","","","","","","","","","100.00","base","refund","-31||-35||-pr_8a||-pr_13","",""
"","","","","","","","","","","100.00","tax","refund","-42||-pr_19","gd_acc_160000",""
"l10n_si_vat_5_purchase_fixed","840","5%","Input VAT 5% - fixed assets","5% FA","5.00","percent","purchase","tax_group_5","","100.00","base","invoice","+31||+35||+pr_8a||+pr_13","","Vstopni DDV 5% - osnovna sredstva"
"","","","","","","","","","","100.00","tax","invoice","+42a||+pr_19a","gd_acc_160000",""
"","","","","","","","","","","100.00","base","refund","-31||-35||-pr_8a||-pr_13","",""
"","","","","","","","","","","100.00","tax","refund","-42a||-pr_19a","gd_acc_160000",""

```

## File: data\template\account.tax.group-si.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@sl"
"tax_group_22","22% VAT","base.si","gd_acc_260800","gd_acc_160800","22% DDV"
"tax_group_95","9,5% VAT","base.si","gd_acc_260800","gd_acc_160800","9,5% DDV"
"tax_group_8","8% VAT","base.si","gd_acc_260800","gd_acc_160800","8% DDV"
"tax_group_5","5% VAT","base.si","gd_acc_260800","gd_acc_160800","5% DDV"
"tax_group_0","0% VAT","base.si","gd_acc_260800","gd_acc_160800","0% DDV"
"tax_group_rc_22","22% RC VAT","base.si","gd_acc_260800","gd_acc_160800","22% SO DDV"
"tax_group_rc_95","9,5% RC VAT","base.si","gd_acc_260800","gd_acc_160800","9,5% SO DDV"
"tax_group_rc_5","5% RC VAT","base.si","gd_acc_260800","gd_acc_160800","5% SO DDV"

```

## File: models\template_si.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('si')
    def _get_si_template_data(self):
        return {
            'property_account_receivable_id': 'gd_acc_120000',
            'property_account_payable_id': 'gd_acc_220000',
            'property_account_expense_categ_id': 'gd_acc_702000',
            'property_account_income_categ_id': 'gd_acc_762000',
            'code_digits': '6',
            'use_storno_accounting': True,
        }

    @template('si', 'res.company')
    def _get_si_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.si',
                'bank_account_code_prefix': '110',
                'cash_account_code_prefix': '100',
                'transfer_account_code_prefix': '109',
                'account_default_pos_receivable_account_id': 'gd_acc_125000',
                'income_currency_exchange_account_id': 'gd_acc_777000',
                'expense_currency_exchange_account_id': 'gd_acc_484000',
                'account_sale_tax_id': 'gd_taxr_3',
                'account_purchase_tax_id': 'gd_taxp_3',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_si

```

