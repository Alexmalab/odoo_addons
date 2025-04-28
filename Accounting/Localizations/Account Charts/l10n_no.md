# Odoo Module: l10n_no

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

def _preserve_tag_on_taxes(env):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(env, 'l10n_no')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Norway - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['no'],
    'version': '2.1',
    'author': 'Rolv Råen',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """This is the module to manage the accounting chart for Norway in Odoo.

Updated for Odoo 9 by Bringsvor Consulting AS <www.bringsvor.com>
""",
    'depends': [
        'base_iban',
        'base_vat',
        'account',
    ],
    'data': [
        'data/account_tax_report_data.xml',
        'views/res_partner_views.xml',
        'views/res_company_views.xml',
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
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.no"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_sales_goods_services_homeland" model="account.report.line">
                <field name="name">Sales of goods and services in Norway</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_code_3" model="account.report.line">
                        <field name="name">3 Sales and withdrawals of goods and services (high rate 25%) - base</field>
                        <field name="code">BASE_3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_3_tax" model="account.report.line">
                        <field name="name">3 Sales and withdrawals of goods and services (high rate 25%) - tax</field>
                        <field name="code">TAX_3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_3_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_31" model="account.report.line">
                        <field name="name">31 Sales and withdrawals of goods and services (medium rate 15%) - base</field>
                        <field name="code">BASE_31</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_31_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_31_tax" model="account.report.line">
                        <field name="name">31 Sales and withdrawals of goods and services (medium rate 15%) - tax</field>
                        <field name="code">TAX_31</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_31_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_33" model="account.report.line">
                        <field name="name">33 Sales and withdrawals of goods and services (low rate 12%) - base</field>
                        <field name="code">BASE_33</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_33_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">33 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_33_tax" model="account.report.line">
                        <field name="name">33 Sales and withdrawals of goods and services (low rate 12%) - tax</field>
                        <field name="code">TAX_33</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_33_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">33 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_5" model="account.report.line">
                        <field name="name">5 Sales and purchases of goods and services exempt from VAT (0%)</field>
                        <field name="code">BASE_5</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_6" model="account.report.line">
                        <field name="name">6 Sales of goods and services exempt from the VAT Act (0%)</field>
                        <field name="code">BASE_6</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6 Base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_sales_goods_services_abroad" model="account.report.line">
                <field name="name">Sales of goods and services abroad</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_code_52" model="account.report.line">
                        <field name="name">52 Sales of goods and services abroad exempt from VAT (0%)</field>
                        <field name="code">BASE_52</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_52_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">52 Base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_purchases_goods_services_homeland" model="account.report.line">
                <field name="name">Purchases of goods and services in Norway</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_code_1" model="account.report.line">
                        <field name="name">1 Purchases of goods and services with right of deduction (high rate 25%)</field>
                        <field name="code">TAX_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_11" model="account.report.line">
                        <field name="name">11 Purchases of goods and services with right of deduction (medium rate 15%)</field>
                        <field name="code">TAX_11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_13" model="account.report.line">
                        <field name="name">13 Purchases of goods and services with right of deduction (low rate 12%)</field>
                        <field name="code">TAX_13</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_purchases_goods_abroad" model="account.report.line">
                <field name="name">Purchases of goods from abroad (imports)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_code_14" model="account.report.line">
                        <field name="name">14 Deduction on purchases of goods from abroad (VAT paid on importation, high rate 25%)</field>
                        <field name="code">TAX_14</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">14 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_15" model="account.report.line">
                        <field name="name">15 Deduction on purchases of goods from abroad (VAT paid on importation, medium rate 15%)</field>
                        <field name="code">TAX_15</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_81" model="account.report.line">
                        <field name="name">81 Purchase of goods from abroad with right of deduction (high rate 25%) - base</field>
                        <field name="code">BASE_81</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_81_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">81 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_81_tax" model="account.report.line">
                        <field name="name">81 Purchase of goods from abroad with right of deduction (high rate 25%) - tax</field>
                        <field name="code">TAX_81</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_81_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">81 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_82" model="account.report.line">
                        <field name="name">82 Purchase of goods from abroad without right of deduction (high rate 25%) - base</field>
                        <field name="code">BASE_82</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_82_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">82 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_82_tax" model="account.report.line">
                        <field name="name">82 Purchase of goods from abroad without right of deduction (high rate 25%) - tax</field>
                        <field name="code">TAX_82</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_82_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">82 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_83" model="account.report.line">
                        <field name="name">83 Purchase of goods from abroad with right of deduction (medium rate 15%) - base</field>
                        <field name="code">BASE_83</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_83_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">83 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_83_tax" model="account.report.line">
                        <field name="name">83 Purchase of goods from abroad with right of deduction (medium rate 15%) - tax</field>
                        <field name="code">TAX_83</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_83_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">83 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_84" model="account.report.line">
                        <field name="name">84 Purchases of goods from abroad that are not deductible (medium rate 15%) - base</field>
                        <field name="code">BASE_84</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_84_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">84 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_84_tax" model="account.report.line">
                        <field name="name">84 Purchases of goods from abroad that are not deductible (medium rate 15%) - tax</field>
                        <field name="code">TAX_84</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_84_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">84 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_85" model="account.report.line">
                        <field name="name">85 Purchases of goods from abroad on which VAT is not charged (zero rate 0%)</field>
                        <field name="code">BASE_85</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_85_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">85 Base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_purchases_services_abroad" model="account.report.line">
                <field name="name">Purchases of services from abroad (imports)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_code_86" model="account.report.line">
                        <field name="name">86 Purchase of services from abroad with right of deduction (high rate 25%) - base</field>
                        <field name="code">BASE_86</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_86_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">86 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_86_tax" model="account.report.line">
                        <field name="name">86 Purchase of services from abroad with right of deduction (high rate 25%) - tax</field>
                        <field name="code">TAX_86</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_86_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">86 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_87" model="account.report.line">
                        <field name="name">87 Purchase of services from abroad without deductibility (high rate 25%) - base</field>
                        <field name="code">BASE_87</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_87_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">87 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_87_tax" model="account.report.line">
                        <field name="name">87 Purchase of services from abroad without deductibility (high rate 25%) - tax</field>
                        <field name="code">TAX_87</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_87_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">87 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_88" model="account.report.line">
                        <field name="name">88 Purchase of services from abroad with right of deduction (low rate 12%) - base</field>
                        <field name="code">BASE_88</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_88_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">88 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_88_tax" model="account.report.line">
                        <field name="name">88 Purchase of services from abroad with right of deduction (low rate 12%) - tax</field>
                        <field name="code">TAX_88</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_88_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">88 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_89" model="account.report.line">
                        <field name="name">89 Purchase of services from abroad without deductibility (low rate 12%) - base</field>
                        <field name="code">BASE_89</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_89_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">89 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_89_tax" model="account.report.line">
                        <field name="name">89 Purchase of services from abroad without deductibility (low rate 12%) - tax</field>
                        <field name="code">TAX_89</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_89_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">89 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_fish_etc" model="account.report.line">
                <field name="name">Fish, etc.</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_code_12" model="account.report.line">
                        <field name="name">12 Purchase of fish and other marine wildlife resources (11.11%)</field>
                        <field name="code">TAX_12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_32" model="account.report.line">
                        <field name="name">32 Sale of fish and other marine wildlife resources (11.11%) - base</field>
                        <field name="code">BASE_32</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_32_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_32_tax" model="account.report.line">
                        <field name="name">32 Sale of fish and other marine wildlife resources (11.11%) - tax</field>
                        <field name="code">TAX_32</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_32_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_emission_and_gold" model="account.report.line">
                <field name="name">Carbon credits and gold</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_code_51" model="account.report.line">
                        <field name="name">51 Sale of carbon credits and gold to traders (0%)</field>
                        <field name="code">BASE_51</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_51_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">51 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_91" model="account.report.line">
                        <field name="name">91 Purchase of carbon credits and gold with deductibility (high rate 25%) - base</field>
                        <field name="code">BASE_91</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_91_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">91 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_91_tax" model="account.report.line">
                        <field name="name">91 Purchase of carbon credits and gold with deductibility (high rate 25%) - tax</field>
                        <field name="code">TAX_91</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_91_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">91 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_92" model="account.report.line">
                        <field name="name">92 Purchase of carbon credits and gold without deductibility (high rate 25%) - base</field>
                        <field name="code">BASE_92</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_92_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">92 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_92_tax" model="account.report.line">
                        <field name="name">92 Purchase of carbon credits and gold without deductibility (high rate 25%) - tax</field>
                        <field name="code">TAX_92</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_92_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">92 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_sum" model="account.report.line">
                <field name="name">Sum</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_to_be_paid" model="account.report.line">
                        <field name="name">Tax to pay</field>
                        <field name="code">SUM</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_to_be_paid_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TAX_3.balance+TAX_31.balance+TAX_32.balance+TAX_33.balance+TAX_81.balance+TAX_82.balance+TAX_83.balance+TAX_84.balance+TAX_86.balance+TAX_87.balance+TAX_88.balance+TAX_89.balance+TAX_91.balance+TAX_92.balance-TAX_1.balance-TAX_11.balance-TAX_12.balance-TAX_13.balance-TAX_14.balance-TAX_15.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-no.csv

```csv
"id","code","name","account_type","tag_ids","reconcile","name@nb_NO"
"no_a_cash","1000","Research and development","asset_current","","False","Forskning og utvikling"
"chart1010","1010","Research and development, self-developed","asset_current","","False","Forskning og utvikling, egenutviklet"
"chart1020","1020","Concessions","asset_current","","False","Konsesjoner"
"chart1030","1030","Patents","asset_current","","False","Patenter"
"chart1040","1040","Licenses","asset_current","","False","Lisenser"
"chart1050","1050","Trademarks","asset_current","","False","Varemerker"
"chart1060","1060","Others rights","asset_current","","False","Andre rettigheter"
"chart1070","1070","Deferred tax benefit","asset_current","","False","Utsatt skattefordel"
"chart1080","1080","Goodwill","asset_current","","False","Velvilje"
"chart1100","1100","Buildings","asset_current","","False","Bygninger"
"chart1120","1120","Building facilities","asset_current","","False","Bygningsmessige anlegg"
"chart1130","1130","Construction in progress","asset_current","","False","Anlegg under utførelse"
"chart1140","1140","Agricultural and forestry properties","asset_current","","False","Jord- og skogbrukseiendommer"
"chart1150","1150","Plots and others plots of land","asset_current","","False","Tomter og andre grunnarealer"
"chart1160","1160","Housing including land","asset_current","","False","Boliger inklusive tomter"
"chart1190","1190","Others fixed assets","asset_current","","False","Andre anleggsmidler"
"chart1200","1200","Machinery and equipment","asset_current","","False","Maskiner og anlegg"
"chart1210","1210","Machines and facilities under construction","asset_current","","False","Maskiner og anlegg under utførelse"
"chart1220","1220","Ships, rigs, planes","asset_current","","False","Skip, rigger, fly"
"chart1230","1230","Cars","asset_current","","False","Biler"
"chart1239","1239","Wagon trains, trucks and buses","asset_current","","False","Vogntog, lastebiler og busser"
"chart1240","1240","Others means of transport","asset_current","","False","Andre transportmidler"
"chart1249","1249","Others means of transport","asset_current","","False","Andre transportmidler"
"chart1250","1250","Inventory","asset_current","","False","Inventar"
"chart1260","1260","Fixed building equipment with other depreciation","asset_current","","False","Fast bygningsinventar med annen avskrivning"
"chart1270","1270","Tools etc.","asset_current","","False","Verktøy mv."
"chart1280","1280","Office machines","asset_current","","False","Kontormaskiner"
"chart1290","1290","Others operating assets","asset_current","","False","Andre driftsmidler"
"chart1300","1300","Investments in subsidiaries","asset_current","","False","Investeringer i datterselskaper"
"chart1310","1310","Investments other companies in the same group","asset_current","","False","Investeringer annet foretak i samme konsern"
"chart1320","1320","Loans to companies in the same group","asset_current","","False","Lån til foretak samme konsern"
"chart1330","1330","Investments in affiliated companies","asset_current","","False","Investeringer i tilknyttede selskap"
"chart1340","1340","Loans to affiliated companies","asset_current","","False","Lån til tilknyttede selskap"
"chart1350","1350","Investments in stocks and shares","asset_current","","False","Investeringer i aksjer og andeler"
"chart1360","1360","Bonds","asset_current","","False","Obligasjoner"
"chart1370","1370","Claims on owners, board members, etc.","asset_current","","False","Fordringer på eiere, styremedlemmer mv."
"chart1380","1380","Receivables from employees","asset_current","","False","Fordringer på ansatte"
"chart1390","1390","Others long-term receivables","asset_current","","False","Andre langsiktige fordringer"
"chart1399","1399","Others changes","asset_current","","False","Andre fordringer"
"chart1400","1400","Raw materials and purchased semi-finished products","asset_current","","False","Råvarer og innkjøpte halvfabrikata"
"chart1401","1401","Semi-finished products","asset_current","","False","Halvfabrikata"
"chart1420","1420","Goods under production","asset_current","","False","Varer under tilvirkning"
"chart1440","1440","Finished self-made goods","asset_current","","False","Ferdige egentilvirkede varer"
"chart1460","1460","Purchased goods for resale","asset_current","","False","Innkjøpte varer for videresalg"
"chart1480","1480","Advance payment to suppliers","asset_current","","False","Forskuddsbetaling til leverandører"
"chart1481","1481","Short-term receivables from suppliers","asset_current","","False","Kortsiktige fordringer hos leverandører"
"chart1500","1500","Accounts receivable","asset_receivable","","True","Kundefordringer"
"chart1501","1501","Accounts Receivable (PoS)","asset_receivable","","True","Kundefordringer (PoS)"
"chart1509","1509","Accounts receivable not entered in the ledger","asset_current","","False","Ikke reskontroførte kundefordringer"
"chart1513","1513","Uninvoiced accounts receivable","asset_current","","False","Ikke fakturerte kundefordringer"
"chart1514","1514","Accounts receivable - services","asset_current","","False","Kundfordringer - tjenester"
"chart1520","1520","Other short-term receivables","asset_current","","False",""
"chart1530","1530","Earned, not invoiced operating income","asset_current","","False","Opptjente, ikke fakturerte driftsinntekter"
"chart1550","1550","Accounts receivable from companies in the same group","asset_current","","False","Kundefordringer på selskap samme konsern"
"chart1560","1560","Others claims on companies within the same group","asset_current","","False","Andre fordringer på selskap innen samme konsern"
"chart1570","1570","Other short-term receivables","asset_current","","False",""
"chart1571","1571","Salary advance","asset_current","","False","Lønnsforskudd"
"chart1572","1572","Others short-term loans to employees","asset_current","","False","Andre kortsiktige lån til ansatte"
"chart1579","1579","Others short-term receivables","asset_current","","False","Andre kortsiktige fordringer"
"chart1580","1580","Provision for losses on receivables","asset_current","","False","Avsetning tap på fordringer"
"chart1600","1600","Output VAT","asset_current","","False","Utgående merverdiavgift"
"chart1601","1601","Outbound VAT high rate","asset_current","","False","Utgående merverdiavgift høy sats"
"chart1602","1602","Outgoing VAT buy earn. from abroad","asset_current","","False","Utgående merverdiavgift kjøp tjen. fra utlandet"
"chart1603","1603","Outgoing value added tax medium rate","asset_current","","False","Utgående merverdiavgift middels sats"
"chart1604","1604","Outbound VAT low rate","asset_current","","False","Utgående merverdiavgift lav sats"
"chart1605","1605","Basis outgoing VAT, high rate","asset_current","","False","Grunnlag utgående merverdiavgift, høy sats"
"chart1606","1606","Basis outgoing value added tax, medium rate","asset_current","","False","Grunnlag utgående merverdiavgift, middels sats"
"chart1607","1607","Basis outgoing VAT, low rate","asset_current","","False","Grunnlag utgående merverdiavgift, lav sats"
"chart1610","1610","Input VAT","asset_current","","False","Inngående merverdiavgift"
"chart1611","1611","Input VAT high rate","asset_current","","False","Inngående merverdiavgift høy sats"
"chart1612","1612","Incoming VAT buy earn. from abroad","asset_current","","False","Inngående merverdiavgift kjøp tjen. fra utlandet"
"chart1613","1613","Input VAT medium rate","asset_current","","False","Inngående merverdiavgift middels sats"
"chart1614","1614","Input VAT low rate","asset_current","","False","Inngående merverdiavgift lav sats"
"chart1640","1640","Settlement Account VAT","asset_current","","False","Oppgjørskonto merverdiavgift"
"chart1670","1670","Requirements for public subsidies","asset_current","","False","Krav på offentlige tilskudd"
"chart1700","1700","Prepaid rent","asset_current","","False","Forskuddsbetalt leie"
"chart1710","1710","Interest paid in advance","asset_current","","False","Forskuddsbetalt rente"
"chart1720","1720","Salary paid in advance","asset_current","","False","Forskuddsbetalt lønn"
"chart1740","1740","Prepaid expenses not accrued","asset_current","","False","Forskuddsbetalt, ikke påløpt lønn"
"chart1749","1749","Others prepaid costs","asset_current","","False","Andre forskuddbetalte kostnader"
"chart1750","1750","Accrued rental income","asset_current","","False","Påløpte leieinntekter"
"chart1760","1760","Accrued interest income","asset_current","","False","Påløpte renteinntekter"
"chart1780","1780","Requirements for payment of company capital","asset_current","","False","Krav på innbetaling av selskapskapital"
"chart1790","1790","Interim account","asset_current","","False","Interimskonto"
"chart1800","1800","Shares & shares in companies in the same cons.","asset_current","","False","Aksjer & andeler i foretak i samme kons."
"chart1810","1810","Market-based shares","asset_current","","False","Markesdbaserte aksjer"
"chart1820","1820","Others shares","asset_current","","False","Andre aksjer"
"chart1830","1830","Market-based bonds","asset_current","","False","Markedsbaserte obligasjoner"
"chart1840","1840","Others bonds","asset_current","","False","Andre obligasjoner"
"chart1850","1850","Market-based certificates","asset_current","","False","Markedsbaserte sertifikater"
"chart1860","1860","Others certificates","asset_current","","False","Andre sertifikater"
"chart1870","1870","Others market-based financial instruments","asset_current","account.account_tag_financing","False","Andre markedsbaserte finansielle instrumenter"
"chart1880","1880","Others financial instruments","asset_current","account.account_tag_financing","False","Andre finansielle instrumenter"
"chart1890","1890","Shares outside the group","asset_current","","False","Andeler utenfor konsern"
"chart1905","1905","Cash Euro","asset_current","","False","Kontanter Euro"
"chart1908","1908","Cash, other currency","asset_current","","False","Kontanter, annen valuta"
"chart1910","1910","Cash register","asset_current","","False","Kasse"
"chart1918","1918","Cash differentials","asset_current","","False","Kassedifferenser"
"chart1919","1919","Box in transfer","asset_current","","False","Kasse i transfer"
"chart1921","1921","Bank deposits 2","asset_current","","False","Bankinnskudd 2"
"chart1942","1942","Bank unallocated deposit","asset_current","","False","Bank ikke allokert innbetaling"
"chart1944","1944","Bank not identified payment","asset_current","","False","Bank ikke identifisert innbetaling"
"chart1950","1950","Bank deposit for tax deductions","asset_current","","False","Bankinnskudd for skattetrekk"
"chart1980","1980","Currency account Euro","asset_current","","False","Valutakonto Euro"
"chart2000","2000","Share capital","equity","","False","Aksjekapital"
"chart2010","2010","Own shares","equity","","False","Egne aksjer"
"chart2020","2020","Premium fund","equity","","False","Overkursfond"
"chart2030","2030","Other equity","equity","","False","Annen innskutt egenkapital"
"chart2036","2036","Foundation expenses","equity","","False","Stiftelsesutgifter"
"chart2040","2040","Fund for valuation differences","equity","","False","Fond for vurderingsforskjeller"
"chart2050","2050","Other equity","equity","","False","Annen innskutt egenkapital"
"chart2055","2055","Deposit cash","equity","","False","Innskudd kontanter"
"chart2060","2060","Private withdrawal","equity","","False","Privatuttak"
"chart2070","2070","Withholding tax","equity","","False","Forskuddsskatt"
"chart2080","2080","Uncovered loss","equity","","False","Udekket tap"
"chart2098","2090","Profit or loss from previous year","equity","","False","Gevinst eller tap fra foregående år"
"chart2100","2100","Pension obligations","liability_current","","False","Pensjonsforpliktelser"
"chart2120","2120","Deferred tax","liability_current","","False","Utsatt skatt"
"chart2140","2140","Set aside for warranty & service instructions","liability_current","","False","Avsetn. for garanti- & serviceforpl."
"chart2160","2160","unearned income","liability_current","","False","Uopptjent inntekt"
"chart2180","2180","Others provisions for commitments","liability_current","","False","Andre avsetninger for forpiktelser"
"chart2188","2188","Provision for VAT due","liability_current","","False","Avsetning for skyldig merverdiavgift"
"chart2200","2200","Convertible loans","liability_current","","False","Konvertible lån"
"chart2210","2210","Bond loan","liability_current","","False","Obligsjonslån"
"chart2220","2220","Debt to credit institutions","liability_current","","False","Gjeld til kredittinstitusjoner"
"chart2240","2240","Mortgage loans","liability_current","","False","Pantelån"
"chart2251","2251","Debt to employees","liability_current","","False","Gjeld til ansatte"
"chart2255","2255","Debt to owners","liability_current","","False","Gjeld til eiere"
"chart2260","2260","Debt to companies in the same group","liability_current","","False","Gjeld til selskap i samme konsern"
"chart2270","2270","Others foreign currency loans","liability_current","","False","Andre valutalån"
"chart2280","2280","Quiet stakeholder contributions and corresponding loan capital","liability_current","","False","Stille interessentinnskudd og ansv. lånekapital"
"chart2299","2299","Other long-term debt","liability_current","","False","Annen langsiktig gjeld"
"chart2300","2300","Convertible loans","liability_current","","False","Konvertible lån"
"chart2320","2320","Certificate loan","liability_current","","False","Sertifikatlån"
"chart2340","2340","Others foreign currency loans","liability_current","","False","Andre valutalån"
"chart2360","2360","Construction loan","liability_current","","False","Byggelån"
"chart2380","2380","Overdraft","liability_current","","True","Kassakreditt"
"chart2390","2390","Other debts to credit institutions","liability_current","","False","Annen gjeld til kredittinstitusjon"
"chart2400","2400","Accounts payable","liability_payable","","True","Leverandørgjeld"
"chart2409","2409","Accounts payable not entered in the ledger","liability_current","","False","Ikke reskontroført leverandørgjeld"
"chart2441","2441","Accounts payable - services","liability_payable","","True","Leverandørgjeld - tjenester"
"chart2442","2442","Uninvoiced goods received","liability_current","","False","Ikke fakturerte mottatte varer"
"chart2460","2460","Accounts payable to companies in the same group","liability_payable","","True","Leverandørgjeld til selskap i samme konsern"
"chart2490","2490","Accrued living debt, goods received","liability_current","","False","Påløpt lev.gjeld, mottatte varer"
"chart2491","2491","Bills of exchange","liability_current","","False","Vekselgjeld"
"chart2500","2500","Deferred tax payable, not offset","liability_current","","False","Avsatt betalbar skatt, ikke utlignet"
"chart2510","2510","Payable tax, offset","liability_current","","False","Betalbar skatt, utlignet"
"chart2530","2530","Tax refund according to the Tax Act §31 subsection 5","liability_current","","False","Refusjon skatt etter Skatteloven §31 5. ledd"
"chart2540","2540","Advance tax","liability_current","","False","Forhåndsskatt"
"chart2600","2600","Advance payments","liability_current","","False","Forskuddstrekk"
"chart2610","2610","Additional features","liability_current","","False","Påleggstrekk"
"chart2620","2620","Contribution deduction","liability_current","","False","Bidragstrekk"
"chart2630","2630","Social security benefits","liability_current","","False","Trygdetrekk"
"chart2640","2640","Insurance coverage","liability_current","","False","Forsikringstrekk"
"chart2650","2650","Union dues deducted","liability_current","","False","Trukket fagforeningskontigent"
"chart2690","2690","Second move","liability_current","","False","Andre trekk"
"chart2700","2700","Output VAT","liability_current","","False","Utgående merverdiavgift"
"chart2701","2701","Outbound VAT high rate","liability_current","","False","Utgående merverdiavgift høy sats"
"chart2702","2702","Outgoing VAT buy earn. from abroad","liability_current","","False","Utgående merverdiavgift kjøp tjen. fra utlandet"
"chart2703","2703","Outgoing value added tax medium rate","liability_current","","False","Utgående merverdiavgift middels sats"
"chart2704","2704","Outbound VAT low rate","liability_current","","False","Utgående merverdiavgift lav sats"
"chart2710","2710","Input VAT","liability_current","","False","Inngående merverdiavgift"
"chart2711","2711","Input VAT high rate","liability_current","","False","Inngående merverdiavgift høy sats"
"chart2712","2712","Incoming VAT buy earn. from abroad","liability_current","","False","Inngående merverdiavgift kjøp tjen. fra utlandet"
"chart2713","2713","Input VAT medium rate","liability_current","","False","Inngående merverdiavgift middels sats"
"chart2714","2714","Input VAT low rate","liability_current","","False","Inngående merverdiavgift lav sats"
"chart2727","2727","Output VAT, buy goods from abroad, high rate","liability_current","","False","Utgående mva, kjøp varer fra utlandet, høy sats"
"chart2728","2728","Outbound VAT, buy goods from abroad, medium rate","liability_current","","False","Utgående mva, kjøp varer fra utlandet, middels sats"
"chart2740","2740","Settlement Account VAT","liability_current","","False","Oppgjørskonto merverdiavgift"
"chart2741","2741","Input VAT, buy goods from abroad, high rate","liability_current","","False","Inngående mva, kjøp varer fra utlandet, høy sats"
"chart2742","2742","Input VAT, buy goods from abroad, medium rate","liability_current","","False","Inngående mva, kjøp varer fra utlandet, middels sats"
"chart2745","2745","Basis outgoing VAT purchase of services abroad","liability_current","","False","Grunnlag utgående mva kjøp av tjenester utland"
"chart2746","2746","Offsetting account basis for purchase of services abroad","liability_current","","False","Motkonto grunnlag kjøp av tjenester utland"
"chart2761","2761","Basic import goods, high rate","liability_current","","False","Grunnlag innførsel varer, høy sats"
"chart2762","2762","Counter account basis import goods, high rate","liability_current","","False","Motkonto grunnlag innførsel varer, høy sats"
"chart2763","2763","Basic import goods, medium rate","liability_current","","False","Grunnlag innførsel varer, middels sats"
"chart2764","2764","Counter account basis import goods, medium rate","liability_current","","False","Motkonto grunnlag innførsel varer, middels sats"
"chart2765","2765","Basic import goods, no VAT","liability_current","","False","Grunnlag innførsel varer, ingen mva"
"chart2766","2766","Counter account basis import goods, no VAT","liability_current","","False","Motkonto grunnlag innførsel varer, ingen mva"
"chart2770","2770","Obliged employer's tax","liability_current","","False","Skyldig arbeidsgiveravgift"
"chart2780","2780","Accrued employer's tax","liability_current","","False","Påløpt arbeidsgiveravgift"
"chart2781","2781","Employer's tax accrued on holidays","liability_current","","False","Arb.giv.avg. pål. feriep."
"chart2785","2785","Accrued employer contribution holiday pay","liability_current","","False","Påløpt arbeidsgiveravgift ferielønn"
"chart2790","2790","Others public charges","liability_current","","False","Andre offentlige avgifter"
"chart2800","2800","Allocated dividend","liability_current","","False","Avsatt utbytte"
"chart2900","2900","Advance from customers","liability_current","","False","Forskudd fra kunder"
"chart2902","2902","Advance customer gift cards","liability_current","","False","Forskudd kunder gavekort"
"chart2903","2903","Advance customer allowance","liability_current","","False","Forskudd kunder tilgodelapp"
"chart2910","2910","Debt to employees and owners","liability_current","","False","Gjeld til ansatte og eiere"
"chart2920","2920","Debt to companies in the same group","liability_current","","False","Gjeld til selskap i samme konsern"
"chart2930","2930","Payment","liability_current","","False","Lønn"
"chart2940","2940","Vacation money","liability_current","","False","Feriepenger"
"chart2950","2950","Accrued interest","liability_current","","False","Påløpte renter"
"chart2960","2960","Accrued costs and advance payment. deposit","liability_current","","False","Påløpte kostn. og forskuddsbet. inskudd"
"chart2970","2970","Unearned income, provision","liability_current","","False","Uopptjent inntekt , avsetning"
"chart2980","2980","Provisions and liabilities","liability_current","","False","Avsetninger og forpliktelser"
"chart2990","2990","Other short-term debt","liability_current","","False","Annen kortsiktig gjeld"
"chart3000","3000","Revenue from sales of merchandise tax pl. high rate","income","account.account_tag_operating","False","Salgsinntekt handelsvarer avgiftspl. høy sats"
"chart3010","3010","Sales revenue own. goods tax pl. high rate","income","account.account_tag_operating","False","Salgsinntekt egentilv. varer avgiftspl. høy sats"
"chart3020","3020","Sales revenue services taxable high rate","income","","False","Salgsinntekt tjenester avgiftspl. høy sats"
"chart3030","3030","Sales revenue of merchandise excl. tax. medium rate","income","","False","Salgsinntekt handelsvarer avgiftpl. middels sats"
"chart3035","3035","Raw materials and purchased semi-finished products","income","","False","Råvarer og innkjøpte halvfabrikata"
"chart3040","3040","Sales revenue own. goods tax plus medium rate","income","","False","Salgsinntekt egentilv. varer avgiftpl middels sats"
"chart3050","3050","Sales income services taxable low rate","income","","False","Salgsinntekter tjenester avgiftspl. lav sats"
"chart3060","3060","Withdrawal of goods subject to tax at a high rate","income","","False","Uttak av varer avgiftspliktig høy sats"
"chart3061","3061","Withdrawal of goods, subject to tax, high rate","income","","False","Uttak av varer, avgiftspliktig, høy sats"
"chart3062","3062","Withdrawal of goods, deposit required, medium rate","income","","False","Uttak av varer, avg.pliktig, middels sats"
"chart3063","3063","Withdrawal of goods subject to tax medium rate","income","","False","Uttak av varer avgiftspliktig middels sats"
"chart3070","3070","Withdrawal of services subject to tax at a high rate","income","","False","Uttak av tjenester avgiftspliktig høy sats"
"chart3071","3071","Withdrawal of services, chargeable, high rate","income","","False","Uttak av tjenester, avgiftspliktig, høy sats"
"chart3074","3074","Withdrawal of services subject to tax at a low rate","income","","False","Uttak av tjenester avgiftspliktig lav sats"
"chart3080","3080","Discounts and other sales income, taxable","income","","False","Rabatter og annen salgsinntektsred., avgiftspl."
"chart3090","3090","Refundable expenses at the buyer's expense, taxable","income","","False","Refunderbare utlegg for kjøpers regning, avgiftspl"
"chart3095","3095","Uninvoiced turnover","income","","False","Ikke fakturert omsetning"
"chart3096","3096","Invoiced but not delivered","income","","False","Fakturert men ikke levert"
"chart3100","3100","Revenue from the sale of merchandise tax-free","income","account.account_tag_operating","False","Salgsinntekt handelsvarer avgiftsfri"
"chart3105","3105","Sales revenue merchandise, export, tax-free","income","","False","Salgsinntekt handelsvarer, utførsel, avgiftsfri"
"chart3110","3110","Sales revenue self-made goods tax-free","income","account.account_tag_operating","False","Salgsinntekt egentilvirkede varer avgiftsfri"
"chart3120","3120","Sales revenue services tax-free","income","account.account_tag_operating","False","Salgsinntekt tjenester avgiftsfri"
"chart3125","3125","Sales revenue services, export, duty-free","income","","False","Salgsinntekt tjenester, utførsel, avgiftsfri"
"chart3160","3160","Withdrawal of goods free of charge","income","account.account_tag_operating","False","Uttak av varer avgiftsfritt"
"chart3170","3170","Withdrawal of services, free of charge","income","","False","Uttak av tjenester, avgiftsfritt"
"chart3180","3180","Discounts and others sales revenue reduction tax-free","income","","False","Rabatter og andre salgsinntektsreduksjon avgiftsfri"
"chart3190","3190","Refundable outlays at the buyer's expense, free of charge","income","","False","Refunderbare utlegg for kjøpers regning, avg.fri"
"chart3200","3200","Sales revenue merchandise goods outside the ag. area","income","","False","Salgsinntekt handelsvarervarer utenfor avg.omr"
"chart3210","3210","Sales income of self-made goods outside the ag. area","income","","False","Salgsinntekt egentilvirkede varer utenfor avg.omr"
"chart3220","3220","Sales revenue from services outside the avg. area","income","","False","Salgsinntekt tjenester utenfor avg.omr"
"chart3260","3260","Withdrawal of goods outside the tax area","income","","False","Uttak av varer utenfor avgiftsområdet"
"chart3280","3280","Discounts and other sales revenue reduction","income","","False","Rabatter og annen salgsinntektsreduksjon"
"chart3281","3281","Cash discount, given to the customer","income","","False","Kasserabatt, git kunde"
"chart3300","3300","Special publicly. dept. manufactured/sold goods","income","","False","Spes. offent. avg. tilvirk./solgte varer"
"chart3301","3301","Special off. dept. add./sold goods freely","income","","False","Spesiell off. avg. tilv./solgte varer fritt"
"chart3302","3302","Environmental tax for added/sold goods subject to tax","income","","False","Miljø avgift for tilv./solgte varer avgiftspliktig"
"chart3303","3303","Environmental tax for added/sold goods tax-free","income","","False","Miljø avgift for tilv./solgte varer avgiftsfritt"
"chart3400","3400","Special publicly. dept. manufactured/sold goods","income","","False","Spes. offent. avg. tilvirk./solgte varer"
"chart3440","3440","Special public subsidies for services","income","","False","Spes. offentlige tilskudd for tjenester"
"chart3500","3500","Unearned income guarantee","income","","False","Uopptjente inntekter garanti"
"chart3510","3510","Unearned income service","income","","False","Uopptjente inntekter service"
"chart3600","3600","Rental income real estate","income","","False","Leieinntekter fast eiendom"
"chart3610","3610","Rental income others fixed assets","income","","False","Leieinntekter andre varige driftsmidler"
"chart3620","3620","Others rental income","income","","False","Andre leieinntekter"
"chart3700","3700","Commission income","income","","False","Provisjonsinntekter"
"chart3800","3800","Gain on disposal of fixed assets","income","","False","Gevinst ved avgang av anleggsmidler"
"chart3900","3900","Others operating-related income, taxable","income","","False","Andre driftsrelaterte inntekter, avgiftspliktig"
"chart3910","3910","Outbound postage, subject to tax","income","","False","Utgående porto, avgiftspliktig"
"chart3920","3920","Outgoing fees, taxable","income","","False","Utgående gebyrer, avgiftspliktig"
"chart3950","3950","Other operating income, tax-free","income","","False","Annen driftsrelatert inntekt, avgiftsfritt"
"chart3960","3960","Outbound postage, free of charge","income","","False","Utgående porto, avgiftsfritt"
"chart3970","3970","Outgoing fees, tax-free","income","","False","Utgående gebyrer, avgiftsfritt"
"chart4000","4000","Purchase of raw materials and semi-finished products at a high rate","expense","","False","Innkjøp av råvarer og halvfabrikata høy sats"
"chart4030","4030","Purchase of raw materials and semi-finished products medium rate","expense","","False","Innkjøp av råvarer og halvfabrikata middels sats"
"chart4035","4035","Purchase of goods and semi-finished products, low tax rate","expense","","False","Innkjøp varer og halvfabrikata, lav avgiftssats"
"chart4060","4060","Freight, customs and shipping","expense","","False","Frakt, toll og spedisjon"
"chart4070","4070","Purchase price reduction","expense","","False","Innkjøpsprisreduksjon"
"chart4090","4090","Inventory change","expense","","False","Beholdningsendring"
"chart4100","4100","Purchasing goods under production at a high rate","expense","","False","Innkjøp varer under tilvirkning høy sats"
"chart4130","4130","Purchasing goods under production medium rate","expense","","False","Innkjøp varer under tilvirkning middels sats"
"chart4160","4160","Freight, customs and shipping","expense","","False","Frakt, toll og spedisjon"
"chart4170","4170","Purchase price reduction","expense","","False","Innkjøpsprisreduksjon"
"chart4190","4190","Inventory change","expense","","False","Beholdningsendring"
"chart4200","4200","Purchase ready-made self-made goods at a high rate","expense","","False","Innkjøp ferdig egentilvirkede varer høy sats"
"chart4230","4230","Purchase ready-made self-made goods at a medium rate","expense","","False","Innkjøp ferdig egentilvirkede varer middels sats"
"chart4260","4260","Freight, customs and shipping","expense","","False","Frakt, toll og spedisjon"
"chart4270","4270","Purchase price reduction, subject to tax","expense","","False","Innkjøpsprisreduksjon, avgiftspliktig"
"chart4290","4290","Inventory change","expense","","False","Beholdningsendring"
"chart4300","4300","Purchasing goods for resale at a high rate","expense","","False","Innkjøp varer for videresalg høy sats"
"chart4330","4330","Purchase goods for resale medium rate","expense","","False","Innkjøp varer for videresalg middels sats"
"chart4360","4360","Freight, duty, etc. relating to the purchase of goods for resale","expense","","False","Frakt, toll m.m. vedr. innkjøp av varer for videresalg"
"chart4370","4370","Discounts, etc. regarding the purchase of goods for resale","expense","","False","Rabatter m.m. vedr. innkjøp av varer for videresalg"
"chart4380","4380","Cost of goods","expense","","False","Varekostnad"
"chart4390","4390","Inventory change goods for resale","expense","","False","Beholdningsendring varer for videresalg"
"chart4400","4400","Free Purchase","expense","","False","Fritt Kjøp"
"chart4470","4470","Purchase price reductions, free","expense","","False","Innkjøpsprisreduksjoner, fritt"
"chart4500","4500","Foreign trade and subcontracting","expense","","False","Fremmedytelser og underentreprise"
"chart4590","4590","Inventory change","expense","","False","Beholdningsendring"
"chart4600","4600","Packaging materials","expense","","False","Emballasjematerialer"
"chart4690","4690","Inventory change 2","expense","","False","Beholdningsendring 2"
"chart4800","4800","Exp. fee required, purchase of goods","expense","","False","Eksp. gebyr pliktig, vareinnkjøp"
"chart4810","4810","Freight and postage payable, goods purchased","expense","","False","Frakt og porto pliktig, vareinnkjøp"
"chart4860","4860","Exp. free of charge, purchase of goods","expense","","False","Eksp. gebyr fritt, vareinnkjøp"
"chart4900","4900","Other accruals","expense","","False","Annen periodisering"
"chart4910","4910","Adjustment of stock","expense","","False","Justering av lager"
"chart4990","4990","Inventory change","expense","","False","Beholdningsendring"
"chart5000","5000","Salary to employees","expense","","False","Lønn til ansatte"
"chart5005","5005","Agreed tariff allowances","expense","","False","Avtalte tariffgodtgjørelser"
"chart5090","5090","Salary accrual account","expense","","False","Periodiseringskonto lønn"
"chart5091","5091","Accrued, unpaid wages","expense","","False","Påløpt, ikke utbetalt lønn"
"chart5092","5092","Vacation money","expense","","False","Feriepenger"
"chart5099","5099","Others salary postings","expense","","False","Andre lønnsposteringer"
"chart5100","5100","Salary to employees, hourly employees","expense","","False","Lønn til ansatte, timeansatte"
"chart5105","5105","Agreed collective benefits, hourly employees","expense","","False","Avtalte tariffgodtgjørelser, timeansatte"
"chart5180","5180","Holiday pay calculated","expense","","False","Feriepenger beregnet"
"chart5182","5182","Employer's tax accrued holiday pay","expense","","False","Arbeidsgiveravgift påløpte feriepenger"
"chart5191","5191","Accrued, not paid salary hourly employees","expense","","False","Påløpt, ikke utbetalt lønn timeansatte"
"chart5192","5192","Holiday pay, hourly employees","expense","","False","Feriepenger, timeaansatte"
"chart5199","5199","Others salary postings, hourly employees","expense","","False","Andre lønnsposteringer, timeansatte"
"chart5200","5200","Free car","expense","","False","Fri bil"
"chart5210","5210","Free telephone","expense","","False","Fri telefon"
"chart5220","5220","Free newspaper","expense","","False","Fri avis"
"chart5230","5230","Free lodging and accommodation","expense","","False","Fri losji og bolig"
"chart5240","5240","Interest advantage","expense","","False","Rentefordel"
"chart5251","5251","Group life insurance","expense","","False","Gruppelivsforsikring"
"chart5252","5252","Accident insurance","expense","","False","Ulykkesforsikring"
"chart5260","5260","Dirt surcharge","expense","","False","Smusstillegg"
"chart5280","5280","Others benefits in employment","expense","","False","Andre fordeler i arbeidsforhold"
"chart5290","5290","Counter account for group 52","expense","","False","Motkonto for gruppe 52"
"chart5300","5300","Royalties","expense","","False","Tantieme"
"chart5330","5330","Approved to the board and company assembly","expense","","False","Godtgj. til styre- og bedriftsforsamling"
"chart5390","5390","Other compulsory remuneration","expense","","False","Annen oppgavepliktig godtgjørelse"
"chart5395","5395","Other compulsory remuneration, tax-free","expense","","False","Annen oppgavepliktig godtgjørelse, trekkfri"
"chart5400","5400","Employer's tax","expense","","False","Arbeidsgiveravgift"
"chart5405","5405","Employer tax on accrued holiday pay","expense","","False","Arbeidsgiveravgift av påløpte feriepenger"
"chart5411","5411","Employer's tax accrued on holidays","expense","","False","Arb.giv.avg. pål. feriep."
"chart5420","5420","Reportable pension costs","expense","","False","Innberetningspliktige pensjonskostnader"
"chart5430","5430","Premium pension scheme","expense","","False","Premie pensjonsordning"
"chart5500","5500","Others cost allowances","expense","","False","Andre kostnadsgodtgjørelser"
"chart5510","5510","Overtime food by bill","expense","","False","Overtidsmat etter regning"
"chart5520","5520","Canteen costs","expense","","False","Kantinekostnader"
"chart5600","5600","Work compensation for owners in DLS","expense","","False","Arbeidsgodtgjørelse til eiere i DLS"
"chart5700","5700","Apprentice allowance","expense","","False","Lærlingtilskudd"
"chart5800","5800","Reimbursement of sickness benefits","expense","","False","Refusjon av sykepenger"
"chart5820","5820","Reimbursement of employer's contribution","expense","","False","Refusjon av arbeidsgiveravgift"
"chart5890","5890","Other refunds","expense","","False","Annen refusjon"
"chart5900","5900","Gifts for employees","expense","","False","Gaver til ansatte"
"chart5910","5910","Canteen costs","expense","","False","Kantinekostnader"
"chart5920","5920","Occupational injury insurance","expense","","False","Yrkesskadeforsikring"
"chart5930","5930","Others non-employment insurance.","expense","","False","Andre ikke arb.giv.avg.pliktige forsikr."
"chart5940","5940","Other personnel costs","expense","","False","Øvrige personalkostnader"
"chart5950","5950","Compulsory service pension (OTP)","expense","","False","Obligatorisk tjenestepensjon (OTP)"
"chart5990","5990","Other personnel costs","expense","","False","Øvrige personalkostnader"
"chart6000","6000","Depreciation on buildings & other fixed assets","expense","","False","Avskrivning på bygn. & annen fast eiend."
"chart6010","6010","Depreciation on means of transport, machinery","expense","","False","Avskrivning på transportmidler, maskiner"
"chart6020","6020","Depreciation of intangible assets","expense","","False","Avskrivning på immaterielle eiendeler"
"chart6050","6050","Decrement fixed assets. & food. property","expense","","False","Nedskr. varige driftsmidl. & imat. eiend"
"chart6100","6100","Freight, transport costs and insurance","expense","","False","Frakter, transportkostnader og forsikring"
"chart6110","6110","Customs and forwarding costs for shipping","expense","","False","Toll og spedisjonskostnader ved forsend"
"chart6200","6200","Electricity","expense","","False","Elektrisitet"
"chart6210","6210","Gas","expense","","False","Gass"
"chart6220","6220","Fuel oil","expense","","False","Fyringsolje"
"chart6230","6230","Coal, coke","expense","","False","Kull, koks"
"chart6240","6240","By","expense","","False","Ved"
"chart6250","6250","Gasoline, diesel oil","expense","","False","Bensin, dieselolje"
"chart6260","6260","Water","expense","","False","Vann"
"chart6290","6290","Other fuel","expense","","False","Annen brensel"
"chart6300","6300","Rent premises","expense","","False","Leie lokaler"
"chart6320","6320","Renovation, water, sewage etc.","expense","","False","Renovasjon, vann, avløp mv."
"chart6340","6340","Light, heat","expense","","False","Lys, varme"
"chart6360","6360","Cleaning","expense","","False","Renhold"
"chart6390","6390","Other cost premises","expense","","False","Annen kostnad lokaler"
"chart6400","6400","Rent of operating assets","expense","","False","Leie av driftsmidler"
"chart6410","6410","Rent of fixtures","expense","","False","Leie av inventar"
"chart6420","6420","Rent computer systems","expense","","False","Leie datasystemer"
"chart6430","6430","Rent others office machines","expense","","False","Leie andre kontormaskiner"
"chart6440","6440","Rent means of transport","expense","","False","Leie transportmidler"
"chart6490","6490","Other rental costs","expense","","False","Annen leiekostnad"
"chart6500","6500","Motor-driven tool","expense","","False","Motordrevet verktøy"
"chart6510","6510","Hand tools","expense","","False","Håndverktøy"
"chart6520","6520","Auxiliary tools","expense","","False","Hjelpeverktøy"
"chart6530","6530","Special tools","expense","","False","Spesialverktøy"
"chart6540","6540","Inventory","expense","","False","Inventar"
"chart6550","6550","Operating materials","expense","","False","Driftsmaterialer"
"chart6560","6560","Props","expense","","False","Rekvisita"
"chart6570","6570","Work clothes and protective equipment","expense","","False","Arbeidsklær og verneutstyr"
"chart6590","6590","Other operating equipment","expense","","False","Annet driftsmateriel"
"chart6600","6600","Repairs and maintenance buildings","expense","","False","Reparasjoner og vedlikehold bygninger"
"chart6620","6620","Repairs and maintenance equipment","expense","","False","Reparasjoner og vedlikehold utstyr"
"chart6690","6690","Repair and maintenance other","expense","","False","Reparasjon og vedlikehold annet"
"chart6700","6700","Audit fee","expense","","False","Revisjonshonorar"
"chart6720","6720","Fees for financial & legal assistance","expense","","False","Honorar for økonomisk & juridisk bistand"
"chart6750","6750","Fee accountant","expense","","False","Honorar regnskapsfører"
"chart6785","6785","Purchase of services from abroad","expense","","False","Kjøp av tjenester fra utlandet"
"chart6800","6800","Office supplies","expense","","False","Kontorrekvisita"
"chart6810","6810","Data cost","expense","","False","Datakostnad"
"chart6820","6820","Printed matter","expense","","False","Trykksaker"
"chart6840","6840","Newspapers, magazines, books etc.","expense","","False","Aviser, tidsskrifter, bøker mv."
"chart6860","6860","Meetings, courses, updates etc.","expense","","False","Møter, kurs, oppdatering mv."
"chart6890","6890","Other office costs","expense","","False","Annen kontorkostnad"
"chart6900","6900","Telephone","expense","","False","Telefon"
"chart6901","6901","Phone free","expense","","False","Telefon fritt"
"chart6940","6940","Postage","expense","","False","Porto"
"chart7000","7000","Fuel","expense","","False","Drivstoff"
"chart7020","7020","Maintenance","expense","","False","Vedlikehold"
"chart7040","7040","Insurances","expense","","False","Forsikringer"
"chart7080","7080","Car costs, use of private car in business","expense","","False","Bilkostnader, bruk av privat bil i næring"
"chart7090","7090","Other cost means of transport","expense","","False","Annen kostnad transportmidler"
"chart7100","7100","Car allowance, compulsory","expense","","False","Bilgodtgjørelse, oppgavepliktig"
"chart7130","7130","Travel expenses, duty bound","expense","","False","Reisekostnader, oppgavepliktige"
"chart7140","7140","Travel costs, not compulsory","expense","","False","Reisekostnader, ikke oppgavepliktig"
"chart7150","7150","Dietary expenses, duty bound","expense","","False","Diettkostnader, oppgaveplikig"
"chart7160","7160","Diet costs, not mandatory","expense","","False","Diettkostnader, ikke oppgavepliktig"
"chart7190","7190","Other cost allowance","expense","","False","Annen kostnadsgodtgjørelse"
"chart7200","7200","Commission costs, duty bound","expense","","False","Provisjonskostnader, oppgavepliktige"
"chart7210","7210","Commission costs, not mandatory","expense","","False","Provisjonskostnader, ikke oppgavepliktig"
"chart7300","7300","Selling costs","expense","","False","Salgskostnader"
"chart7320","7320","Advertising costs","expense","","False","Reklamekostnader"
"chart7350","7350","Representation, deductible","expense","","False","Representasjon, fradragsberettiget"
"chart7360","7360","Representation, not deductible","expense","","False","Representasjon, ikke fradragsberettiget"
"chart7390","7390","VAT rounding","expense","","False","MVA-øreavrunding"
"chart7395","7395","Rounding off","expense","","False","Øreavrunding"
"chart7400","7400","Contributions and gifts","expense","","False","Kontingenter og gaver"
"chart7410","7410","Contributions, not deductible","expense","","False","Kontingenter, ikke fradragsberettiget"
"chart7420","7420","Gifts, tax deductible","expense","","False","Gaver, fradragsberettigede"
"chart7430","7430","Gifts, not deductions","expense","","False","Gaver, ikke fradrag"
"chart7500","7500","Insurance premiums","expense","","False","Forsikringspremier"
"chart7550","7550","Warranty and service costs","expense","","False","Garanti- og servicekostnader"
"chart7560","7560","Service costs","expense","","False","Servicekostnader"
"chart7600","7600","License fees and royalties","expense","","False","Lisenesavgifter og royalties"
"chart7610","7610","Patent cost for own patent","expense","","False","Patentkostnad ved egen patent"
"chart7620","7620","Costs of trademarks etc.","expense","","False","Kostnader ved varemerker o.l."
"chart7630","7630","Inspection, sample and stamp duties","expense","","False","Kontroll-,prøve- og stempelavgifter"
"chart7700","7700","Board and company assembly meetings","expense","","False","Styre- og bedriftsforsamlingsmøter"
"chart7710","7710","General Assembly","expense","","False","Generalforsamling"
"chart7730","7730","Costs of own shares","expense","","False","Kostnader ved egne aksjer"
"chart7740","7740","Rounding off, VAT - settlement","expense","","False","Øreavrunding, MVA - oppgjør"
"chart7745","7745","Rounding off, subject to tax","expense","","False","Øreavrunding, avgiftspliktig"
"chart7746","7746","Rounding off, free of charge","expense","","False","Øreavrunding, avgiftsfritt"
"chart7750","7750","Property and party tax","expense","","False","Eiendoms- og festeavgift"
"chart7770","7770","Bank and card fees","expense","","False","Bank og kortgebyrer"
"chart7780","7780","Interest and collection fees","expense","","False","Renter og gebyrer inkasso"
"chart7798","7798","Other cost, deductible","expense","","False","Annen kostnad, fradragsberettiget"
"chart7799","7799","Other cost, not deductible","expense","","False","Annen kostnad, ikke fradragsberettiget"
"chart7800","7800","Loss on disposal of operating assets","expense","","False","Tap ved avgang driftsmidler"
"chart7820","7820","Income from previously written down losses","expense","","False","Innkommet på tidligere nedskrevne fordri"
"chart7830","7830","Bad debts","expense","","False","Tap på fordringer"
"chart7850","7850","Loss due to fire damage","expense","","False","Tap pga. brannskade"
"chart7860","7860","Loss on contracts","expense","","False","Tap på kontrakter"
"chart7900","7900","Inventory change facility under construction","expense","","False","Beholdningsendring anlegg under utførelse"
"chart7910","7910","Outdated goods","expense","","False","Ukurante varer"
"chart8000","8000","Income from investments in subsidiaries","expense","","False","Inntekter på investeringer i datterselskap"
"chart8010","8010","Income from investment in another company. s/ group","expense","","False","Inntekt på investering i annet foretak. s/ konsern"
"chart8020","8020","Income from investment in affiliated companies","expense","","False","Inntekt på investering i tilknyttet selskap"
"chart8030","8030","Interest income on companies in the same group","expense","","False","Renteinntekt på foretak i  samme konsern"
"chart8040","8040","Interest income, tax-free","expense","","False","Renteinntekter, skattefrie"
"chart8050","8050","Other interest income","expense","","False","Annen renteinntekt"
"chart8056","8056","Reminder fee","expense","","False","Påminnelsesavgift"
"chart8060","8060","Currency gain (agio)","expense","","False","Valutagevinst (agio)"
"chart8065","8065","Exchange rate gains, revenue","expense","","False","Valutakursvinster, revenue"
"chart8070","8070","Other financial income","expense","account.account_tag_financing","False","Annen finansinntekt"
"chart8071","8071","Stock dividends","expense","","False","Aksjeutbytte"
"chart8078","8078","Profit realization of shares","expense","","False","Gevinst realisasjon av aksjer"
"chart8080","8080","Value addition finance current assets","expense","account.account_tag_financing","False","Verdiøkning finanseille omløpsmidler"
"chart8090","8090","Income from others investments","expense","","False","Inntekt på andre investeringer"
"chart8100","8100","Valuable of market base finance circulating","expense","account.account_tag_financing","False","Verdired. av markedsbas.finans. omløps."
"chart8110","8110","Write down. of other financial transactions.","expense","account.account_tag_financing","False","Nedskrivn. av andre finansielle omløps."
"chart8120","8120","Impairment of financial fixed assets","expense","account.account_tag_financing","False","Nedskrivning av finansielle anleggsmidl."
"chart8130","8130","Interest cost companies in the same group","expense","","False","Rentekostnad foretak i samme konsern"
"chart8140","8140","Interest costs, not deductible","expense","","False","Rentekostnader, ikke fradragsberettigede"
"chart8150","8150","Other interest expenses","expense","","False","Annen rentekostnad"
"chart8160","8160","Currency loss (disagio)","expense","","False","Valutatap (disagio)"
"chart8178","8178","Loss on realization of shares","expense","","False","Tap ved realisasjon av aksjer"
"chart8179","8179","Other financial cost","expense","","False","Annen finanskostnad"
"chart8300","8300","Payable tax","expense","","False","Betalbar skatt"
"chart8320","8320","Deferred tax","expense","","False","Utsatt skatt"
"chart8350","8350","Tax cost","expense","","False","Skattekostnad"
"chart8400","8400","Extraordinary income","expense","","False","Ekstraordinær inntekt"
"chart8500","8500","Extraordinary cost","expense","","False","Ekstraordinær kostnad"
"chart8600","8600","Payable tax, extraordinary result","expense","","False","Betalbar skatt, ekstraordinært resultat"
"chart8620","8620","Deferred tax, extraordinary result","expense","","False","Utsatt skatt, ekstraordinært resultat"
"chart8800","8800","Annual result","expense","","False","Årsresultat"
"chart8900","8900","Transfers fund for valuation difference","expense","","False","Overføringer fond for vurderingsforskjel"
"chart8910","8910","Transfers jointly owned share capital for","expense","","False","Overføringer felleseid andelskapital for"
"chart8920","8920","Allocated dividends/interest on basic fund certificates","expense","","False","Avsatt utbytte/renter på grunnfondsbevis"
"chart8922","8922","Allocated additional dividend","expense","","False","Avsatt tilleggsutbytte"
"chart8923","8923","Set aside extraordinary dividend","expense","","False","Avsatt ekstraordinært utbytte"
"chart8930","8930","Group contribution","expense","","False","Konsernbidrag"
"chart8940","8940","Shareholder contribution","expense","","False","Aksjonærbidrag"
"chart8950","8950","Fund issue","expense","","False","Fondsemisjon"
"chart8960","8960","Transfers other equity","expense","","False","Overføringer annen egenkapital"
"chart8980","8980","Allocated to unrestricted equity","expense","","False","Avsatt til fri egenkapital"
"chart8990","8990","Uncovered loss","expense","","False","Udekket tap"
"chart9970","9970","Expense account on product template","expense","","False","Utgiftskonto på produktmal"
"chart9990","9990","Income account on product template","income","","False","Inntektskonto på produktmal"

```

## File: data\template\account.tax-no.csv

```csv
"id","name","sequence","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/factor_percent","repartition_line_ids/tag_ids","description@nb_NO","invoice_label@nb_NO"
"tax2","25%","0","1 Input VAT high rate 25%","Deduction for incoming VAT","25.0","percent","purchase","tax_group_25","base","invoice","","","","1 Inngående mva høy sats 25%","Fradrag for inngående mva"
"","","","","","","","","","tax","invoice","chart2711","","+1 Tax","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","chart2711","","-1 Tax","",""
"tax1","0%","","0 No VAT treatment 0%","No VAT treatment (acquisitions)","0.0","percent","purchase","tax_group_0","base","invoice","","","","0 Ingen mvabehandling 0%","Ingen mvabehandling(anskaffelser)"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax3","25%","","3 Output VAT high rate 25%","Output VAT","25.0","percent","sale","tax_group_25","base","invoice","","","+3 Base","3 Utgående mva høy sats 25%","Utgående mva"
"","","","","","","","","","tax","invoice","chart2701","","+3 Tax","",""
"","","","","","","","","","base","refund","","","-3 Base","",""
"","","","","","","","","","tax","refund","chart2701","","-3 Tax","",""
"tax4","0%","","5 VAT-free sales 0%","VAT-free sale","0.0","percent","sale","tax_group_0","base","invoice","","","+5 Base","5 Mvafritt salg 0%","Mvafritt salg"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","-5 Base","",""
"","","","","","","","","","tax","refund","","","","",""
"tax5","0% T","","6 Turnover outside the VAT Act 0%","Turnover outside the VAT Act","0.0","percent","sale","tax_group_0","base","invoice","","","+6 Base","6 Omsetning utenfor mvaloven 0%","Omsetning utenfor merverdiavgiftsloven"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","-6 Base","",""
"","","","","","","","","","tax","refund","","","","",""
"tax6","0% EXEMPT","","7 No VAT treatment (income) 0%","No VAT treatment (income)","0.0","percent","sale","tax_group_0","base","invoice","","","","7 Ingen mvabehandling(inntekter) 0%","Ingen mvabehandling(inntekter)"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax7","15%","","11 Input VAT average rate 15%","Deduction for incoming VAT","15.0","percent","purchase","tax_group_15","base","invoice","","","","11 Inngående mva middel sats 15%","Fradrag for inngående mva"
"","","","","","","","","","tax","invoice","chart2713","","+11 Tax","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","chart2713","","-11 Tax","",""
"tax8","11,11% Fish","","12 Including VAT raw fish 11%","Deduction for incoming VAT","11.11","percent","purchase","tax_group_15","base","invoice","","","","12 Inngående mva råfisk 11%","Fradrag for inngående mva"
"","","","","","","","","","tax","invoice","chart2710","","+12 Tax","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","chart2710","","-12 Tax","",""
"tax9","12%","","13 Input VAT low rate 12%","Deduction for incoming VAT","12.0","percent","purchase","tax_group_12","base","invoice","","","","13 Inngående mva lav sats 12%","Fradrag for inngående mva"
"","","","","","","","","","tax","invoice","chart2714","","+13 Tax","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","chart2714","","-13 Tax","",""
"tax10","25% EX","","14 Import VAT high rate 25%","Deduction for import VAT","25.0","percent","purchase","tax_group_25","base","invoice","","","","14 Innførselsmva høy sats 25%","Fradrag for innførselsmva"
"","","","","","","","","","tax","invoice","chart2711","","+14 Tax","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","chart2711","","-14 Tax","",""
"tax11","15% EX","","15 Import VAT average rate 15%","Deduction for import VAT","15.0","percent","purchase","tax_group_15","base","invoice","","","","15 Innførselsmva middel sats 15%","Fradrag for innførselsmva"
"","","","","","","","","","tax","invoice","chart2713","","+15 Tax","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","chart2713","","-15 Tax","",""
"tax12","0% EX G","","20 Basis for importing goods zero rate 0%","Basis for importing goods","0.0","percent","purchase","tax_group_0","base","invoice","","","","20 Grunnlag ved innførsel av varer nullsats 0%","Grunnlag ved innførsel av varer"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax13","25% EX G","","21 Basis for importing goods high rate 25%","Basis for importing goods","25.0","percent","purchase","tax_group_25","base","invoice","","","","21 Grunnlag ved innførsel av varer høy sats 25%","Grunnlag ved innførsel av varer"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax14","15% EX G","","22 Basis for importing goods average rate 15%","Basis for importing goods","15.0","percent","purchase","tax_group_15","base","invoice","","","","22 Grunnlag ved innførsel av varer middel sats 15%","Grunnlag ved innførsel av varer"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax15","15%","","31 Output VAT average rate 15%","Output VAT","15.0","percent","sale","tax_group_15","base","invoice","","","+31 Base","31 Utgående mva middel sats 15%","Utgående mva"
"","","","","","","","","","tax","invoice","chart2703","","+31 Tax","",""
"","","","","","","","","","base","refund","","","-31 Base","",""
"","","","","","","","","","tax","refund","chart2703","","-31 Tax","",""
"tax16","11% Fish","","32 Output VAT raw fish 11%","Output VAT","11.0","percent","sale","tax_group_15","base","invoice","","","+32 Base","32 Utgående mva råfisk 11%","Utgående mva"
"","","","","","","","","","tax","invoice","chart2700","","+32 Tax","",""
"","","","","","","","","","base","refund","","","-32 Base","",""
"","","","","","","","","","tax","refund","chart2700","","-32 Tax","",""
"tax17","12%","","33 Output VAT low rate 12%","Output VAT","12.0","percent","sale","tax_group_12","base","invoice","","","+33 Base","33 Utgående mva lav sats 12%","Utgående mva"
"","","","","","","","","","tax","invoice","chart2704","","+33 Tax","",""
"","","","","","","","","","base","refund","","","-33 Base","",""
"","","","","","","","","","tax","refund","chart2704","","-33 Tax","",""
"tax18","0% R","","51 Domestic turnover with reverse tax obligation zero rate 0%","Inland turnover with reverse tax liability","0.0","percent","sale","tax_group_0","base","invoice","","","+51 Base","51 Innenlands omsetning med omvendt avgiftsplikt nullsats 0%","Innlands omsetning med omvendt avgiftsplikt"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","-51 Base","",""
"","","","","","","","","","tax","refund","","","","",""
"tax19","0% EX","","52 Export of goods and services zero rate 0%","Export of goods and services","0.0","percent","sale","tax_group_0","base","invoice","","","+52 Base","52 Utførsel av varer og tjenester nullsats 0%","Utførsel av varer og tjenester"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","-52 Base","",""
"","","","","","","","","","tax","refund","","","","",""
"tax20","25% EX G D","","81 Import of goods with deduction for import VAT high rate 25%","Import of goods with deduction for import VAT","25.0","percent","purchase","tax_group_25","base","invoice","","","+81 Base","81 Innførsel av varer med fradrag for innførselsmva høy sats 25%","Innførsel av varer med fradrag for innførselsmva"
"","","","","","","","","","tax","invoice","chart2741","","+81 Tax","",""
"","","","","","","","","","tax","invoice","chart2727","-100","+81 Tax","",""
"","","","","","","","","","base","refund","","","-81 Base","",""
"","","","","","","","","","tax","refund","chart2741","","-81 Tax","",""
"","","","","","","","","","tax","refund","chart2727","-100","-81 Tax","",""
"tax21","25% EX G ND","","82 Import of goods without deduction for import VAT high rate 25%","Import of goods without deduction for import VAT","25.0","percent","purchase","tax_group_25","base","invoice","","","+82 Base","82 Innførsel av varer uten fradrag for innførselsmva høy sats 25%","Innførsel av varer uten fradrag for innførselsmva"
"","","","","","","","","","tax","invoice","chart2741","","+82 Tax","",""
"","","","","","","","","","base","refund","","","-82 Base","",""
"","","","","","","","","","tax","refund","chart2741","","-82 Tax","",""
"tax22","15% EX G D","","83 Import of goods with deduction for import VAT average rate 15%","Import of goods with deduction for import VAT","15.0","percent","purchase","tax_group_15","base","invoice","","","+83 Base","83 Innførsel av varer med fradrag for innførselsmva middel sats 15%","Innførsel av varer med fradrag for innførselsmva"
"","","","","","","","","","tax","invoice","chart2742","","+83 Tax","",""
"","","","","","","","","","tax","invoice","chart2728","-100","+83 Tax","",""
"","","","","","","","","","base","refund","","","-83 Base","",""
"","","","","","","","","","tax","refund","chart2742","","-83 Tax","",""
"","","","","","","","","","tax","refund","chart2728","-100","-83 Tax","",""
"tax23","15% EX G ND","","84 Import of goods without deduction for import VAT average rate 15%","Import of goods without deduction for import VAT","15.0","percent","purchase","tax_group_15","base","invoice","","","+84 Base","84 Innførsel av varer uten fradrag for innførselsmva middel sats 15%","Innførsel av varer uten fradrag for innførselsmva"
"","","","","","","","","","tax","invoice","chart2742","","+84 Tax","",""
"","","","","","","","","","base","refund","","","-84 Base","",""
"","","","","","","","","","tax","refund","chart2742","","-84 Tax","",""
"tax24","0% EX","","85 Import of goods without VAT calculation 0%","Import of goods for which no additional duty shall be calculated","0.0","percent","purchase","tax_group_0","base","invoice","","","+85 Base","85 Innførsel av varer uten mva beregning 0%","Innførsel av varer som det ikke skal beregnes mervediavgift av"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","-85 Base","",""
"","","","","","","","","","tax","refund","","","","",""
"tax25","25% EX S D","","86 Services purchased from abroad with deduction for VAT high rate 25%","Services purchased from abroad with deduction of VAT","25.0","percent","purchase","tax_group_25","base","invoice","","","+86 Base","86 Tjenester kjøpt fra utlandet med fradrag for mva høy sats 25%","Tjenester kjøpt fra utlandet med fradrag for mva"
"","","","","","","","","","tax","invoice","chart2712","","+86 Tax","",""
"","","","","","","","","","tax","invoice","chart2702","-100","+86 Tax","",""
"","","","","","","","","","base","refund","","","-86 Base","",""
"","","","","","","","","","tax","refund","chart2712","","-86 Tax","",""
"","","","","","","","","","tax","refund","chart2702","-100","-86 Tax","",""
"tax26","25% EX S ND","","87 Services purchased from abroad without deduction for VAT high rate 25%","Services purchased from abroad without VAT deduction","25.0","percent","purchase","tax_group_25","base","invoice","","","+87 Base","87 Tjenester kjøpt fra utlandet uten fradrag for mva høy sats 25%","Tjenester kjøpt fra utlandet uten fradrag for mva"
"","","","","","","","","","tax","invoice","chart2712","","+87 Tax","",""
"","","","","","","","","","base","refund","","","-87 Base","",""
"","","","","","","","","","tax","refund","chart2712","","-87 Tax","",""
"tax27","12% EX S D","","88 Services purchased from abroad with deduction for VAT low rate 12%","Services purchased from abroad with deduction of VAT","12.0","percent","purchase","tax_group_12","base","invoice","","","+88 Base","88 Tjenester kjøpt fra utlandet med fradrag for mva lav sats 12%","Tjenester kjøpt fra utlandet med fradrag for mva"
"","","","","","","","","","tax","invoice","chart2712","","+88 Tax","",""
"","","","","","","","","","tax","invoice","chart2702","-100","+88 Tax","",""
"","","","","","","","","","base","refund","","","-88 Base","",""
"","","","","","","","","","tax","refund","chart2712","","-88 Tax","",""
"","","","","","","","","","tax","refund","chart2702","-100","-88 Tax","",""
"tax28","12% EX S ND","","89 Services purchased from abroad without deduction for VAT low rate 12%","Services purchased from abroad without VAT deduction","12.0","percent","purchase","tax_group_12","base","invoice","","","+89 Base","89 Tjenester kjøpt fra utlandet uten fradrag for mva lav sats 12%","Tjenester kjøpt fra utlandet uten fradrag for mva"
"","","","","","","","","","tax","invoice","chart2712","","+89 Tax","",""
"","","","","","","","","","base","refund","","","-89 Base","",""
"","","","","","","","","","tax","refund","chart2712","","-89 Tax","",""
"tax29","25% C G D","","91 Purchase of carbon quotas or gold with deduction for VAT high rate 25%","Purchase of climate quotas or gold with VAT deduction","25.0","percent","purchase","tax_group_25","base","invoice","","","+91 Base","91 Kjøp av klimakvoter eller gull med fradrag for mva høy sats 25%","Kjøp av klimakvoter eller gull med fradrag for mva"
"","","","","","","","","","tax","invoice","chart2711","","+91 Tax","",""
"","","","","","","","","","tax","invoice","chart2701","-100","+91 Tax","",""
"","","","","","","","","","base","refund","","","-91 Base","",""
"","","","","","","","","","tax","refund","chart2711","","-91 Tax","",""
"","","","","","","","","","tax","refund","chart2701","-100","-91 Tax","",""
"tax30","25% C G ND","","92 Purchase of carbon quotas or gold without deduction for VAT high rate 25%","Purchase of climate quotas or gold without VAT deduction","25.0","percent","purchase","tax_group_25","base","invoice","","","+92 Base","92 Kjøp av klimakvoter eller gull uten fradrag for mva høy sats 25%","Kjøp av klimakvoter eller gull uten fradrag for mva"
"","","","","","","","","","tax","invoice","chart2711","","+92 Tax","",""
"","","","","","","","","","base","refund","","","-92 Base","",""
"","","","","","","","","","tax","refund","chart2711","","-92 Tax","",""

```

## File: data\template\account.tax.group-no.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@nb_NO"
"tax_group_0","VAT 0%","base.no","chart2740","chart2740","MVA 0%"
"tax_group_10","VAT 10%","base.no","chart2740","chart2740","MVA 10%"
"tax_group_12","VAT 12%","base.no","chart2740","chart2740","MVA 12%"
"tax_group_15","VAT 15%","base.no","chart2740","chart2740","MVA 15%"
"tax_group_25","VAT 25%","base.no","chart2740","chart2740","MVA 25%"

```

## File: migrations\2.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'no')], order="parent_path"):
        env['account.chart.template'].try_loading('no', company)

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[
        ('no', 'Norway')
    ], ondelete={'no': lambda recs: recs.write({'invoice_reference_model': 'odoo'})})

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from stdnum import luhn


class AccountMove(models.Model):
    _inherit = "account.move"

    def _get_invoice_reference_no_invoice(self):
        """ This computes the reference based on the Odoo format.
            We calculat reference using invoice number and
            partner id and added control digit at last.
        """
        return self._get_kid_number()

    def _get_invoice_reference_no_partner(self):
        """ This computes the reference based on the Odoo format.
            We calculat reference using invoice number and
            partner id and added control digit at last.
        """
        return self._get_kid_number()

    def _get_kid_number(self):
        self.ensure_one()
        invoice_name = ''.join([i for i in self.name if i.isdigit()]).zfill(7)
        ref = (str(self.partner_id.id).zfill(7)[-7:] + invoice_name[-7:])
        return ref + luhn.calc_check_digit(ref)

```

## File: models\res_company.py

```python
# coding: utf-8
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_no_bronnoysund_number = fields.Char(related='partner_id.l10n_no_bronnoysund_number', readonly=False)

```

## File: models\res_partner.py

```python
# coding: utf-8
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_no_bronnoysund_number = fields.Char(string='Register of Legal Entities (Brønnøysund Register Center)', size=9)

    def _deduce_country_code(self):
        if self.l10n_no_bronnoysund_number:
            return 'NO'
        return super()._deduce_country_code()

    def _peppol_eas_endpoint_depends(self):
        # extends account_edi_ubl_cii
        return super()._peppol_eas_endpoint_depends() + ['l10n_no_bronnoysund_number']

```

## File: models\template_no.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('no')
    def _get_no_template_data(self):
        return {
            'code_digits': '4',
            'property_account_receivable_id': 'chart1500',
            'property_account_payable_id': 'chart2400',
            'property_account_expense_categ_id': 'chart4000',
            'property_account_income_categ_id': 'chart3000',
        }

    @template('no', 'res.company')
    def _get_no_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.no',
                'bank_account_code_prefix': '1920',
                'cash_account_code_prefix': '1900',
                'transfer_account_code_prefix': '1940',
                'account_default_pos_receivable_account_id': 'chart1501',
                'income_currency_exchange_account_id': 'chart8060',
                'expense_currency_exchange_account_id': 'chart8160',
                'account_journal_early_pay_discount_loss_account_id': 'chart4370',
                'account_journal_early_pay_discount_gain_account_id': 'chart3080',
                'account_sale_tax_id': 'tax3',
                'account_purchase_tax_id': 'tax2',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_no
from . import account_journal
from . import account_move
from . import res_partner
from . import res_company

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_company_form_inherit_no" model="ir.ui.view">
            <field name="name">res.company.form.inherit.l10n.no</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="base.view_company_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_no_bronnoysund_number" invisible="country_code != 'NO'"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_no" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_no</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="priority">14</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_no_bronnoysund_number" invisible="country_code != 'NO' or not is_company"/>
            </xpath>
        </field>
    </record>
</odoo>

```

