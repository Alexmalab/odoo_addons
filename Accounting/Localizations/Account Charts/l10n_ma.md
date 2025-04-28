# Odoo Module: l10n_ma

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
    'name': 'Morocco - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ma'],
    'author': 'Odoo SA',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Morocco.

This module has been built with the help of Caudigef.
""",
    'depends': [
        'base',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_tax_report_data.xml',
        "views/res_partner_views.xml",
        "views/res_company_views.xml",
        'views/report_invoice.xml',
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
    <record id="tax_report_vat" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ma"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_column_base" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="tax_report_column_base_tax" model="account.report.column">
                <field name="name">Base Tax</field>
                <field name="expression_label">base_tax</field>
            </record>
            <record id="tax_report_column_pro" model="account.report.column">
                <field name="name">Prorata (%)</field>
                <field name="expression_label">pro</field>
                <field name="figure_type">percentage</field>
            </record>
            <record id="tax_report_column_tax" model="account.report.column">
                <field name="name">Tax Due/Ded.</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_part_a" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">A - Breakdown of total revenues</field>
                <field name="children_ids">
                    <record id="tax_report_line_10" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">10 - Amount of the revenues realized, including non-taxable operations (excl. VAT)</field>
                        <field name="code">l10n_ma_vat_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_20.base + l10n_ma_vat_30.base + l10n_ma_vat_40.base + l10n_ma_vat_50.base + l10n_ma_vat_b.base_sum</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_20" model="account.report.line">
                                <field name="hierarchy_level">5</field>
                                <field name="name">20 - Operations outside the scope of VAT</field>
                                <field name="code">l10n_ma_vat_20</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_20_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_30" model="account.report.line">
                                <field name="hierarchy_level">5</field>
                                <field name="name">30 - Exempt operations without right of deduction (Article 91 of the CGI)</field>
                                <field name="code">l10n_ma_vat_30</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_30_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">30</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_40" model="account.report.line">
                                <field name="hierarchy_level">5</field>
                                <field name="name">40 - Exempt operations with right of deduction (Article 92 of the CGI)</field>
                                <field name="code">l10n_ma_vat_40</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_40_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">40</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_50" model="account.report.line">
                                <field name="hierarchy_level">5</field>
                                <field name="name">50 - Operations carried out under suspension of VAT (Article 94 of the CGI)</field>
                                <field name="code">l10n_ma_vat_50</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_50_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">50</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_55" model="account.report.line">
                                <field name="hierarchy_level">5</field>
                                <field name="name">55 - Taxable operations made by mobile payment (Article 247 ter of the CGI)</field>
                                <field name="code">l10n_ma_vat_55</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_55_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_56" model="account.report.line">
                                <field name="hierarchy_level">5</field>
                                <field name="name">56 - Exempt, out-of-scope or suspended operations by mobile payment</field>
                                <field name="code">l10n_ma_vat_56</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_56_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_60" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">60 - Taxable revenues to be allocated (difference line 10 - (20 + 30 + 40 + 50)) (excl. VAT)</field>
                        <field name="code">l10n_ma_vat_60</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_60_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_10.base - l10n_ma_vat_20.base - l10n_ma_vat_30.base - l10n_ma_vat_40.base - l10n_ma_vat_50.base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_part_b" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">B - Breakdown of taxable revenues</field>
                <field name="code">l10n_ma_vat_b</field>
                <field name="expression_ids">
                    <record id="tax_report_part_b_base_sum" model="account.report.expression">
                        <field name="label">base_sum</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_b_20.base_sum + l10n_ma_vat_b_14.base_sum + l10n_ma_vat_b_10.base_sum + l10n_ma_vat_b_7.base_sum</field>
                    </record>
                    <record id="tax_report_part_b_tax_sum" model="account.report.expression">
                        <field name="label">tax_sum</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_b_20.tax_sum + l10n_ma_vat_b_14.tax_sum + l10n_ma_vat_b_10.tax_sum + l10n_ma_vat_b_7.tax_sum</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_part_b_20" model="account.report.line">
                        <field name="hierarchy_level">1</field>
                        <field name="name">Standard rate of 20% with right to deduct</field>
                        <field name="code">l10n_ma_vat_b_20</field>
                        <field name="expression_ids">
                            <record id="tax_report_part_b_20_base_sum" model="account.report.expression">
                                <field name="label">base_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_80.base + l10n_ma_vat_81.base + l10n_ma_vat_82.base + l10n_ma_vat_83.base + l10n_ma_vat_87.base + l10n_ma_vat_93.base + l10n_ma_vat_94.base + l10n_ma_vat_95.base + l10n_ma_vat_102.base</field>
                            </record>
                            <record id="tax_report_part_b_20_tax_sum" model="account.report.expression">
                                <field name="label">tax_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_80.tax + l10n_ma_vat_81.tax + l10n_ma_vat_82.tax + l10n_ma_vat_83.tax + l10n_ma_vat_87.tax + l10n_ma_vat_93.tax + l10n_ma_vat_94.tax + l10n_ma_vat_95.tax + l10n_ma_vat_102.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_80" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">80 - Production and distribution operations</field>
                                <field name="code">l10n_ma_vat_80</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_80_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">80B</field>
                                    </record>
                                    <record id="tax_report_line_80_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">80T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_81" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">81 - Services</field>
                                <field name="code">l10n_ma_vat_81</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_81_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">81B</field>
                                    </record>
                                    <record id="tax_report_line_81_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">81T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_82" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">82 - Liberal professions referred to in article 89-I-12°-(b) of the CGI</field>
                                <field name="code">l10n_ma_vat_82</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_82_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">82B</field>
                                    </record>
                                    <record id="tax_report_line_82_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">82T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_83" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">83 - Leasing operations</field>
                                <field name="code">l10n_ma_vat_83</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_83_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">83B</field>
                                    </record>
                                    <record id="tax_report_line_83_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">83T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_87" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">87 - Operations of real estate works companies</field>
                                <field name="code">l10n_ma_vat_87</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_87_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">87B</field>
                                    </record>
                                    <record id="tax_report_line_87_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">87T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_93" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">93 - Sales and delivery of used movable property</field>
                                <field name="code">l10n_ma_vat_93</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_93_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">93B</field>
                                    </record>
                                    <record id="tax_report_line_93_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">93T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_94" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">94 - Self-supply of constructions</field>
                                <field name="code">l10n_ma_vat_94</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_94_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">94B</field>
                                    </record>
                                    <record id="tax_report_line_94_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">94T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_95" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">95 - Margin regime referred to in articles 125 bis and 125 quater of the CGI</field>
                                <field name="code">l10n_ma_vat_95</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_95_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">95B</field>
                                    </record>
                                    <record id="tax_report_line_95_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">95T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_102" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">102 - Other</field>
                                <field name="code">l10n_ma_vat_102</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_102_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">102B</field>
                                    </record>
                                    <record id="tax_report_line_102_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">102T</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_part_b_14" model="account.report.line">
                        <field name="hierarchy_level">1</field>
                        <field name="name">Reduced rate of 14%</field>
                        <field name="code">l10n_ma_vat_b_14</field>
                        <field name="expression_ids">
                            <record id="tax_report_part_b_14_base_sum" model="account.report.expression">
                                <field name="label">base_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_b_14_d.base_sum + l10n_ma_vat_b_14_nd.base_sum</field>
                            </record>
                            <record id="tax_report_part_b_14_tax_sum" model="account.report.expression">
                                <field name="label">tax_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_b_14_d.tax_sum + l10n_ma_vat_b_14_nd.tax_sum</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_part_b_14_d" model="account.report.line">
                                <field name="hierarchy_level">2</field>
                                <field name="name">With right to deduct</field>
                                <field name="code">l10n_ma_vat_b_14_d</field>
                                <field name="expression_ids">
                                    <record id="tax_report_part_b_14_d_base_sum" model="account.report.expression">
                                        <field name="label">base_sum</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_84.base + l10n_ma_vat_88.base + l10n_ma_vat_90.base + l10n_ma_vat_104.base</field>
                                    </record>
                                    <record id="tax_report_part_b_14_d_tax_sum" model="account.report.expression">
                                        <field name="label">tax_sum</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_84.tax + l10n_ma_vat_88.tax + l10n_ma_vat_90.tax + l10n_ma_vat_104.tax</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="tax_report_line_84" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">84 - Butter, excluding home-made butter referred to in Article 91 (I-A. 2°)</field>
                                        <field name="code">l10n_ma_vat_84</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_84_base" model="account.report.expression">
                                                <field name="label">base</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">84B</field>
                                            </record>
                                            <record id="tax_report_line_84_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">84T</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_88" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">88 - Passenger and freight transport operations, excluding rail transport</field>
                                        <field name="code">l10n_ma_vat_88</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_88_base" model="account.report.expression">
                                                <field name="label">base</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">88B</field>
                                            </record>
                                            <record id="tax_report_line_88_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">88T</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_90" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">90 - Electrical energy</field>
                                        <field name="code">l10n_ma_vat_90</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_90_base" model="account.report.expression">
                                                <field name="label">base</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">90B</field>
                                            </record>
                                            <record id="tax_report_line_90_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">90T</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_104" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">104 - Other</field>
                                        <field name="code">l10n_ma_vat_104</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_104_base" model="account.report.expression">
                                                <field name="label">base</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">104B</field>
                                            </record>
                                            <record id="tax_report_line_104_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">104T</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_part_b_14_nd" model="account.report.line">
                                <field name="hierarchy_level">2</field>
                                <field name="name">Without right to deduct</field>
                                <field name="code">l10n_ma_vat_b_14_nd</field>
                                <field name="expression_ids">
                                    <record id="tax_report_part_b_14_nd_base_sum" model="account.report.expression">
                                        <field name="label">base_sum</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_91.base</field>
                                    </record>
                                    <record id="tax_report_part_b_14_nd_tax_sum" model="account.report.expression">
                                        <field name="label">tax_sum</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_91.tax</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="tax_report_line_91" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">91 - Services rendered by any soliciting agent or insurance broker in respect of contracts brought by him to an insurance company</field>
                                        <field name="code">l10n_ma_vat_91</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_91_base" model="account.report.expression">
                                                <field name="label">base</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">91B</field>
                                            </record>
                                            <record id="tax_report_line_91_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">91T</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_part_b_10" model="account.report.line">
                        <field name="hierarchy_level">1</field>
                        <field name="name">Reduced rate of 10% with right to deduct</field>
                        <field name="code">l10n_ma_vat_b_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_part_b_10_base_sum" model="account.report.expression">
                                <field name="label">base_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_76.base + l10n_ma_vat_77.base + l10n_ma_vat_78.base + l10n_ma_vat_79.base + l10n_ma_vat_85.base + l10n_ma_vat_86.base + l10n_ma_vat_89.base + l10n_ma_vat_92.base + l10n_ma_vat_96.base + l10n_ma_vat_97.base + l10n_ma_vat_98.base + l10n_ma_vat_99.base + l10n_ma_vat_100.base + l10n_ma_vat_101.base + l10n_ma_vat_103.base + l10n_ma_vat_105.base + l10n_ma_vat_108.base + l10n_ma_vat_109.base + l10n_ma_vat_112.base + l10n_ma_vat_118.base</field>
                            </record>
                            <record id="tax_report_part_b_10_tax_sum" model="account.report.expression">
                                <field name="label">tax_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_76.tax + l10n_ma_vat_77.tax + l10n_ma_vat_78.tax + l10n_ma_vat_79.tax + l10n_ma_vat_85.tax + l10n_ma_vat_86.tax + l10n_ma_vat_89.tax + l10n_ma_vat_92.tax + l10n_ma_vat_96.tax + l10n_ma_vat_97.tax + l10n_ma_vat_98.tax + l10n_ma_vat_99.tax + l10n_ma_vat_100.tax + l10n_ma_vat_101.tax + l10n_ma_vat_103.tax + l10n_ma_vat_105.tax + l10n_ma_vat_108.tax + l10n_ma_vat_109.tax + l10n_ma_vat_112.tax + l10n_ma_vat_118.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_76" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">76 - Photovoltaic panels</field>
                                <field name="code">l10n_ma_vat_76</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_76_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">76B</field>
                                    </record>
                                    <record id="tax_report_line_76_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">76T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_77" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">77 - Solar water heaters</field>
                                <field name="code">l10n_ma_vat_77</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_77_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">77B</field>
                                    </record>
                                    <record id="tax_report_line_77_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">77T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_78" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">78 - Feed for livestock and farmyard animals as well as oil cakes used in their manufacture, excluding other simple feed</field>
                                <field name="code">l10n_ma_vat_78</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_78_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">78B</field>
                                    </record>
                                    <record id="tax_report_line_78_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">78T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_79" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">79 - Equipment for exclusive agricultural use</field>
                                <field name="code">l10n_ma_vat_79</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_79_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">79B</field>
                                    </record>
                                    <record id="tax_report_line_79_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">79T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_85" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">85 - Wood in logs, debarked or simply squared, cork in its natural state, firewood in bundles or sawn to short lengths and charcoal</field>
                                <field name="code">l10n_ma_vat_85</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_85_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">85B</field>
                                    </record>
                                    <record id="tax_report_line_85_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">85T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_86" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">86 - Fishing gear and nets for professional marine fishermen</field>
                                <field name="code">l10n_ma_vat_86</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_86_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">86B</field>
                                    </record>
                                    <record id="tax_report_line_86_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">86T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_89" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">89 - Accommodation and catering operations as well as services provided by cafés</field>
                                <field name="code">l10n_ma_vat_89</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_89_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">89B</field>
                                    </record>
                                    <record id="tax_report_line_89_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">89T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_92" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">92 - Rental operations of buildings used as hotels</field>
                                <field name="code">l10n_ma_vat_92</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_92_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">92B</field>
                                    </record>
                                    <record id="tax_report_line_92_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">92T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_96" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">96 - Cooking salt (rock or sea salt)</field>
                                <field name="code">l10n_ma_vat_96</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_96_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">96B</field>
                                    </record>
                                    <record id="tax_report_line_96_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">96T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_97" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">97 - Milled rice and pasta</field>
                                <field name="code">l10n_ma_vat_97</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_97_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">97B</field>
                                    </record>
                                    <record id="tax_report_line_97_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">97T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_98" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">98 - Food fluid oils excluding palm oil</field>
                                <field name="code">l10n_ma_vat_98</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_98_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">98B</field>
                                    </record>
                                    <record id="tax_report_line_98_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">98T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_99" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">99 - Securities transactions</field>
                                <field name="code">l10n_ma_vat_99</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_99_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">99B</field>
                                    </record>
                                    <record id="tax_report_line_99_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">99T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_100" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">100 - Banking and credit transactions and exchange commissions</field>
                                <field name="code">l10n_ma_vat_100</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_100_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">100B</field>
                                    </record>
                                    <record id="tax_report_line_100_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">100T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_101" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">101 - Transactions involving shares and units shares</field>
                                <field name="code">l10n_ma_vat_101</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_101_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">101B</field>
                                    </record>
                                    <record id="tax_report_line_101_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">101T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_103" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">103 - Operations of sale and delivery of works and objects of art</field>
                                <field name="code">l10n_ma_vat_103</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">103B</field>
                                    </record>
                                    <record id="tax_report_line_103_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">103T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_105" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">105 - Operations of lawyers, interpreters, notaries, adouls, bailiffs and veterinarians</field>
                                <field name="code">l10n_ma_vat_105</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_105_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">105B</field>
                                    </record>
                                    <record id="tax_report_line_105_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">105T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_108" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">108 - Petroleum gas and other gaseous hydrocarbons</field>
                                <field name="code">l10n_ma_vat_108</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_108_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">108B</field>
                                    </record>
                                    <record id="tax_report_line_108_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">108T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_109" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">109 - Petroleum or shale oils, crude or refined</field>
                                <field name="code">l10n_ma_vat_109</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_109_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">109B</field>
                                    </record>
                                    <record id="tax_report_line_109_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">109T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_112" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">112 - Ticket sales operations for museums, cinema and theater</field>
                                <field name="code">l10n_ma_vat_112</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_112_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">112B</field>
                                    </record>
                                    <record id="tax_report_line_112_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">112T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_118" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">118 - Other</field>
                                <field name="code">l10n_ma_vat_118</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_118_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">118B</field>
                                    </record>
                                    <record id="tax_report_line_118_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">118T</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_part_b_7" model="account.report.line">
                        <field name="hierarchy_level">1</field>
                        <field name="name">Reduced rate of 7% with right to deduct</field>
                        <field name="code">l10n_ma_vat_b_7</field>
                        <field name="expression_ids">
                            <record id="tax_report_part_b_7_base_sum" model="account.report.expression">
                                <field name="label">base_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_106.base + l10n_ma_vat_107.base + l10n_ma_vat_110.base + l10n_ma_vat_111.base + l10n_ma_vat_113.base + l10n_ma_vat_114.base + l10n_ma_vat_115.base + l10n_ma_vat_116.base + l10n_ma_vat_117.base + l10n_ma_vat_119.base</field>
                            </record>
                            <record id="tax_report_part_b_7_tax_sum" model="account.report.expression">
                                <field name="label">tax_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_106.tax + l10n_ma_vat_107.tax + l10n_ma_vat_110.tax + l10n_ma_vat_111.tax + l10n_ma_vat_113.tax + l10n_ma_vat_114.tax + l10n_ma_vat_115.tax + l10n_ma_vat_116.tax + l10n_ma_vat_117.tax + l10n_ma_vat_119.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_106" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">106 - Water delivered to the public distribution networks and sanitation services</field>
                                <field name="code">l10n_ma_vat_106</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_106_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">106B</field>
                                    </record>
                                    <record id="tax_report_line_106_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">106T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_107" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">107 - Rental of water and electricity meters</field>
                                <field name="code">l10n_ma_vat_107</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_107_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">107B</field>
                                    </record>
                                    <record id="tax_report_line_107_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">107T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_110" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">110 - Pharmaceutical products, raw materials and products used in their composition as well as non-returnable packaging of these products and materials</field>
                                <field name="code">l10n_ma_vat_110</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_110_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">110B</field>
                                    </record>
                                    <record id="tax_report_line_110_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">110T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_111" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">111 - School supplies, products and materials in their composition</field>
                                <field name="code">l10n_ma_vat_111</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_111_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">111B</field>
                                    </record>
                                    <record id="tax_report_line_111_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">111T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_113" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">113 - Refined or agglomerated sugar</field>
                                <field name="code">l10n_ma_vat_113</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_113_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">113B</field>
                                    </record>
                                    <record id="tax_report_line_113_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">113T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_114" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">114 - Canned sardines</field>
                                <field name="code">l10n_ma_vat_114</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_114_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">114B</field>
                                    </record>
                                    <record id="tax_report_line_114_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">114T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_115" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">115 - Milk powder</field>
                                <field name="code">l10n_ma_vat_115</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_115_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">115B</field>
                                    </record>
                                    <record id="tax_report_line_115_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">115T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_116" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">116 - Household soap</field>
                                <field name="code">l10n_ma_vat_116</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_116_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">116B</field>
                                    </record>
                                    <record id="tax_report_line_116_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">116T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_117" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">117 - Economy car and products and materials used in its in its manufacture</field>
                                <field name="code">l10n_ma_vat_117</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_117_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">117B</field>
                                    </record>
                                    <record id="tax_report_line_117_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">117T</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_119" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">119 - Other</field>
                                <field name="code">l10n_ma_vat_119</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_119_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">119B</field>
                                    </record>
                                    <record id="tax_report_line_119_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">119T</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_120" model="account.report.line">
                        <field name="hierarchy_level">2</field>
                        <field name="name">120 - Reversal of VAT in different ways (cessation, regularization, ...)</field>
                        <field name="code">l10n_ma_vat_120</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_120_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_121" model="account.report.line">
                        <field name="hierarchy_level">2</field>
                        <field name="name">121 - Withholding tax on the amount of commissions allocated by insurance companies to their brokers</field>
                        <field name="code">l10n_ma_vat_121</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_121_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_122" model="account.report.line">
                        <field name="hierarchy_level">2</field>
                        <field name="name">122 - Withholding tax on interest paid by credit institutions</field>
                        <field name="code">l10n_ma_vat_122</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_122_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_123" model="account.report.line">
                        <field name="hierarchy_level">2</field>
                        <field name="name">123 - Withholding tax on proceeds from securitization transactions</field>
                        <field name="code">l10n_ma_vat_123</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_123_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_124" model="account.report.line">
                        <field name="hierarchy_level">2</field>
                        <field name="name">124 - Withholding tax on transactions carried out by non-residents for the benefit of Moroccan customers excluded from the scope of application of VAT</field>
                        <field name="code">l10n_ma_vat_124</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_124_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_part_c" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">C - Operations carried out with non-resident taxpayers (Article 115 of the CGI)</field>
                <field name="children_ids">
                    <record id="tax_report_line_129" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">129 - Operations with non-resident taxpayers</field>
                        <field name="code">l10n_ma_vat_129</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_129_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">129B</field>
                            </record>
                            <record id="tax_report_line_129_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">129T</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_130" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">130 - Total VAT Payable</field>
                <field name="code">l10n_ma_vat_130</field>
                <field name="expression_ids">
                    <record id="tax_report_line_130_tax_criterium" model="account.report.expression">
                        <field name="label">tax_criterium</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_b.tax_sum - l10n_ma_vat_190.tax</field>
                    </record>
                    <record id="tax_report_line_130_tax_extra" model="account.report.expression">
                        <field name="label">tax_extra</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_121.tax + l10n_ma_vat_122.tax + l10n_ma_vat_123.tax + l10n_ma_vat_124.tax + l10n_ma_vat_129.tax</field>
                        <field name="subformula">if_other_expr_above(l10n_ma_vat_130.tax_criterium, EUR(0))</field>
                    </record>
                    <record id="tax_report_line_130_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_b.tax_sum + l10n_ma_vat_130.tax_extra</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_part_d" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">D - Breakdown of deductions</field>
                <field name="code">l10n_ma_vat_d</field>
                <field name="expression_ids">
                    <record id="tax_report_part_d_tax_sum" model="account.report.expression">
                        <field name="label">tax_sum</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_d_1.tax_sum + l10n_ma_vat_d_2.tax_sum</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_ma_vat_d_prorata" model="account.report.line">
                        <field name="hierarchy_level">1</field>
                        <field name="name">Percentage of prorated deductions (ADC110)</field>
                        <field name="code">l10n_ma_vat_d_prorata</field>
                        <field name="expression_ids">
                            <record id="l10n_ma_vat_d_prorata_pro" model="account.report.expression">
                                <field name="label">pro</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=0</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_part_d_1" model="account.report.line">
                        <field name="hierarchy_level">1</field>
                        <field name="name">1 - NON-FIXED ASSETS PURCHASES</field>
                        <field name="code">l10n_ma_vat_d_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_part_d_1_tax_sum" model="account.report.expression">
                                <field name="label">tax_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_d_1_1.tax_sum + l10n_ma_vat_d_1_2.tax_sum</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_part_d_1_1" model="account.report.line">
                                <field name="hierarchy_level">2</field>
                                <field name="name">Services</field>
                                <field name="code">l10n_ma_vat_d_1_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_part_d_1_1_tax_sum" model="account.report.expression">
                                        <field name="label">tax_sum</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_140.tax + l10n_ma_vat_141.tax + l10n_ma_vat_142.tax + l10n_ma_vat_143.tax + l10n_ma_vat_144.tax + l10n_ma_vat_153.tax</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="tax_report_line_140" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">140 - Services (20%)</field>
                                        <field name="code">l10n_ma_vat_140</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_140_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">140T</field>
                                            </record>
                                            <record id="tax_report_line_140_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_140.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_141" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">141 - Transportation (14%)</field>
                                        <field name="code">l10n_ma_vat_141</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_141_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">141T</field>
                                            </record>
                                            <record id="tax_report_line_141_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_141.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_142" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">142 - Banking operation (10%)</field>
                                        <field name="code">l10n_ma_vat_142</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_142_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">142T</field>
                                            </record>
                                            <record id="tax_report_line_142_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_142.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_143" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">143 - Hotels for travelers, and all real estate for tourism purposes (10%)</field>
                                        <field name="code">l10n_ma_vat_143</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_143_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">143T</field>
                                            </record>
                                            <record id="tax_report_line_143_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_143.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_144" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">144 - Operations performed by lawyers, interpreters, notaries, adouls, bailiffs and veterinarians (10%)</field>
                                        <field name="code">l10n_ma_vat_144</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_144_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">144T</field>
                                            </record>
                                            <record id="tax_report_line_144_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_144.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_153" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">153 - Other services (10%)</field>
                                        <field name="code">l10n_ma_vat_153</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_153_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">153T</field>
                                            </record>
                                            <record id="tax_report_line_153_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_153.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_part_d_1_2" model="account.report.line">
                                <field name="hierarchy_level">2</field>
                                <field name="name">Other non-fixed assets purchases</field>
                                <field name="code">l10n_ma_vat_d_1_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_part_d_1_2_tax_sum" model="account.report.expression">
                                        <field name="label">tax_sum</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_145.tax + l10n_ma_vat_146.tax + l10n_ma_vat_147.tax + l10n_ma_vat_148.tax + l10n_ma_vat_149.tax + l10n_ma_vat_150.tax + l10n_ma_vat_151.tax + l10n_ma_vat_152.tax + l10n_ma_vat_155.tax + l10n_ma_vat_156.tax + l10n_ma_vat_158.tax</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="tax_report_line_145" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">145 - Import purchases (20%)</field>
                                        <field name="code">l10n_ma_vat_145</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_145_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">145T</field>
                                            </record>
                                            <record id="tax_report_line_145_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_145.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_146" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">146 - Domestic purchases (20%)</field>
                                        <field name="code">l10n_ma_vat_146</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_146_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">146T</field>
                                            </record>
                                            <record id="tax_report_line_146_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_146.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_147" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">147 - Import purchases (14%)</field>
                                        <field name="code">l10n_ma_vat_147</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_147_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">147T</field>
                                            </record>
                                            <record id="tax_report_line_147_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_147.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_148" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">148 - Domestic purchases (14%)</field>
                                        <field name="code">l10n_ma_vat_148</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_148_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">148T</field>
                                            </record>
                                            <record id="tax_report_line_148_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_148.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_149" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">149 - Import purchases (10%)</field>
                                        <field name="code">l10n_ma_vat_149</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_149_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">149T</field>
                                            </record>
                                            <record id="tax_report_line_149_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_149.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_150" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">150 - Domestic purchases (10%)</field>
                                        <field name="code">l10n_ma_vat_150</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_150_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">150T</field>
                                            </record>
                                            <record id="tax_report_line_150_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_150.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_151" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">151 - Import purchases (7%)</field>
                                        <field name="code">l10n_ma_vat_151</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_151_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">151T</field>
                                            </record>
                                            <record id="tax_report_line_151_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_151.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_152" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">152 - Domestic purchases (7%)</field>
                                        <field name="code">l10n_ma_vat_152</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_152_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">152T</field>
                                            </record>
                                            <record id="tax_report_line_152_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_152.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_155" model="account.report.line">
                                        <field name="hierarchy_level">2</field>
                                        <field name="name">155 - Contract work (20%)</field>
                                        <field name="code">l10n_ma_vat_155</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_155_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">155T</field>
                                            </record>
                                            <record id="tax_report_line_155_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_155.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_156" model="account.report.line">
                                        <field name="hierarchy_level">3</field>
                                        <field name="name">156 - Subcontracting (real estate work) (20%)</field>
                                        <field name="code">l10n_ma_vat_156</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_156_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">156T</field>
                                            </record>
                                            <record id="tax_report_line_156_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_156.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_158" model="account.report.line">
                                        <field name="hierarchy_level">2</field>
                                        <field name="name">158 - Undisclosed VAT (Article 125 ter of the CGI)</field>
                                        <field name="code">l10n_ma_vat_158</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_158_base_tax" model="account.report.expression">
                                                <field name="label">base_tax</field>
                                                <field name="engine">external</field>
                                                <field name="formula">most_recent</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                            <record id="tax_report_line_158_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_ma_vat_158.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_part_d_2" model="account.report.line">
                        <field name="hierarchy_level">1</field>
                        <field name="name">2 - FIXED ASSETS</field>
                        <field name="code">l10n_ma_vat_d_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_part_d_2_tax_sum" model="account.report.expression">
                                <field name="label">tax_sum</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_162.tax + l10n_ma_vat_163.tax + l10n_ma_vat_164.tax + l10n_ma_vat_165.tax + l10n_ma_vat_166.tax + l10n_ma_vat_167.tax + l10n_ma_vat_168.tax + l10n_ma_vat_160.tax + l10n_ma_vat_169.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_162" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">162 - Import purchases (20%)</field>
                                <field name="code">l10n_ma_vat_162</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_162_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">162T</field>
                                    </record>
                                    <record id="tax_report_line_162_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_162.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_163" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">163 - Domestic purchases (20%)</field>
                                <field name="code">l10n_ma_vat_163</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_163_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">163T</field>
                                    </record>
                                    <record id="tax_report_line_163_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_163.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_164" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">164 - Self-supply other than constructions (20%)</field>
                                <field name="code">l10n_ma_vat_164</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_164_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">164T</field>
                                    </record>
                                    <record id="tax_report_line_164_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_164.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_165" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">165 - Facilities and installations (20%)</field>
                                <field name="code">l10n_ma_vat_165</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_165_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">165T</field>
                                    </record>
                                    <record id="tax_report_line_165_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_165.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_166" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">166 - Constructions (20%)</field>
                                <field name="code">l10n_ma_vat_166</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_166_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">166T</field>
                                    </record>
                                    <record id="tax_report_line_166_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_166.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_167" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">167 - Self-supply of constructions (20%)</field>
                                <field name="code">l10n_ma_vat_167</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_167_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">167T</field>
                                    </record>
                                    <record id="tax_report_line_167_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_167.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_168" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">168 - Other fixed assets (14%)</field>
                                <field name="code">l10n_ma_vat_168</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_168_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">168T</field>
                                    </record>
                                    <record id="tax_report_line_168_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_168.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_160" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">160 - Other fixed assets (10%)</field>
                                <field name="code">l10n_ma_vat_160</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_160_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">160T</field>
                                    </record>
                                    <record id="tax_report_line_160_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_160.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_169" model="account.report.line">
                                <field name="hierarchy_level">3</field>
                                <field name="name">169 - Other fixed assets (7%)</field>
                                <field name="code">l10n_ma_vat_169</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_169_base_tax" model="account.report.expression">
                                        <field name="label">base_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">169T</field>
                                    </record>
                                    <record id="tax_report_line_169_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_ma_vat_169.base_tax * l10n_ma_vat_d_prorata.pro / 100</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_182" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">182 - Total deductions (lines 140 to 169)</field>
                <field name="code">l10n_ma_vat_182</field>
                <field name="expression_ids">
                    <record id="tax_report_line_182_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_d_1.tax_sum + l10n_ma_vat_d_2.tax_sum</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_part_ded" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">Deduction adjustments</field>
                <field name="children_ids">
                    <record id="tax_report_line_170" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">170 - Credit from previous period</field>
                        <field name="code">l10n_ma_vat_170</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_170_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_170_initial_carryover" model="account.report.expression">
                                <field name="label">initial_carryover</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">170T</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_170_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_170._applied_carryover_tax + l10n_ma_vat_170.initial_carryover</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_180" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">180 - Additional deduction of the pro-rata regularization</field>
                        <field name="code">l10n_ma_vat_180</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_180_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_181" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">181 - Cumulative tax credit fraction</field>
                        <field name="code">l10n_ma_vat_181</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_181_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_185" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">185 - Amount of cancelled credit, remaining chargeable, not reimbursed for investment property</field>
                        <field name="code">l10n_ma_vat_185</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_185_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_186" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">186 - Amount claimed as reimbursement for investment property (Article 103 bis of the CGI)</field>
                        <field name="code">l10n_ma_vat_186</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_186_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_187" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">187 - Amount of reimbursement authorized (Article 103 of the CGI)</field>
                        <field name="code">l10n_ma_vat_187</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_187_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_190" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">190 - Total deductible VAT ((182 to 185) - (186 and 187))</field>
                <field name="code">l10n_ma_vat_190</field>
                <field name="expression_ids">
                    <record id="tax_report_line_190_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ma_vat_182.tax + l10n_ma_vat_170.tax + l10n_ma_vat_180.tax + l10n_ma_vat_181.tax + l10n_ma_vat_185.tax - l10n_ma_vat_186.tax - l10n_ma_vat_187.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_part_results" model="account.report.line">
                <field name="hierarchy_level">0</field>
                <field name="name">VAT Results</field>
                <field name="children_ids">
                    <record id="tax_report_line_200" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">200 - Payable VAT (130 - 190)</field>
                        <field name="code">l10n_ma_vat_200</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_200_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_130.tax - l10n_ma_vat_190.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_201" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">201 - Credit (190 - 130)</field>
                        <field name="code">l10n_ma_vat_201</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_201_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_190.tax - l10n_ma_vat_130.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_202" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">202 - 15% reduction of the credit for the period ((182 + 180) - 130) x 15%</field>
                        <field name="code">l10n_ma_vat_202</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_202_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_203" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">203 - Credit to be carried forward (201 - 202)</field>
                        <field name="code">l10n_ma_vat_203</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_203_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_201.tax - l10n_ma_vat_202.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                            <record id="tax_report_line_203_carryover" model="account.report.expression">
                                <field name="label">_carryover_tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_203.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                                <field name="carryover_target">l10n_ma_vat_170._applied_carryover_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_204" model="account.report.line">
                        <field name="hierarchy_level">3</field>
                        <field name="name">204 - Credit with payment including VAT due under articles 115, 116 and 117 of the CGI</field>
                        <field name="code">l10n_ma_vat_204</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_204_tax_criterium" model="account.report.expression">
                                <field name="label">tax_criterium</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_b.tax_sum - l10n_ma_vat_190.tax</field>
                            </record>
                            <record id="tax_report_line_204_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ma_vat_121.tax + l10n_ma_vat_122.tax + l10n_ma_vat_123.tax + l10n_ma_vat_124.tax + l10n_ma_vat_129.tax</field>
                                <field name="subformula">if_other_expr_below(l10n_ma_vat_204.tax_criterium, EUR(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-ma.csv

```csv
"id","code","name","account_type","reconcile","name@fr"
"pcg_1111","1111","Share Capital","equity","False","Capital social"
"pcg_1112","1112","Endowment Funds","equity","False","Fonds de dotation"
"pcg_11171","11171","Individual Capital","equity","False","Capital individuel"
"pcg_11175","11175","Owner's Account","equity","False","Compte de l'exploitant"
"pcg_1119","1119","Shareholders, Uncalled Subscribed Capital","equity","False","Actionnaires, capital souscrit non appelé"
"pcg_1121","1121","Share Premiums","equity","False","Primes d'émission"
"pcg_1122","1122","Merger Premiums","equity","False","Primes de fusion"
"pcg_1123","1123","Contribution Premiums","equity","False","Primes d'apport"
"pcg_1130","1130","Revaluation Differences","equity","False","Écarts de réévaluation"
"pcg_1140","1140","Legal Reserve","equity","False","Réserve légale"
"pcg_1151","1151","Statutory or Contractual Reserves","equity","False","Réserves statutaires ou contractuelles"
"pcg_1152","1152","Optional Reserves","equity","False","Réserves facultatives"
"pcg_1155","1155","Regulated Reserves","equity","False","Réserves réglementées"
"pcg_1161","1161","Retained Result (Credit Balance)","equity","False","Report à nouveau (Solde créditeur)"
"pcg_1169","1169","Retained Result (Debit Balance)","equity","False","Report à nouveau (Solde débiteur)"
"pcg_1181","1181","Net Results Pending Allocation (Credit Balance)","equity","False","Résultats nets en instance d'affectation (Solde créditeur)"
"pcg_1189","1189","Net Results Pending Allocation (Debit Balance)","equity","False","Résultats nets en instance d'affectation (Solde débiteur)"
"pcg_1191","1191","Net Result for the Fiscal Year (Credit Balance)","equity","False","Résultat net de l'exercice (Solde créditeur)"
"pcg_1199","1199","Net Result for the Fiscal Year (Debit Balance)","equity","False","Résultat net de l'exercice (Solde débiteur)"
"pcg_1311","1311","Investment Grants Received","equity","False","Subventions d'investissement reçues"
"pcg_1319","1319","Investment Grants Recorded in the Income and Expense Account","equity","False","Subventions d'investissement inscrites au compte de produits et charges"
"pcg_1351","1351","Provisions for Accelerated Depreciations","equity","False","Provisions pour amortissements dérogatoires"
"pcg_1352","1352","Provisions for Capital Gains Pending Taxation","equity","False","Provisions pour plus-values en instance d'imposition"
"pcg_1354","1354","Provisions for Investments","equity","False","Provisions pour investissements"
"pcg_1355","1355","Provisions for Reconstitution of Deposits","equity","False","Provisions pour reconstitution des gisements"
"pcg_1356","1356","Provisions for Acquisition and Construction of Housing","equity","False","Provisions pour acquisition et construction de logements"
"pcg_1358","1358","Other Regulated Provisions","equity","False","Autres provisions réglementées"
"pcg_1410","1410","Bond Loans","liability_non_current","False","Emprunts obligataires"
"pcg_1481","1481","Loans from Credit Institutions","liability_non_current","False","Emprunts auprès des établissements de crédit"
"pcg_1482","1482","Advances from the State","liability_non_current","False","Avances de l'État"
"pcg_1483","1483","Liabilities Related to Equity Investments","liability_non_current","False","Dettes rattachées à des participations"
"pcg_1484","1484","Fund Notes","liability_non_current","False","Billets de fonds"
"pcg_1485","1485","Advances Received and Blocked Current Accounts","liability_non_current","False","Avances reçues et comptes courants bloqués"
"pcg_1486","1486","Suppliers of Non-Current Assets","liability_non_current","False","Fournisseurs d'immobilisations"
"pcg_1487","1487","Deposits and Guarantees Received","liability_non_current","False","Dépôts et cautionnements reçus"
"pcg_1488","1488","Miscellaneous Financial Liabilities","liability_non_current","False","Dettes de financement diverses"
"pcg_1511","1511","Provisions for Disputes","liability_non_current","False","Provisions pour litiges"
"pcg_1512","1512","Provisions for Warranties Given to Customers","liability_non_current","False","Provisions pour garanties données aux clients"
"pcg_1513","1513","Provisions for Own Insurer","liability_non_current","False","Provisions pour propre assureur"
"pcg_1514","1514","Provisions for Losses on Futures Markets","liability_non_current","False","Provisions pour pertes sur marchés à terme"
"pcg_1515","1515","Provisions for Fines, Double Duties, Penalties","liability_non_current","False","Provisions pour amendes, doubles droits, pénalités"
"pcg_1516","1516","Provisions for Foreign Exchange Losses","liability_non_current","False","Provisions pour pertes de change"
"pcg_1518","1518","Other Provisions for Risks","liability_non_current","False","Autres provisions pour risques"
"pcg_1551","1551","Provisions for Taxes","liability_non_current","False","Provisions pour impôts"
"pcg_1552","1552","Provisions for Pensions and Similar Obligations","liability_non_current","False","Provisions pour pensions de retraite et obligations similaires"
"pcg_1555","1555","Provisions for Deferred Expenses","liability_non_current","False","Provisions pour charges à répartir sur plusieurs exercices"
"pcg_1558","1558","Other Provisions for Expenses","liability_non_current","False","Autres provisions pour charges"
"pcg_1601","1601","Headquarters Liaison Accounts","liability_current","False","Comptes de liaison du siège"
"pcg_1605","1605","Subsidiaries' Liaison Accounts","asset_current","False","Comptes de liaison des établissements"
"pcg_1710","1710","Increase in Non-Current Receivables","liability_current","False","Augmentation des créances immobilisées"
"pcg_1720","1720","Decrease in Financing Liabilities","liability_current","False","Diminution des dettes de financement"
"pcg_2111","2111","Incorporation Costs","asset_non_current","False","Frais de constitution"
"pcg_2112","2112","Pre-Start-Up Costs","asset_non_current","False","Frais préalables au démarrage"
"pcg_2113","2113","Capital Increase Costs","asset_non_current","False","Frais d'augmentation du capital"
"pcg_2114","2114","Expenses Related to Mergers, Demergers and Conversions","asset_non_current","False","Frais sur opérations de fusions, scissions et transformations"
"pcg_2116","2116","Prospecting Costs","asset_non_current","False","Frais de prospection"
"pcg_2117","2117","Publicity Costs","asset_non_current","False","Frais de publicité"
"pcg_2118","2118","Other Start-Up Costs","asset_non_current","False","Autres frais préliminaires"
"pcg_2121","2121","Acquisition Costs of Non-Current Assets","asset_non_current","False","Frais d'acquisition des immobilisations"
"pcg_2125","2125","Issuance Costs of Loans","asset_non_current","False","Frais d'émission des emprunts"
"pcg_2128","2128","Other Deferred Expenses","asset_non_current","False","Autres charges à répartir"
"pcg_2130","2130","Bond Redemption Premiums","asset_non_current","False","Primes de remboursement des obligations"
"pcg_2210","2210","Non-Current Assets in Research and Development","asset_non_current","False","Immobilisation en recherche et développement"
"pcg_2220","2220","Patents, Trademarks, Rights and Similar Assets","asset_non_current","False","Brevets, marques, droits et valeurs similaires"
"pcg_2230","2230","Commercial Funds","asset_non_current","False","Fonds commercial"
"pcg_2285","2285","Other Intangible Non-Current Assets","asset_non_current","False","Autres immobilisations incorporelles"
"pcg_2311","2311","Bare Land","asset_fixed","False","Terrains nus"
"pcg_2312","2312","Developed Land","asset_fixed","False","Terrains aménagés"
"pcg_2313","2313","Built-Up Land","asset_fixed","False","Terrains bâtis"
"pcg_2314","2314","Deposit Land","asset_fixed","False","Terrains de gisement"
"pcg_2316","2316","Layout and Development of Land","asset_fixed","False","Agencements et aménagements de terrains"
"pcg_2318","2318","Other Land","asset_fixed","False","Autres terrains"
"pcg_23211","23211","Industrial Buildings (A, B ...)","asset_fixed","False","Bâtiments industriels (A, B ...)"
"pcg_23214","23214","Administrative and Commercial Buildings (A, B ...)","asset_fixed","False","Bâtiments administratifs et commerciaux (A, B ...)"
"pcg_23218","23218","Other Buildings","asset_fixed","False","Autres bâtiments"
"pcg_2323","2323","Constructions on Third Party Land","asset_fixed","False","Constructions sur terrains d'autrui"
"pcg_2325","2325","Infrastructure Works","asset_fixed","False","Ouvrages d'infrastructure"
"pcg_2327","2327","Layout and Development of Constructions","asset_fixed","False","Agencements et aménagements des constructions"
"pcg_2328","2328","Other Constructions","asset_fixed","False","Autres constructions"
"pcg_2331","2331","Technical Installations","asset_fixed","False","Installations techniques"
"pcg_23321","23321","Equipment","asset_fixed","False","Matériel"
"pcg_23324","23324","Tools","asset_fixed","False","Outillage"
"pcg_2333","2333","Identifiable Recoverable Packaging","asset_fixed","False","Emballages récupérables identifiables"
"pcg_2338","2338","Other Technical Installations, Equipment and Tools","asset_fixed","False","Autres installations techniques, matériel et outillage"
"pcg_2340","2340","Transport Equipment","asset_fixed","False","Matériel de transport"
"pcg_2351","2351","Office Furniture","asset_fixed","False","Mobilier de bureau"
"pcg_2352","2352","Office Equipment","asset_fixed","False","Matériel de bureau"
"pcg_2355","2355","Computer Equipment","asset_fixed","False","Matériel informatique"
"pcg_2356","2356","Miscellaneous Layout, Installation and Development (Non-Business Assets)","asset_fixed","False","Agencements, installations et aménagements divers (biens n'appartenant pas à l'entreprise)"
"pcg_2358","2358","Other Furniture, Office Equipment, Miscellaneous Amenities","asset_fixed","False","Autres mobilier, matériel de bureau et aménagements divers"
"pcg_2380","2380","Other Tangible Non-Current Assets","asset_fixed","False","Autres immobilisations corporelles"
"pcg_2392","2392","Land and Constructions in Progress","asset_fixed","False","Immobilisations corporelles en cours des terrains et constructions"
"pcg_2393","2393","Technical Installations, Equipment and Tools in Progress","asset_fixed","False","Immobilisations corporelles en cours des installations techniques, matériel et outillage"
"pcg_2394","2394","Transport Equipment in Progress","asset_fixed","False","Immobilisations corporelles en cours de matériel de transport"
"pcg_2395","2395","Furniture, Office Equipment, Miscellaneous Amenities in Progress","asset_fixed","False","Immobilisations corporelles en cours de mobilier, matériel de bureau et aménagements divers"
"pcg_2397","2397","Advances and Deposits Paid on Orders for Tangible Non-Current Assets","asset_prepayments","False","Avances et acomptes versés sur commandes d'immobilisations corporelles"
"pcg_2398","2398","Other Tangible Non-Current Assets in Progress","asset_fixed","False","Autres immobilisations corporelles en cours"
"pcg_2411","2411","Loans to Personnel","asset_non_current","False","Prêts au personnel"
"pcg_2415","2415","Loans to Associates","asset_non_current","False","Prêts aux associés"
"pcg_2416","2416","Fund Notes","asset_non_current","False","Billets de fonds"
"pcg_2418","2418","Other Loans","asset_non_current","False","Autres prêts"
"pcg_2481","2481","Long-Term Securities (Debt Rights)","asset_non_current","False","Titres immobilisés (Droits de créance)"
"pcg_24811","24811","Bonds (Obligations)","asset_non_current","False","Obligations"
"pcg_24813","24813","Equipment Bonds","asset_non_current","False","Bons d'équipement"
"pcg_24814","24814","Miscellaneous Bonds","asset_non_current","False","Bons divers"
"pcg_2483","2483","Receivables Related to Equity Investments","asset_non_current","False","Créances rattachées à des participations"
"pcg_24861","24861","Deposits","asset_prepayments","False","Dépôts"
"pcg_24864","24864","Surety Bonds","asset_non_current","False","Cautionnements"
"pcg_2487","2487","Non-Current Receivables","asset_non_current","False","Créances immobilisées"
"pcg_2488","2488","Miscellaneous Financial Receivables","asset_non_current","False","Créances financières diverses"
"pcg_2510","2510","Equity Securities","asset_non_current","False","Titres de participation"
"pcg_2581","2581","Shares","asset_non_current","False","Actions"
"pcg_2588","2588","Miscellaneous Securities","asset_non_current","False","Titres divers"
"pcg_2710","2710","Decrease in Non-Current Assets","asset_non_current","False","Diminution des créances immobilisées"
"pcg_2720","2720","Increase in Financing Liabilities","asset_non_current","False","Augmentation des dettes de financement"
"pcg_28111","28111","Depreciations of Incorporation Costs","asset_non_current","False","Amortissements des frais de constitution"
"pcg_28112","28112","Depreciations of Pre-Start-Up Costs","asset_non_current","False","Amortissements des frais préliminaires au démarrage"
"pcg_28113","28113","Depreciations of Capital Increase Costs","asset_non_current","False","Amortissements des frais d'augmentation du capital"
"pcg_28114","28114","Depreciations of Expenses Related to Mergers, Demergers and Conversions","asset_non_current","False","Amortissements des frais sur opérations de fusions, scissions et transformations"
"pcg_28116","28116","Depreciations of Prospecting Costs","asset_non_current","False","Amortissements des frais de prospection"
"pcg_28117","28117","Depreciations of Publicity Costs","asset_non_current","False","Amortissements des frais de publicité"
"pcg_28118","28118","Depreciations of Other Start-Up Costs","asset_non_current","False","Amortissements des autres frais préliminaires"
"pcg_28121","28121","Depreciations of Acquisition Costs of Non-Current Assets","asset_non_current","False","Amortissements des frais d'acquisition des immobilisations"
"pcg_28125","28125","Depreciations of Issuance Costs of Loans","asset_non_current","False","Amortissements des frais d'émission des emprunts"
"pcg_28128","28128","Depreciations of Other Deferred Expenses","asset_non_current","False","Amortissements des autres charges à répartir"
"pcg_2813","2813","Depreciations of Bond Redemption Premiums","asset_non_current","False","Amortissements des primes de remboursements des obligations"
"pcg_2821","2821","Depreciations of Non-Current Assets in Research and Development","asset_non_current","False","Amortissements de l'immobilisation en recherche et développement"
"pcg_2822","2822","Depreciations of Patents, Trademarks, Rights and Similar Assets","asset_non_current","False","Amortissements des brevets, marques, droits et valeurs similaires"
"pcg_2823","2823","Depreciations of Commercial Funds","asset_non_current","False","Amortissement du fond commercial"
"pcg_2828","2828","Depreciations of Other Intangible Non-Current Assets","asset_non_current","False","Amortissements des autres immobilisations incorporelles"
"pcg_28311","28311","Depreciations of Bare Land","asset_fixed","False","Amortissements des terrains nus"
"pcg_28312","28312","Depreciations of Developed Land","asset_fixed","False","Amortissements des terrains aménagés"
"pcg_28313","28313","Depreciations of Built-Up Land","asset_fixed","False","Amortissements des terrains bâtis"
"pcg_28314","28314","Depreciations of Deposit Land","asset_fixed","False","Amortissements des terrains de gisement"
"pcg_28316","28316","Depreciations of Layout and Development of Land","asset_fixed","False","Amortissements des agencements et aménagements de terrains"
"pcg_28318","28318","Depreciations of Other Land","asset_fixed","False","Amortissements des autres terrains"
"pcg_28321","28321","Depreciations of Buildings","asset_fixed","False","Amortissements des bâtiments"
"pcg_28323","28323","Depreciations of Constructions on Third Party Land","asset_fixed","False","Amortissements des constructions sur terrains d'autrui"
"pcg_28325","28325","Depreciations of Infrastructure Works","asset_fixed","False","Amortissements des ouvrages d'infrastructure"
"pcg_28327","28327","Depreciations of Layout and Development of Constructions","asset_fixed","False","Amortissements des installations, agencements et aménagements des constructions"
"pcg_28328","28328","Depreciations of Other Constructions","asset_fixed","False","Amortissements des autres constructions"
"pcg_28331","28331","Depreciations of Technical Installations","asset_fixed","False","Amortissements des installations techniques"
"pcg_28332","28332","Depreciations of Equipment and Tools","asset_fixed","False","Amortissements du matériel et outillage"
"pcg_28333","28333","Depreciations of Identifiable Recoverable Packaging","asset_fixed","False","Amortissements des emballages récupérables identifiables"
"pcg_28338","28338","Depreciations of Other Technical Installations, Equipment and Tools","asset_fixed","False","Amortissements des autres installations techniques, matériel et outillage"
"pcg_2834","2834","Depreciations of Transport Equipment","asset_fixed","False","Amortissements du matériel de transport"
"pcg_28351","28351","Depreciations of Office Furniture","asset_fixed","False","Amortissements du mobilier de bureau"
"pcg_28352","28352","Depreciations of Office Equipment","asset_fixed","False","Amortissements du matériel de bureau"
"pcg_28355","28355","Depreciations of Computer Equipment","asset_fixed","False","Amortissements du matériel informatique"
"pcg_28356","28356","Depreciations of Miscellaneous Layout, Installation and Development","asset_fixed","False","Amortissements des agencements, installations et aménagements divers"
"pcg_28358","28358","Depreciations of Other Furniture, Office Equipment, Miscellaneous Amenities","asset_fixed","False","Amortissements des autres mobilier, matériel de bureau et aménagements divers"
"pcg_2838","2838","Depreciations of Other Tangible Non-Current Assets","asset_fixed","False","Amortissements des autres immobilisations corporelles"
"pcg_2920","2920","Provisions for Depreciation of Intangible Non-Current Assets","asset_non_current","False","Provisions pour dépréciation des immobilisations incorporelles"
"pcg_2930","2930","Provisions for Depreciation of Tangible Non-Current Assets","asset_fixed","False","Provisions pour dépréciation des immobilisations corporelles"
"pcg_2941","2941","Provisions for Depreciation of Non-Current Loans","asset_non_current","False","Provisions pour dépréciation des prêts immobilisés"
"pcg_2948","2948","Provisions for Depreciation of Other Financial Receivables","asset_non_current","False","Provisions pour dépréciation des autres créances financières"
"pcg_2951","2951","Provisions for Depreciation of Equity Securities","asset_non_current","False","Provisions pour dépréciation des titres de participation"
"pcg_2958","2958","Provisions for Depreciation of Other Long-Term Securities","asset_non_current","False","Provisions pour dépréciation des autres titres immobilisés"
"pcg_3111","3111","Goods (Group A)","asset_current","False","Marchandises (Groupe A)"
"pcg_3112","3112","Goods (Group B)","asset_current","False","Marchandises (Groupe B)"
"pcg_3116","3116","Goods in Transit","asset_current","False","Marchandises en cours de route"
"pcg_3118","3118","Other Goods","asset_current","False","Autres marchandises"
"pcg_31211","31211","Raw Materials (Group A)","asset_current","False","Matières premières (Groupe A)"
"pcg_31212","31212","Raw Materials (Group B)","asset_current","False","Matières premières (Groupe B)"
"pcg_31221","31221","Consumable Materials (Group A)","asset_current","False","Matières consommables (Groupe A)"
"pcg_31222","31222","Consumable Materials (Group B)","asset_current","False","Matières consommables (Groupe B)"
"pcg_31223","31223","Fuels","asset_current","False","Combustibles"
"pcg_31224","31224","Maintenance Products","asset_current","False","Produits d'entretien"
"pcg_31225","31225","Workshop and Factory Supplies","asset_current","False","Fournitures d'atelier et d'usine"
"pcg_31226","31226","Store Supplies","asset_current","False","Fournitures de magasin"
"pcg_31227","31227","Office Supplies","asset_current","False","Fournitures de bureau"
"pcg_31231","31231","Lost Packaging","asset_current","False","Emballages perdus"
"pcg_31232","31232","Unidentifiable Recoverable Packaging","asset_current","False","Emballages récupérables non identifiables"
"pcg_31233","31233","Mixed-Use Packaging","asset_current","False","Emballages à usage mixte"
"pcg_3126","3126","Consumable Materials and Supplies in Transit","asset_current","False","Matières et fournitures consommables en cours de route"
"pcg_3128","3128","Other Consumable Materials and Supplies","asset_current","False","Autres matières et fournitures consommables"
"pcg_31311","31311","Produced Goods in Progress","asset_current","False","Biens produits en cours"
"pcg_31312","31312","Intermediary Goods in Progress","asset_current","False","Biens intermédiaires en cours"
"pcg_31317","31317","Residual Goods in Progress","asset_current","False","Biens résiduels en cours"
"pcg_31341","31341","Work in Progress","asset_current","False","Travaux en cours"
"pcg_31342","31342","Studies in Progress","asset_current","False","Études en cours"
"pcg_31343","31343","Provided Services in Progress","asset_current","False","Prestations en cours"
"pcg_3138","3138","Other Products in Progress","asset_current","False","Autres produits en cours"
"pcg_31411","31411","Intermediary Products (Group A)","asset_current","False","Produits intermédiaires (Groupe A)"
"pcg_31412","31412","Intermediary Products (Group B)","asset_current","False","Produits intermédiaires (Groupe B)"
"pcg_31451","31451","Waste","asset_current","False","Déchets"
"pcg_31452","31452","Scrap","asset_current","False","Rebuts"
"pcg_31453","31453","Recovered Materials","asset_current","False","Matières de récupération"
"pcg_3148","3148","Other Intermediary and Residual Products","asset_current","False","Autres produits intermédiaires et produits résiduels"
"pcg_3151","3151","Finished Products (Group A)","asset_current","False","Produits finis (Groupe A)"
"pcg_3152","3152","Finished Products (Group B)","asset_current","False","Produits finis (Groupe B)"
"pcg_3156","3156","Finished Products in Progress","asset_current","False","Produits finis en cours de route"
"pcg_3158","3158","Other Finished Products","asset_current","False","Autres produits finis"
"pcg_3411","3411","Suppliers - Advances Paid on Operating Orders","asset_current","True","Fournisseurs - Avances acomptes versés sur commandes d'exploit"
"pcg_3413","3413","Suppliers - Receivables for Packaging and Materials to be Returned","asset_current","True","Fournisseurs - Créances pour emballages et matériel à rendre"
"pcg_3417","3417","Discounts, Rebates and Refunds to be Obtained - Credits Not Yet Received","asset_current","True","Rabais, remises et ristournes à obtenir - Avoirs non encore reçus"
"pcg_3418","3418","Other Supplier Receivables","asset_current","True","Autres fournisseurs débiteurs"
"pcg_34211","34211","Customers - Category A","asset_receivable","True","Clients - Catégorie A"
"pcg_34212","34212","Customers - Category B","asset_receivable","True","Clients - Catégorie B"
"pcg_34218","34218","Customers (POS)","asset_receivable","True","Clients (POS)"
"pcg_3423","3423","Customers - Holdbacks","asset_receivable","True","Clients - Retenues de garantie"
"pcg_3424","3424","Doubtful or Contentious Customers","asset_receivable","True","Clients douteux ou litigieux"
"pcg_3425","3425","Customers - Notes Receivable","asset_receivable","True","Clients - Effets à recevoir"
"pcg_34271","34271","Customers - Invoices to be Issued","asset_receivable","True","Clients - Factures à établir"
"pcg_34272","34272","Receivables on Work Not Yet Billable","asset_receivable","True","Créances sur travaux non encore facturables"
"pcg_3428","3428","Other Customers and Related Accounts","asset_receivable","True","Autres clients et comptes rattachés"
"pcg_3431","3431","Advances and Deposits to Personnel","asset_prepayments","False","Avances et acomptes au personnel"
"pcg_3438","3438","Personnel - Other Receivables","asset_current","False","Personnel - Autres débiteurs"
"pcg_34511","34511","Investment Grants Receivable","asset_current","False","Subventions d'investissement à recevoir"
"pcg_34512","34512","Operating Grants Receivable","asset_current","False","Subventions d'exploitation à recevoir"
"pcg_34513","34513","Balancing Grants Receivable","asset_current","False","Subventions d'équilibre à recevoir"
"pcg_3453","3453","Prepayments of Income Taxes","asset_current","False","Acomptes sur impôts sur les résultats"
"pcg_34551","34551","Government - Recoverable VAT on Non-Current Assets","asset_current","False","État - TVA récupérable sur les immobilisations"
"pcg_34552","34552","Government - Recoverable VAT on Expenses","asset_current","False","État - TVA récupérable sur les charges"
"pcg_34553","34553","Government - Recoverable VAT on Non-Current Assets (Transient)","asset_current","False","État - TVA récupérable sur les immobilisations (Transitoire)"
"pcg_34554","34554","Government - Recoverable VAT on Expenses (Transient)","asset_current","False","État - TVA récupérable sur les charges (Transitoire)"
"pcg_3456","3456","Government - VAT Credit (According to Declarations)","asset_current","False","État - Crédit de TVA (suivant déclarations)"
"pcg_3458","3458","Government - Other Receivables","asset_current","False","État - Autres comptes débiteurs"
"pcg_3461","3461","Partners - Partnership Contribution Accounts","asset_current","False","Associés - Comptes d'apport en société"
"pcg_3462","3462","Shareholders - Unpaid Subscribed and Called Capital","asset_current","False","Actionnaires - Capital souscrit et appelé non versé"
"pcg_3463","3463","Partners' Current Accounts in Debit","asset_current","False","Comptes courants des associés débiteurs"
"pcg_3464","3464","Partners - Joint Operations","asset_current","False","Associés - Opérations faites en commun"
"pcg_3467","3467","Receivables Related to Partners' Accounts","asset_current","False","Créances rattachées aux comptes d'associés"
"pcg_3468","3468","Other Partner Accounts Receivable","asset_current","False","Autres comptes d'associés débiteurs"
"pcg_3481","3481","Receivables on Disposal of Non-Current Assets","asset_current","False","Créances sur cessions d'immobilisations"
"pcg_3482","3482","Receivables on Disposal of Current Assets","asset_current","False","Créances sur cessions d'éléments d'actif circulant"
"pcg_3487","3487","Receivables Related to Other Debtors","asset_current","False","Créances rattachées aux autres débiteurs"
"pcg_3488","3488","Various Debtors","asset_current","False","Divers débiteurs"
"pcg_3491","3491","Prepaid Expenses","asset_current","False","Charges constatées d'avances"
"pcg_3493","3493","Accrued and Unpaid Interest Receivable","asset_current","False","Intérêts courus et non échus à percevoir"
"pcg_3495","3495","Periodic Expense Allocation Accounts","asset_current","False","Comptes de répartition périodique des charges"
"pcg_3497","3497","Suspense Accounts - Receivable","asset_current","False","Comptes transitoires ou d'attente - Débiteurs"
"pcg_3501","3501","Shares, Paid-Up Portion","asset_non_current","False","Actions, partie libérée"
"pcg_3502","3502","Shares, Unpaid Portion","asset_non_current","False","Actions, partie non libérée"
"pcg_3504","3504","Bonds","asset_non_current","False","Obligations"
"pcg_35061","35061","Savings Bonds","asset_non_current","False","Bons de caisse"
"pcg_35062","35062","Treasury Bonds","asset_non_current","False","Bons du Trésor"
"pcg_3508","3508","Other Securities and Similar Investments","asset_non_current","False","Autres titres et valeurs de placement similaires"
"pcg_3701","3701","Decrease in Current Receivables (FX)","asset_current","False","Diminution des créances circulantes (EC)"
"pcg_3702","3702","Increase in Current Liabilities (FX)","asset_current","False","Augmentation des dettes circulantes (EC)"
"pcg_3911","3911","Provisions for Depreciation of Goods","asset_current","False","Provisions pour dépréciation des marchandises"
"pcg_3912","3912","Provisions for Depreciation of Materials and Supplies","asset_current","False","Provisions pour dépréciation des matières et fournitures"
"pcg_3913","3913","Provisions for Depreciation of Products in Progress","asset_current","False","Provisions pour dépréciation des produits en cours"
"pcg_3914","3914","Provisions for Depreciation of Intermediary Products","asset_current","False","Provisions pour dépréciation des produits intermédiaires"
"pcg_3915","3915","Provisions for Depreciation of Finished Products","asset_current","False","Provisions pour dépréciation des produits finis"
"pcg_3941","3941","Provisions for Depreciation of Supplier Receivables, Advances and Down Payments","asset_current","False","Provisions pour dépréciation des fournisseurs débiteurs, avances et acomptes"
"pcg_3942","3942","Provisions for Depreciation of Customers and Related Accounts","asset_current","False","Provisions pour dépréciation des clients et comptes rattachés"
"pcg_3943","3943","Provisions for Depreciation of Personnel - Receivables","asset_current","False","Provisions pour dépréciation du personnel - Débiteur"
"pcg_3946","3946","Provisions for Depreciation of Partner Accounts - Receivables","asset_current","False","Provisions pour dépréciation des comptes d'associés débiteurs"
"pcg_3948","3948","Provisions for Depreciation of Other Receivables","asset_current","False","Provisions pour dépréciation des autres débiteurs"
"pcg_3950","3950","Provisions for Depreciation of Securities and Investment Assets","asset_current","False","Provisions pour dépréciation des titres et valeurs de placement"
"pcg_44111","44111","Suppliers - Category A","liability_payable","True","Fournisseurs - Catégorie A"
"pcg_44112","44112","Suppliers - Category B","liability_payable","True","Fournisseurs - Catégorie B"
"pcg_4413","4413","Suppliers - Holdbacks","liability_payable","True","Fournisseurs - Retenues de garantie"
"pcg_4415","4415","Suppliers - Notes Payable","liability_payable","True","Fournisseurs - Effets à payer"
"pcg_4417","4417","Suppliers - Invoices Not Received","liability_payable","True","Fournisseurs - Factures non parvenues"
"pcg_4418","4418","Other Suppliers and Related Accounts","liability_payable","True","Autres fournisseurs et comptes rattachés"
"pcg_4421","4421","Customers - Advances and Deposits Received on Orders in Progress","liability_payable","True","Clients - Avances et acomptes reçus sur commandes en cours"
"pcg_4425","4425","Customers - Payables for Returnable Packaging and Materials","liability_payable","True","Clients - Dettes pour emballages et matériel consignés"
"pcg_4427","4427","Discounts, Rebates and Discounts to be Granted - Credit Notes to be Issued","liability_payable","True","Rabais, remises et ristournes à accorder - Avoirs à établir"
"pcg_4428","4428","Other Customer Payables","liability_payable","True","Autres clients créditeurs"
"pcg_4432","4432","Remuneration Due to Personnel","liability_current","False","Rémunérations dues au personnel"
"pcg_4433","4433","Personnel Deposits in Credit","liability_current","False","Dépôts du personnel créditeurs"
"pcg_4434","4434","Wage Objections","liability_current","False","Oppositions sur salaires"
"pcg_4437","4437","Accrued Personnel Expenses","liability_current","False","Charges du personnel à payer"
"pcg_4438","4438","Personnel - Other Payables","liability_current","False","Personnel - Autres créditeurs"
"pcg_4441","4441","National Social Security Fund","liability_current","False","Caisse Nationale de Sécurité Sociale"
"pcg_4443","4443","Pension Funds","liability_current","False","Caisses de retraite"
"pcg_4445","4445","Mutual Insurances","liability_current","False","Mutuelles"
"pcg_4447","4447","Social Charges to be Paid","liability_current","False","Charges sociales à payer"
"pcg_4448","4448","Other Social Organizations","liability_current","False","Autres organismes sociaux"
"pcg_44521","44521","Government - City Tax and Municipal Tax","liability_current","False","État - Taxe urbaine et taxe d'édilité"
"pcg_44522","44522","Government - Patent","liability_current","False","État - Patente"
"pcg_44525","44525","Government - General Income Tax (IGR)","liability_current","False","État - IGR"
"pcg_4453","4453","Government - Income Taxes","liability_current","False","État - Impôts sur les résultats"
"pcg_4455","4455","Government - VAT Invoiced","liability_current","False","État - TVA facturée"
"pcg_44551","44551","Government - VAT Invoiced (Transient)","liability_current","False","État - TVA facturée(Transitoire)"
"pcg_4456","4456","Government - VAT Due (According to Declarations)","liability_current","False","État - TVA due (suivant déclarations)"
"pcg_4457","4457","Government - Taxes Payable","liability_current","False","État - Impôts et taxes à payer"
"pcg_4458","4458","Government - Other Accounts Payable","liability_current","False","État - Autres comptes créditeurs"
"pcg_4461","4461","Partners - Capital to be Repaid","liability_current","False","Associés - Capital à rembourser"
"pcg_4462","4462","Partners - Payments Received on Capital Increase","liability_current","False","Associés - Versements reçus sur augmentation de capital"
"pcg_4463","4463","Partners' Current Accounts in Credit","liability_current","False","Comptes courants des associés créditeurs"
"pcg_4464","4464","Partners - Joint Operations","liability_current","False","Associés - Opérations faites en commun"
"pcg_4465","4465","Partners - Dividends Payable","liability_current","False","Associés - Dividendes à payer"
"pcg_4468","4468","Other Partner Accounts - Payables","liability_current","False","Autres comptes d'associés - Créditeurs"
"pcg_4481","4481","Payables on Acquisition of Non-Current Assets","liability_current","False","Dettes sur acquisition d'immobilisations"
"pcg_4483","4483","Payables on Purchases of Securities and Investment Assets","liability_current","False","Dettes sur acquisitions de titres et valeurs de placement"
"pcg_4484","4484","Matured Bonds to be Redeemed","liability_current","False","Obligations échues à rembourser"
"pcg_4485","4485","Bonds, Coupons Payable","liability_current","False","Obligations, coupons à payer"
"pcg_4487","4487","Payables to Other Creditors","liability_current","False","Dettes rattachées aux autres créanciers"
"pcg_4488","4488","Various Creditors","liability_current","False","Divers créanciers"
"pcg_4491","4491","Deferred Revenues","liability_current","False","Produits constatés d'avance"
"pcg_4493","4493","Accrued and Unpaid Interest Payable","liability_current","False","Intérêts courus et non échus à payer"
"pcg_4495","4495","Periodic Revenue Allocation Accounts","liability_current","False","Comptes de répartition périodique des produits"
"pcg_4497","4497","Suspense Accounts - Payable","liability_current","False","Comptes transitoires ou d'attente - Créditeurs"
"pcg_4501","4501","Provisions for Disputes","liability_current","False","Provisions pour litiges"
"pcg_4502","4502","Provisions for Warranties Given to Customers","liability_current","False","Provisions pour garanties données aux clients"
"pcg_4505","4505","Provisions for Fines, Double Duties, Penalties","liability_current","False","Provisions pour amendes, doubles droits et pénalités"
"pcg_4506","4506","Provisions for Foreign Exchange Losses","liability_current","False","Provisions pour pertes de change"
"pcg_4507","4507","Provisions for Taxes","liability_current","False","Provisions pour impôts"
"pcg_4508","4508","Other Provisions for Risks and Expenses","liability_current","False","Autres provisions pour risques et charges"
"pcg_4701","4701","Increase in Current Receivables (FX)","liability_current","False","Augmentation des créances circulantes (EC)"
"pcg_4702","4702","Decrease in Current Liabilities (FX)","liability_current","False","Diminution des dettes circulantes (EC)"
"pcg_51111","51111","Checks in Portfolio","asset_current","False","Chèques en portefeuille"
"pcg_51112","51112","Checks Being Cashed","asset_current","False","Chèques à l'encaissement"
"pcg_51131","51131","Notes to be Cashed","asset_current","False","Effets échus à encaisser"
"pcg_51132","51132","Notes Being Cashed","asset_current","False","Effets à l'encaissement"
"pcg_5115","5115","Transfer of Funds","asset_current","True","Virements de fonds"
"pcg_5118","5118","Other Securities to be Cashed","asset_current","False","Autres valeurs à encaisser"
"pcg_5141","5141","Banks (Debit Balance)","asset_cash","False","Banques (solde débiteurs)"
"pcg_5143","5143","General Treasury","asset_current","False","Trésorerie Générale"
"pcg_5146","5146","Postal Checks","asset_current","False","Chèques Postaux"
"pcg_5148","5148","Other Financial and Similar Institutions (Debit Balances)","asset_current","False","Autres établissements financiers et assimilés (Soldes débiteurs)"
"pcg_51611","51611","Central Cash","asset_cash","False","Caisses Centrale"
"pcg_51613","51613","Cash (Branch or Agency A)","asset_cash","False","Caisses (Succursale ou agence A)"
"pcg_51614","51614","Cash (Branch or Agency B)","asset_cash","False","Caisses (Succursale ou agence B)"
"pcg_5165","5165","Imprest Accounts and Letters of Credit","asset_current","False","Régies d'avances et accréditifs"
"pcg_5520","5520","Discount Credits","liability_current","False","Crédits d'escompte"
"pcg_5530","5530","Cash Credits","liability_current","False","Crédits de trésorerie"
"pcg_5541","5541","Banks (Credit Balances)","liability_current","False","Banques (Soldes créditeurs)"
"pcg_5548","5548","Other Financial and Similar Institutions (Credit Balances)","liability_current","False","Autres établissements financiers et assimilés (Soldes créditeurs)"
"pcg_5900","5900","Provisions for Depreciation of Cash Accounts","liability_current","False","Provisions pour dépréciation des comptes de trésorerie"
"pcg_6111","6111","Purchases of Goods (Group A)","expense","False","Achats de marchandises (Groupe A)"
"pcg_6112","6112","Purchases of Goods (Group B)","expense","False","Achats de marchandises (Groupe B)"
"pcg_6114","6114","Change in Inventory of Goods","expense","False","Variation de stocks de marchandises"
"pcg_6118","6118","Purchases of Goods Resold from Previous Years","expense","False","Achats revendus de marchandises des exercices antérieurs"
"pcg_6119","6119","Discounts, Rebates and Refunds Obtained on Purchases of Goods","expense","False","Rabais, remises et ristournes obtenus sur achats de marchandises"
"pcg_61211","61211","Purchases of Raw Materials A","expense","False","Achats de matières premières A"
"pcg_61212","61212","Purchases of Raw Materials B","expense","False","Achats de matières premières B"
"pcg_61221","61221","Purchases of Materials and Supplies A","expense","False","Achats de matières et fournitures A"
"pcg_61222","61222","Purchases of Materials and Supplies B","expense","False","Achats de matières et fournitures B"
"pcg_61223","61223","Purchases of Fuels","expense","False","Achats de combustibles"
"pcg_61224","61224","Purchases of Maintenance Products","expense","False","Achats de produits d'entretien"
"pcg_61225","61225","Purchases of Workshop and Factory Supplies","expense","False","Achats de fournitures d'atelier et d'usine"
"pcg_61226","61226","Purchases of Store Supplies","expense","False","Achats de fournitures de magasin"
"pcg_61227","61227","Purchases of Office Supplies","expense","False","Achats de fournitures de bureau"
"pcg_61231","61231","Purchases of Lost Packaging","expense","False","Achats d'emballages perdus"
"pcg_61232","61232","Purchases of Unidentifiable Recoverable Packaging","expense","False","Achats d'emballages récupérables non identifiables"
"pcg_61233","61233","Purchases of Mixed-Use Packaging","expense","False","Achats d'emballages à usage mixte"
"pcg_61241","61241","Change in Inventories of Raw Materials","expense","False","Variation des stocks des matières premières"
"pcg_61242","61242","Change in Inventories of Consumable Materials and Supplies","expense","False","Variation des stocks de matières et fournitures consommables"
"pcg_61243","61243","Change in Inventories of Packaging","expense","False","Variation des stocks des emballages"
"pcg_61251","61251","Purchases of Non-Storable Supplies (Water, Electricity ...)","expense","False","Achats de fournitures non stockables (Eau, électricité ...)"
"pcg_61252","61252","Purchases of Maintenance Supplies","expense","False","Achats de fournitures d'entretien"
"pcg_61253","61253","Purchase of Small Tools and Equipment","expense","False","Achats de petit outillage et petit équipement"
"pcg_61254","61254","Purchases of Office Supplies","expense","False","Achats de fournitures de bureau"
"pcg_61261","61261","Purchases of Works","expense","False","Achats des travaux"
"pcg_61262","61262","Purchases of Studies","expense","False","Achats des études"
"pcg_61263","61263","Purchases of Services Provided","expense","False","Achats des prestations de service"
"pcg_6128","6128","Purchases of Materials and Supplies from Previous Years","expense","False","Achats de matières et de fournitures des exercices antérieurs"
"pcg_61291","61291","Discounts, Rebates and Refunds Obtained on Purchases of Raw Materials","expense","False","Rabais, remises et ristournes obtenus sur achats de matières premières"
"pcg_61292","61292","Discounts, Rebates and Refunds Obtained on Purchases of Consumable Materials and Supplies","expense","False","Rabais, remises et ristournes obtenus sur achats de matières et fournitures consommables"
"pcg_61293","61293","Discounts, Rebates and Refunds Obtained on Purchases of Packaging","expense","False","Rabais, remises et ristournes obtenus sur achats des emballages"
"pcg_61295","61295","Discounts, Rebates and Refunds Obtained on Non-Stock Purchases","expense","False","Rabais, remises et ristournes obtenus surs achats non stockés"
"pcg_61296","61296","Discounts, Rebates and Refunds Obtained on Purchases of Works, Studies and Services Provided","expense","False","Rabais, remises et ristournes obtenus sur achats de travaux, études et prestations de service"
"pcg_61298","61298","Discounts, Rebates and Refunds Obtained on Purchases of Materials and Supplies from Previous Years","expense","False","Rabais, remises et ristournes obtenus sur achats de matières et fournitures des exercices antérieurs"
"pcg_61311","61311","Rental of Land","expense","False","Location de terrains"
"pcg_61312","61312","Rental of Buildings","expense","False","Location de constructions"
"pcg_61313","61313","Rental of Equipment and Tools","expense","False","Location de matériel et d'outillage"
"pcg_61314","61314","Rental of Office Furniture and Equipment","expense","False","Location de mobilier et matériel de bureau"
"pcg_61315","61315","Rental of Computer Equipment","expense","False","Location de matériel informatique"
"pcg_61316","61316","Rental of Transport Equipment","expense","False","Location de matériel de transport"
"pcg_61317","61317","Damages on Returned Packaging","expense","False","Malis sur emballages rendus"
"pcg_61318","61318","Miscellaneous Rentals and Rental Charges","expense","False","Locations et charges locatives divers"
"pcg_61321","61321","Lease Payments - Furniture and Equipment","expense","False","Redevances de crédit-bail - mobilier et matériel"
"pcg_61331","61331","Maintenance and Repairs of Real Estate","expense","False","Entretien et réparations des biens immobiliers"
"pcg_61332","61332","Maintenance and Repairs of Movable Property","expense","False","Entretien et réparations des biens mobiliers"
"pcg_61335","61335","Maintenance","expense","False","Maintenance"
"pcg_61341","61341","Multi-Risk Insurances (Theft, Fire, Civil Liability)","expense","False","Assurances multirisque (Vol, incendie, RC)"
"pcg_61343","61343","Insurances - Business Risks","expense","False","Assurances - Risques d'exploitation"
"pcg_61345","61345","Insurances - Transport Equipment","expense","False","Assurances - Matériel de transport"
"pcg_61348","61348","Other Insurances","expense","False","Autres assurances"
"pcg_61351","61351","Remuneration of Occasional Personnel","expense","False","Rémunérations du personnel occasionnel"
"pcg_61352","61352","Remuneration of Temporary Personnel","expense","False","Rémunérations du personnel intérimaire"
"pcg_61353","61353","Remuneration of Personnel Seconded or Loaned to the Company","expense","False","Rémunérations du personnel détaché ou prêté à l'entreprise"
"pcg_61361","61361","Commissions and Brokerage","expense","False","Commissions et courtages"
"pcg_61365","61365","Fees","expense","False","Honoraires"
"pcg_61367","61367","Legal and Litigation Costs","expense","False","Frais d'actes et de contentieux"
"pcg_61371","61371","Patent Royalties","expense","False","Redevances pour brevets"
"pcg_61378","61378","Other Royalties","expense","False","Autres redevances"
"pcg_61411","61411","General Studies","expense","False","Etudes générales"
"pcg_61413","61413","Research","expense","False","Recherches"
"pcg_61415","61415","General Documentation","expense","False","Documentation générale"
"pcg_61416","61416","Technical Documentation","expense","False","Documentation technique"
"pcg_61421","61421","Transport of Personnel","expense","False","Transports du personnel"
"pcg_61425","61425","Transport on Purchases","expense","False","Transports sur achats"
"pcg_61426","61426","Transport on Sales","expense","False","Transports sur ventes"
"pcg_61428","61428","Other Transport","expense","False","Autres transports"
"pcg_61431","61431","Travels and Displacements","expense","False","Voyages et déplacements"
"pcg_61433","61433","Moving Expenses","expense","False","Frais de déménagement"
"pcg_61435","61435","Missions","expense","False","Missions"
"pcg_61436","61436","Receptions","expense","False","Réceptions"
"pcg_61441","61441","Advertisements and Insertions","expense","False","Annonces et insertions"
"pcg_61442","61442","Samples, Catalogs and Printed Advertising","expense","False","Echantillons, catalogues et imprimés publicitaires"
"pcg_61443","61443","Fairs and Exhibitions","expense","False","Foires et expositions"
"pcg_61444","61444","Advertising Bonuses","expense","False","Primes de publicité"
"pcg_61446","61446","Publications","expense","False","Publications"
"pcg_61447","61447","Gifts to Customers","expense","False","Cadeaux à la clientèle"
"pcg_61448","61448","Other Advertising and Public Relations Expenses","expense","False","Autres charges de publicité et relations publiques"
"pcg_61451","61451","Postal Expenses","expense","False","Frais postaux"
"pcg_61455","61455","Telephone Expenses","expense","False","Frais de téléphone"
"pcg_61456","61456","Telex and Telegram Expenses","expense","False","Frais de télex et de télégrammes"
"pcg_61461","61461","Contributions","expense","False","Cotisations"
"pcg_61462","61462","Donations","expense","False","Dons"
"pcg_61471","61471","Expenses of Purchase and Sale of Securities","expense","False","Frais d'achat et de vente des titres"
"pcg_61472","61472","Commercial Paper Expenses","expense","False","Frais sur effets de commerce"
"pcg_61473","61473","Fees and Commissions on Banking Services","expense","False","Frais et commissions sur services bancaires"
"pcg_6148","6148","Other External Expenses from Previous Years","expense","False","Autres charges externes des exercices antérieurs"
"pcg_6149","6149","Discounts, Rebates and Refunds Obtained on Other External Expenses","expense","False","Rabais, remises et ristournes obtenus sur autres charges externes"
"pcg_61611","61611","City Tax and Municipal Tax","expense","False","Taxe urbaine et taxe d'édilité"
"pcg_61612","61612","Patent","expense","False","Patente"
"pcg_61615","61615","Local Taxes","expense","False","Taxes locales"
"pcg_6165","6165","Indirect Taxes and Duties","expense","False","Impôts et taxes indirects"
"pcg_61671","61671","Registration and Stamp Duties","expense","False","Droits d'enregistrement et de timbre"
"pcg_61673","61673","Vehicle Taxes","expense","False","Taxes sur les véhicules"
"pcg_61678","61678","Other Taxes, Duties and Similar Rights","expense","False","Autres impôts, taxes et droits assimilés"
"pcg_6168","6168","Taxes and Duties from Previous Years","expense","False","Impôts et taxes des exercices antérieurs"
"pcg_61711","61711","Salaries and Wages","expense","False","Appointements et salaires"
"pcg_61712","61712","Bonuses and Gratuities","expense","False","Primes et gratifications"
"pcg_61713","61713","Allowances and Other Benefits","expense","False","Indemnités et avantages divers"
"pcg_61714","61714","Commisions to Personnel","expense","False","Commissions au personnel"
"pcg_61715","61715","Remuneration of Directors, Managers and Partners","expense","False","Rémunération des administrateurs, gérants et associés"
"pcg_61741","61741","Social Security Contributions","expense","False","Cotisations de sécurité sociale"
"pcg_61742","61742","Contributions to Pension Funds","expense","False","Cotisations aux caisses de retraite"
"pcg_61743","61743","Contributions to Mutual Insurance Companies","expense","False","Cotisations aux mutuelles"
"pcg_61744","61744","Family Benefits","expense","False","Prestations familiales"
"pcg_61745","61745","Work Accident Insurances","expense","False","Assurances accident de travail"
"pcg_61761","61761","Group Insurances","expense","False","Assurances groupe"
"pcg_61762","61762","Pension Benefits","expense","False","Prestations de retraites"
"pcg_61763","61763","Allocations to Social Works","expense","False","Allocations aux oeuvres sociales"
"pcg_61764","61764","Clothing and Workwear","expense","False","Habillement et vêtements de travail"
"pcg_61765","61765","Compensation in Lieu of Notice and Termination of Employment","expense","False","Indemnités de préavis et de licenciement"
"pcg_61766","61766","Occupational Medicine and Pharmacy","expense","False","Médecine de travail, pharmacie"
"pcg_61768","61768","Other Various Social Charges","expense","False","Autres charges sociales diverses"
"pcg_6177","6177","Operator's Remuneration","expense","False","Rémunération de l'exploitant"
"pcg_61771","61771","Operator's Salaries and Wages","expense","False","Appointements et salaires de l'exploitant"
"pcg_61774","61774","Social Charges on Operator's Wages and Salaries","expense","False","Charges sociales sur appointements et salaires de l'exploitant"
"pcg_6178","6178","Personnel Expenses of Previous Years","expense","False","Charges de personnel des exercices antérieurs"
"pcg_6181","6181","Attendance Fees","expense","False","Jetons de présence"
"pcg_6182","6182","Losses on Uncollectible Receivables","expense","False","Pertes sur créances irrécouvrables"
"pcg_6185","6185","Losses on Joint Operations","expense","False","Pertes sur opérations faites en commun"
"pcg_6186","6186","Transfer of Profits from Joint Operations","expense","False","Transfert de profits sur opérations faites en commun"
"pcg_6188","6188","Other Operating Expenses from Previous Years","expense","False","Autres charges d'exploitation des exercices antérieurs"
"pcg_61911","61911","Operating Allocations for Amortization of Preliminary Costs","expense","False","Dotations d'exploitation aux amortissements des frais préliminaires"
"pcg_61912","61912","Operating Allocations for Amortization of Charges to be Distributed","expense","False","Dotations d'exploitation aux amortissements des charges à répartir"
"pcg_61921","61921","Operating Allocations for Amortization of Assets in Research and Development","expense","False","Dotations d'exploitation aux amortissements de l'immobilisation en recherche et développement"
"pcg_61922","61922","Operating Allocations for Amortization of Patents, Trademarks, Rights and Similar Assets","expense","False","Dotations d'exploitation aux amortissements des brevets, marques, droits et valeurs similaires"
"pcg_61923","61923","Operating Allocations for Amortization of Commercial Funds","expense","False","Dotations d'exploitation aux amortissements du fonds commercial"
"pcg_61928","61928","Operating Allocations for Amortization of Other Intangible Non-Current Assets","expense","False","Dotations d'exploitation aux amortissements des autres immobilisations incorporelles"
"pcg_61931","61931","Operating Allocations for Depreciation of Land","expense","False","Dotations d'exploitation aux amortissements des terrains"
"pcg_61932","61932","Operating Allocations for Depreciation of Constructions","expense","False","Dotations d'exploitation aux amortissements des constructions"
"pcg_61933","61933","Operating Allocations for Depreciation of Technical Installations, Equipment and Tools","expense","False","Dotations d'exploitation aux amortissements des installations techniques, matériel et outillage"
"pcg_61934","61934","Operating Allocations for Depreciation of Transport Equipment","expense","False","Dotations d'exploitation aux amortissements du matériel de transport"
"pcg_61935","61935","Operating Allocations for Depreciation of Furniture, Office Equipment, Miscellaneous Amenities","expense","False","Dotations d'exploitation aux amortissements des mobilier, matériel de bureau et aménagements divers"
"pcg_61938","61938","Operating Allocations for Depreciation of Other Tangible Non-Current Assets","expense","False","Dotations d'exploitation aux amortissements des autres immobilisations corporelles"
"pcg_61942","61942","Operating Allocations to Provisions for Depreciation of Intangible Non-Current Assets","expense","False","Dotations d'exploitation aux provisions pour dépréciation des immobilisations incorporelles"
"pcg_61943","61943","Operating Allocations to Provisions for Depreciation of Tangible Non-Current Assets","expense","False","Dotations d'exploitation aux provisions pour dépréciation des immobilisations corporelles"
"pcg_61955","61955","Operating Allocations to Provisions for Long-Term Risks and Expenses","expense","False","Dotations d'exploitation aux provisions pour risques et charges durables"
"pcg_61957","61957","Operating Allocations to Provisions for Temporary Risks and Expenses","expense","False","Dotations d'exploitation aux provisions pour risques et charges momentanés"
"pcg_61961","61961","Operating Allocations to Provisions for Depreciation of Inventory","expense","False","Dotations d'exploitation aux provisions pour dépréciation des stocks"
"pcg_61964","61964","Operating Allocations to Provisions for Depreciation of Receivables from Current Assets","expense","False","Dotations d'exploitation aux provisions pour dépréciation des créances de l'actif circulant"
"pcg_61981","61981","Operating Allocations for Amortization or Depreciation from Previous Years","expense","False","Dotations d'exploitation aux amortissements des exercices antérieurs"
"pcg_61984","61984","Operating Allocations to Provisions from Previous Years","expense","False","Dotations d'exploitation aux provisions des exercices antérieurs"
"pcg_63111","63111","Interest on Loans","expense","False","Intérêts des emprunts"
"pcg_63113","63113","Interest on Debts Related to Equity Investments","expense","False","Intérêts des dettes rattachées à des participations"
"pcg_63114","63114","Interest on Current Accounts and Deposits","expense","False","Intérêts des comptes courants et dépôts créditeurs"
"pcg_63115","63115","Interest on Bank and Financing Operations","expense","False","Intérêts bancaires et sur opérations de financement"
"pcg_63118","63118","Other Interest on Loans and Debts","expense","False","Autres intérêts des emprunts et dettes"
"pcg_6318","6318","Interest Expenses from Previous Years","expense","False","Charges d'intérêts des exercices antérieurs"
"pcg_6331","6331","Foreign Exchange Losses for the Year","expense","False","Pertes de change propres à l'exercice"
"pcg_6338","6338","Foreign Exchange Losses from Previous Years","expense","False","Pertes de change des exercices antérieurs"
"pcg_6382","6382","Losses on Receivables from Equity Investments","expense","False","Pertes sur créances liées à des participations"
"pcg_6385","6385","Net Expenses on Disposal of Securities and Investment Assets","expense","False","Charges nettes sur cessions de titres et valeurs de placement"
"pcg_63861","63861","Cash Difference","expense","False","Différence de trésorerie"
"pcg_63862","63862","Early Payment Discount","expense","False","Escompte pour paiement anticipé"
"pcg_63868","63868","Other Discounts Granted","expense","False","Autres escomptes accordés"
"pcg_6388","6388","Other Financial Expenses from Previous Years","expense","False","Autres charges financières des exercices antérieurs"
"pcg_6391","6391","Allocations for Amortization of Bond Redemption Premiums","expense","False","Dotations aux amortissements des primes de remboursement des obligations"
"pcg_6392","6392","Allocations to Provisions for Depreciation of Financial Non-Current Assets","expense","False","Dotations aux provisions pour dépréciation des immobilisations financières"
"pcg_6393","6393","Allocations to Provisions for Financial Risks and Expenses","expense","False","Dotations aux provisions pour risques et charges financiers"
"pcg_6394","6394","Allocations to Provisions for Depreciation of Securities and Investment Assets","expense","False","Dotations aux provisions pour dépréciation des titres et valeurs de placement"
"pcg_6396","6396","Allocations to Provisions for Depreciation of Cash Accounts","expense","False","Dotations aux provisions pour dépréciation des comptes de trésorerie"
"pcg_6398","6398","Financial Allocations from Previous Years","expense","False","Dotations financières des exercices antérieurs"
"pcg_6512","6512","Net Depreciation Value of Intangible Non-Current Assets Sold","expense_depreciation","False","Valeurs nettes d'amortissements des immobilisations incorporelles cédées"
"pcg_6513","6513","Net Depreciation Value of Tangible Non-Current Assets Sold","expense_depreciation","False","Valeurs nettes d'amortissements des immobilisations corporelles cédées"
"pcg_6514","6514","Net Depreciation Value of Financial Non-Current Assets Sold (Ownership Rights)","expense_depreciation","False","Valeurs nettes d'amortissements des immobilisations financières cédées (droits de propriété)"
"pcg_6518","6518","Net Depreciation Value of Non-Current Assets Sold from Previous Years","expense_depreciation","False","Valeurs nettes d'amortissements des immobilisations cédées des exercices antérieurs"
"pcg_6561","6561","Subsidies Granted for the Year","expense","False","Subventions accordées de l'exercice"
"pcg_6568","6568","Subsidies Granted from Previous Years","expense","False","Subventions accordés des exercices antérieurs"
"pcg_65811","65811","Contract Penalties","expense","False","Pénalités sur marchés"
"pcg_65812","65812","Forfeitures","expense","False","Dédits"
"pcg_6582","6582","Tax Reminders (Other than Income Tax)","expense","False","Rappels d'impôts (autres qu'impôts sur les résultats)"
"pcg_65831","65831","Tax Penalties and Fines","expense","False","Pénalités et amendes fiscales"
"pcg_65833","65833","Criminal Penalties and Fines","expense","False","Pénalités et amendes pénales"
"pcg_6585","6585","Receivables that Have Become Uncollectible","expense","False","Créances devenues irrécouvrables"
"pcg_65861","65861","Donations","expense","False","Dons"
"pcg_65862","65862","Liberalities","expense","False","Libéralités"
"pcg_65863","65863","Prizes","expense","False","Lots"
"pcg_6588","6588","Other Non-Current Expenses from Previous Years","expense","False","Autres charges non courantes des exercices antérieurs"
"pcg_65911","65911","Allocations to Exceptional Depreciation of Non-Current Assets in Non-Values","expense","False","Dotations aux amortissements exceptionnels de l'immobilisation en non-valeurs"
"pcg_65912","65912","Allocations to Exceptional Depreciation of Intangible Non-Current Assets","expense","False","Dotations aux amortissements exceptionnels des immobilisations incorporelles"
"pcg_65913","65913","Allocations to Exceptional Depreciation of Tangible Non-Current Assets","expense","False","Dotations aux amortissements exceptionnels des immobilisations corporelles"
"pcg_65941","65941","Non-Current Allocations for Depreciation","expense","False","Dotations non courantes pour amortissements"
"pcg_65942","65942","Non-Current Allocations for Capital Gains Pending Taxation","expense","False","Dotations non courantes pour plus-values en instance d'imposition"
"pcg_65944","65944","Non-Current Allocations for Investments","expense","False","Dotations non courantes pour investissements"
"pcg_65945","65945","Non-Current Allocations for Replenishment of Deposits","expense","False","Dotations non courantes pour reconstitution de gisements"
"pcg_65946","65946","Non-Current Allocations for the Acquisition and Construction of Housing","expense","False","Dotations non courantes pour acquisition et construction de logements"
"pcg_65955","65955","Non-Current Allocations to Provisions for Long-Term Risks and Expenses","expense","False","Dotations non courantes aux provisions pour risques et charges durables"
"pcg_65957","65957","Non-Current Allocations to Provisions for Temporary Risks and Expenses","expense","False","Dotations non courantes aux provisions pour risques et charges momentanés"
"pcg_65962","65962","Non-Current Allocations to Provisions for Depreciation of Non-Current Assets","expense","False","Dotations non courantes aux amortissements pour dépréciation de l'actif immobilisé"
"pcg_65963","65963","Non-Current Allocations to Provisions for Depreciation of Current Assets","expense","False","Dotations non courantes aux provisions pour dépréciation de l'actif circulant"
"pcg_6598","6598","Non-Current Allocations from Previous Years","expense","False","Dotations non courantes des exercices antérieurs"
"pcg_6701","6701","Income Taxes on Profits","expense","False","Impôts sur les bénéfices"
"pcg_6705","6705","Minimum Annual Corporate Taxation","expense","False","Imposition minimale annuelle des sociétés"
"pcg_6708","6708","Income Tax Reminders and Relief","expense","False","Rappels et dégrèvements des impôts sur les résultats"
"pcg_7111","7111","Sales of Goods in Morocco","income","False","Ventes de marchandises au Maroc"
"pcg_7113","7113","Sales of Goods Abroad","income","False","Ventes de marchandises à l'étranger"
"pcg_7118","7118","Sales of Goods from Previous Years","income","False","Ventes de marchandises des exercices antérieurs"
"pcg_7119","7119","Discounts, Rebates and Refunds Granted by the Company","income","False","Rabais, remises et ristournes accordés par l'entreprise"
"pcg_71211","71211","Sales of Finished Products (Production in Morocco)","income","False","Ventes de produits finis (Production au Maroc)"
"pcg_71212","71212","Sales of Intermediary Products (Production in Morroco)","income","False","Ventes de produits intermédiaires (Production au Maroc)"
"pcg_71217","71217","Sales of Residual Products (Production in Morocco)","income","False","Ventes de produits résiduels (Production au Maroc)"
"pcg_71221","71221","Sales of Finished Products (Production Abroad)","income","False","Ventes de produits finis (Production à l'étranger)"
"pcg_71222","71222","Sales of Intermediary Products (Production Abroad)","income","False","Ventes de produits intermédiaires (Production à l'étranger)"
"pcg_71241","71241","Works (Production in Morocco)","income","False","Travaux (Production au Maroc)"
"pcg_71242","71242","Studies (Production in Morocco)","income","False","Etudes (Production au Maroc)"
"pcg_71243","71243","Services Provided (Production in Morocco)","income","False","Prestations de service (Production au Maroc)"
"pcg_71251","71251","Works (Production Abroad)","income","False","Travaux (Production à l'étranger)"
"pcg_71252","71252","Studies (Production Abroad)","income","False","Etudes (Production à l'étranger)"
"pcg_71253","71253","Service Provided (Production Abroad)","income","False","Prestations de service (Production à l'étranger)"
"pcg_7126","7126","Royalties for Patents, Trademarks, Rights and Similar Assets","income","False","Redevances pour brevets, marques, droits et valeurs similaires"
"pcg_71271","71271","Miscellaneous Rentals Received","income","False","Locations diverses reçues"
"pcg_71272","71272","Commissions and Brokerage Fees Received","income","False","Commissions et courtages reçus"
"pcg_71273","71273","Income of Services Operated for the Benefit of the Personnel","income","False","Produits de services exploités dans l'intérêt du personnel"
"pcg_71275","71275","Bonuses on Returnable Packaging","income","False","Bonis sur reprises d'emballages consignés"
"pcg_71276","71276","Ports and Incidentals Charged","income","False","Ports et frais accessoires facturés"
"pcg_71278","71278","Other Related Sales and Income","income","False","Autres ventes et produits accessoires"
"pcg_7128","7128","Sales of Goods and Services Produced from Previous Years","income","False","Ventes de biens et services produits des exercices antérieurs"
"pcg_71291","71291","Discounts, Rebates and Refunds Granted on Sales in Morocco of Produced Goods","income","False","Rabais, Remises et Ristournes accordés sur ventes au Maroc des biens produits"
"pcg_71292","71292","Discounts, Rebates and Refunds Granted on Sales Abroad of Produced Goods","income","False","Rabais, Remises et Ristournes accordés sur ventes à l'étranger des biens produits"
"pcg_71294","71294","Discounts, Rebates and Refunds Granted on Sales in Morocco of Produced Services","income","False","Rabais, Remises et Ristournes accordés sur ventes au Maroc des services produits"
"pcg_71295","71295","Discounts, Rebates and Refunds Granted on Sales Abroad of Produced Services","income","False","Rabais, Remises et Ristournes accordés sur ventes à l'étranger des services produits"
"pcg_71298","71298","Discounts, Rebates and Refunds Granted on Sales of Goods and Services from Previous Years","income","False","Rabais, Remises et Ristournes accordés sur ventes de biens et services produits des exercices antérieurs"
"pcg_71311","71311","Changes in Inventory of Produced Goods in Progress","income","False","Variation des stocks de biens produits en cours"
"pcg_71312","71312","Changes in Inventory of Intermediary Products in Progress","income","False","Variation des stocks de produits intermédiaires en cours"
"pcg_71317","71317","Changes in Inventory of Residual Products in Progress","income","False","Variation des stocks de produits résiduels en cours"
"pcg_71321","71321","Changes in Inventory of Finished Products","income","False","Variation des stocks de produits finis"
"pcg_71322","71322","Changes in Inventory of Intermediary Products","income","False","Variation des stocks de produits intermédiaires"
"pcg_71327","71327","Changes in Inventory of Residual Products","income","False","Variation des stocks de produits résiduels"
"pcg_71341","71341","Changes in Inventory of Works in Progress","income","False","Variation des stocks de travaux en cours"
"pcg_71342","71342","Changes in Inventory of Studies in Progress","income","False","Variation des stocks d'études en cours"
"pcg_71343","71343","Changes in Inventory of Services Provided in Progress","income","False","Variation des stocks de prestations en cours"
"pcg_7140","7140","Non-Current Assets Produced by the Company for its Own Use","income","False","Immobilisations produites par l'entreprise pour elle-même"
"pcg_7141","7141","Non-Current Assets in Non-Values Produced","income","False","Immobilisations en non valeurs produites"
"pcg_7142","7142","Intangible Non-Current Assets Produced","income","False","Immobilisations incorporelles produites"
"pcg_7143","7143","Tangible Non-Current Assets Produced","income","False","Immobilisations corporelles produites"
"pcg_7148","7148","Non-Current Assets Produced from Previous Years","income","False","Immobilisations produites des exercices antérieurs"
"pcg_7161","7161","Operation Subsidies Received for the Year","income","False","Subventions d'exploitation reçues de l'exercice"
"pcg_7168","7168","Operation Subsidies Received for Previous Years","income","False","Subventions d'exploitation reçues des exercices antérieurs"
"pcg_7180","7180","Other Operating Income","income","False","Autres produits d'exploitation"
"pcg_7181","7181","Attendance Fees Received","income","False","Jetons de présence reçus"
"pcg_7182","7182","Income from Real Estate not Assigned to Operations","income","False","Revenus des immeubles non affectés à l'exploitation"
"pcg_7185","7185","Profits on Joint Operations","income","False","Profits sur opérations faites en commun"
"pcg_7186","7186","Transfer of Losses on Joint Operations","income","False","Transfert de pertes sur opérations faites en commun"
"pcg_7188","7188","Other Operating Income from Previous Years","income","False","Autres produits d'exploitation des exercices antérieurs"
"pcg_7190","7190","Operating Reversals; Expense Transfers","income","False","Reprises d'exploitation; Transferts de charges"
"pcg_7191","7191","Reversals of Depreciation of Non-Current Assets in Non-Values","income","False","Reprises sur amortissements de l'immobilisation en non-valeurs"
"pcg_7192","7192","Reversals of Depreciation of Intangible Non-Current Assets","income","False","Reprises sur amortissements des immobilisations incorporelles"
"pcg_7193","7193","Reversals of Depreciation of Tangible Non-Current Assets","income","False","Reprises sur amortissement des immobilisations corporelles"
"pcg_7194","7194","Reversals of Provisions for Depreciation of Non-Current Assets","income","False","Reprises sur provisions pour dépréciation des immobilisations"
"pcg_7195","7195","Reversals of Provisions for Risks and Expenses","income","False","Reprises sur provisions pour risques et charges"
"pcg_7196","7196","Reversals of Provisions for Depreciation of Current Assets","income","False","Reprises sur provisions pour dépréciation de l'actif circulant"
"pcg_71971","71971","Transfers of Operating Expenses - Purchases of Goods","income","False","Transferts de charges d'exploitation - Achats de marchandises"
"pcg_71972","71972","Transfers of Operating Expenses - Consumed Purchases of Materials and Supplies","income","False","Transferts de charges d'exploitation - Achats consommés de matières et fournitures"
"pcg_71973","71973","Transfers of Operating Expenses - Other External Expenses","income","False","Transferts de charges d'exploitation - Autres charges externes"
"pcg_71975","71975","Transfers of Operating Expenses - Taxes and Duties","income","False","Transferts de charges d'exploitation - Impôts et taxes"
"pcg_71976","71976","Transfers of Operating Expenses - Personnel Expenses","income","False","Transferts de charges d'exploitation - Charges du personnel"
"pcg_71978","71978","Transfers of Operating Expenses - Other Operating Expenses","income","False","Transferts de charges d'exploitation - Autres charges d'exploitation"
"pcg_71981","71981","Reversals of Depreciation from Previous Years","income","False","Reprises sur amortissements des exercices antérieurs"
"pcg_71984","71984","Reversals of Provisions from Previous Years","income","False","Reprises sur provisions des exercices antérieurs"
"pcg_7321","7321","Income from Equity Securities","income_other","False","Revenus des titres de participation"
"pcg_7325","7325","Income from Other Non-Current Assets","income_other","False","Revenus des autres titres immobilisés"
"pcg_7328","7328","Income from Equity Securities and Other Non-Current Assets from Previous Years","income_other","False","Produits des titres de participation et des autres titres immobilisés des exercices antérieurs"
"pcg_7331","7331","Foreign Exchange Gains for the Year","income_other","False","Gains de change propres à l'exercice"
"pcg_7338","7338","Foreign Exchange Gains from Previous Years","income_other","False","Gains de change des exercices antérieurs"
"pcg_73811","73811","Interest from Loans","income_other","False","Intérêts des prêts"
"pcg_73813","73813","Income from Other Financial Receivables","income_other","False","Revenus des autres créances financières"
"pcg_7383","7383","Income from Receivables Related to Equity Investments","income_other","False","Revenus des créances rattachées à des participations"
"pcg_7384","7384","Income from Securities and Investment Assets","income_other","False","Revenus des titres et valeurs de placement"
"pcg_7385","7385","Net Income on Disposal of Securities and Investment Assets","income_other","False","Produits nets sur cessions de titres et valeurs de placement"
"pcg_73861","73861","Cash Difference","income_other","False","Différence de trésorerie"
"pcg_73862","73862","Early Payment Discount","income_other","False","Escompte pour paiement anticipé"
"pcg_73868","73868","Other Discounts Obtained","income_other","False","Autres escomptes obtenus"
"pcg_7388","7388","Interest and Other Financial Income from Previous Years","income_other","False","Intérêts et autres produits financiers des exercices antérieurs"
"pcg_7391","7391","Reversals of Amortization of Bond Redemption Premiums","income_other","False","Reprises sur amortissements des primes de remboursement des obligations"
"pcg_7392","7392","Reversals of Provisions for Depreciation of Financial Non-Current Assets","income_other","False","Reprises sur provisions pour dépréciation des immobilisations financières"
"pcg_7393","7393","Reversals of Provisions for Financial Risks and Expenses","income_other","False","Reprises sur provisions pour risques et charges financiers"
"pcg_7394","7394","Reversals of Provisions for Depreciation of Securities and Investment Assets","income_other","False","Reprises sur provisions pour dépréciation des titres et valeurs de placement"
"pcg_7396","7396","Reversals of Provisions for Depreciation of Cash Accounts","income_other","False","Reprises sur provisions pour dépréciation des comptes de trésorerie"
"pcg_73971","73971","Transfer of Financial Expenses - Interest Expenses","income_other","False","Transferts de charges financières - Charges d'intérêts"
"pcg_73973","73973","Transfer of Financial Expenses - Foreign Exchange Losses","income_other","False","Transferts de charges financières - Pertes de change"
"pcg_73978","73978","Transfer of Financial Expenses - Other Financial Expenses","income_other","False","Transferts de charges financières - Autres charges financières"
"pcg_7398","7398","Reversals of Financial Allocations from Previous Years","income_other","False","Reprises sur dotations financières des exercices antérieurs"
"pcg_7512","7512","Income from Disposal of Intangible Non-Current Assets","income_other","False","Produits des cessions des immobilisations incorporelles"
"pcg_7513","7513","Income from Disposal of Tangible Non-Current Assets","income_other","False","Produits des cessions des immobilisations corporelles"
"pcg_7514","7514","Income from Disposal of Financial Non-Current Assets (Ownership Rights)","income_other","False","Produits des cessions des immobilisations financières (Droits de propriété)"
"pcg_7518","7518","Income from Disposals from Previous Years","income_other","False","Produits des cessions des exercices antérieurs"
"pcg_7561","7561","Balancing Subsidies Received for the Year","income_other","False","Subventions d'équilibre reçues de l'exercice"
"pcg_7568","7568","Balancing Subsidies Received from Previous Years","income_other","False","Subventions d'équilibre reçues des exercices antérieurs"
"pcg_7577","7577","Reversals of Investment Subsidies for the Year","income_other","False","Reprises sur subventions d'investissement de l'exercice"
"pcg_7578","7578","Reversals of Investment Subsidies of Previous Years","income_other","False","Reprises sur subventions d'investissement des exercices antérieurs"
"pcg_75811","75811","Contract Penalties Received","income_other","False","Pénalités reçus sur marchés"
"pcg_75812","75812","Forfeitures Received","income_other","False","Dédits reçus"
"pcg_7582","7582","Tax Relief (Other than Income Tax)","income_other","False","Dégrèvements d'impôts (Autres qu'impôts sur les résultats)"
"pcg_7585","7585","Returns on Settled Receivables","income_other","False","Rentrées sur créances soldées"
"pcg_75861","75861","Donations","income_other","False","Dons"
"pcg_75862","75862","Liberalities","income_other","False","Libéralités"
"pcg_75863","75863","Prizes","income_other","False","Lots"
"pcg_7588","7588","Other Non-Current Income from Previous Years","income_other","False","Autres produits non courants des exercices antérieurs"
"pcg_75911","75911","Non-Current Reversals of Exceptional Depreciation of Non-Current Assets in Non Values","income_other","False","Reprises non courantes sur amortissements exceptionnels de l'immobilisation en non valeurs"
"pcg_75912","75912","Non-Current Reversals of Exceptional Depreciation of Intangible Non-Current Assets","income_other","False","Reprises non courantes sur amortissements exceptionnels des immobilisations incorporelles"
"pcg_75913","75913","Non-Current Reversals of Exceptional Depreciation of Tangible Non-Current Assets","income_other","False","Reprises non courantes sur amortissements exceptionnels des immobilisations corporelles"
"pcg_75941","75941","Reversals of Accelerated Depreciation","income_other","False","Reprises sur amortissements dérogatoires"
"pcg_75942","75942","Reversals of Capital Gains Pending Taxation","income_other","False","Reprises sur plus-values en instance d'imposition"
"pcg_75944","75944","Reversals of Allocations for Investments","income_other","False","Reprises sur provisions pour investissements"
"pcg_75945","75945","Reversals of Allocations for Replenishment of Deposits","income_other","False","Reprises sur provisions pour reconstitution de gisements"
"pcg_75946","75946","Reversals of Allocations for the Acquisition and Construction of Housing","income_other","False","Reprises sur provisions pour acquisition et construction de logements"
"pcg_75955","75955","Reversals of Provisions for Long-Term Risks and Expenses","income_other","False","Reprises sur provisions pour risques et charges durables"
"pcg_75957","75957","Reversals of Provisions for Temporary Risks and Expenses","income_other","False","Reprises sur provisions pour risques et charges momentanés"
"pcg_75962","75962","Non-Current Reversals of Provisions for Depreciation of Non-Current Assets","income_other","False","Reprises non courantes sur provisions pour dépréciation de l'actif immobilisé"
"pcg_75963","75963","Non-Current Reversals of Provisions for Depreciation of Current Assets","income_other","False","Reprises non courantes sur provisions pour dépréciation de l'actif circulant"
"pcg_7597","7597","Non-Current Expense Transfers","income_other","False","Transferts de charges non courantes"
"pcg_7598","7598","Non-Current Reversals from Previous Years","income_other","False","Reprises non courantes des exercices antérieurs"

```

## File: data\template\account.fiscal.position-ma.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@fr"
"afpt_morocco","1","Morocco","1","","base.ma","","","","","","Maroc"
"afpt_imp_exp","2","Import/Export","1","","","","","","","","Importation/Exportation"
"","","","","","","","","","pcg_7111","pcg_7113",""
"","","","","","","","","","pcg_71211","pcg_71221",""
"","","","","","","","","","pcg_71212","pcg_71222",""
"","","","","","","","","","pcg_71242","pcg_71252",""
"","","","","","","","","","pcg_71243","pcg_71253",""
"","","","","","","","","","pcg_71294","pcg_71295",""
"","","","","","","","vat_out_20_80","vat_out_20_129","","",""
"","","","","","","","vat_out_20_81","vat_out_20_129","","",""
"","","","","","","","vat_out_20_82","vat_out_20_129","","",""
"","","","","","","","vat_out_20_83","vat_out_20_129","","",""
"","","","","","","","vat_out_20_87","vat_out_20_129","","",""
"","","","","","","","vat_out_20_93","vat_out_20_129","","",""
"","","","","","","","vat_out_20_94","vat_out_20_129","","",""
"","","","","","","","vat_out_20_95","vat_out_20_129","","",""
"","","","","","","","vat_out_20_102","vat_out_20_129","","",""
"","","","","","","","vat_out_14_84","vat_out_14_129","","",""
"","","","","","","","vat_out_14_88","vat_out_14_129","","",""
"","","","","","","","vat_out_14_90","vat_out_14_129","","",""
"","","","","","","","vat_out_14_104","vat_out_14_129","","",""
"","","","","","","","vat_out_14_91","vat_out_14_129","","",""
"","","","","","","","vat_out_10_76","vat_out_10_129","","",""
"","","","","","","","vat_out_10_77","vat_out_10_129","","",""
"","","","","","","","vat_out_10_78","vat_out_10_129","","",""
"","","","","","","","vat_out_10_79","vat_out_10_129","","",""
"","","","","","","","vat_out_10_85","vat_out_10_129","","",""
"","","","","","","","vat_out_10_86","vat_out_10_129","","",""
"","","","","","","","vat_out_10_89","vat_out_10_129","","",""
"","","","","","","","vat_out_10_92","vat_out_10_129","","",""
"","","","","","","","vat_out_10_96","vat_out_10_129","","",""
"","","","","","","","vat_out_10_97","vat_out_10_129","","",""
"","","","","","","","vat_out_10_98","vat_out_10_129","","",""
"","","","","","","","vat_out_10_99","vat_out_10_129","","",""
"","","","","","","","vat_out_10_100","vat_out_10_129","","",""
"","","","","","","","vat_out_10_101","vat_out_10_129","","",""
"","","","","","","","vat_out_10_103","vat_out_10_129","","",""
"","","","","","","","vat_out_10_105","vat_out_10_129","","",""
"","","","","","","","vat_out_10_108","vat_out_10_129","","",""
"","","","","","","","vat_out_10_109","vat_out_10_129","","",""
"","","","","","","","vat_out_10_112","vat_out_10_129","","",""
"","","","","","","","vat_out_10_118","vat_out_10_129","","",""
"","","","","","","","vat_out_7_106","vat_out_7_129","","",""
"","","","","","","","vat_out_7_107","vat_out_7_129","","",""
"","","","","","","","vat_out_7_110","vat_out_7_129","","",""
"","","","","","","","vat_out_7_111","vat_out_7_129","","",""
"","","","","","","","vat_out_7_113","vat_out_7_129","","",""
"","","","","","","","vat_out_7_114","vat_out_7_129","","",""
"","","","","","","","vat_out_7_115","vat_out_7_129","","",""
"","","","","","","","vat_out_7_116","vat_out_7_129","","",""
"","","","","","","","vat_out_7_117","vat_out_7_129","","",""
"","","","","","","","vat_out_7_119","vat_out_7_129","","",""
"","","","","","","","vat_in_20_146","vat_in_20_145","","",""
"","","","","","","","vat_in_14_148","vat_in_14_147","","",""
"","","","","","","","vat_in_10_150","vat_in_10_149","","",""
"","","","","","","","vat_in_7_152","vat_in_7_151","","",""
"","","","","","","","vat_in_20_163","vat_in_20_162","","",""

```

## File: data\template\account.group-ma.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@fr"
"group_1","1","","Permanent Funding","Financement permanent"
"group_11","11","","Shareholders' Equity","Capitaux propres"
"group_111","111","","Corporate or Personal Capital","Capital social ou personnel"
"group_112","112","","Share, Merger and Contribution Premiums","Primes d'émission, de fusion et d'apport"
"group_113","113","","Revaluation Differences","Écarts de réévaluation"
"group_114","114","","Legal Reserve","Réserve légale"
"group_115","115","","Other Reserves","Autres réserves"
"group_116","116","","Retained Result","Report à nouveau"
"group_118","118","","Net Results Pending Allocation","Résultats nets en instance d'affectation"
"group_119","119","","Net Result for the Fiscal Year","Résultat net de l'exercice"
"group_13","13","","Related Shareholders' Equity","Capitaux propres assimilés"
"group_131","131","","Investment Grants","Subventions d'investissement"
"group_135","135","","Regulated Provisions","Provisions réglementées"
"group_14","14","","Financial Debts","Dettes de financement"
"group_141","141","","Bond Loans","Emprunts obligataires"
"group_148","148","","Other Financial Liabilities","Autres dettes de financement"
"group_15","15","","Long-Term Provisions for Risks and Expenses","Provisions durables pour risques et charges"
"group_151","151","","Provisions for Risks","Provisions pour risques"
"group_155","155","","Provisions for Expenses","Provisions pour charges"
"group_16","16","","Establishments' and Branches' Liaison Accounts","Comptes de liaison des établissements et succursales"
"group_160","160","","Establishments' and Branches' Liaison Accounts","Comptes de liaison des établissements et succursales"
"group_17","17","","Foreign Exchange Liabilities","Écarts de conversion passif"
"group_171","171","","Increase in Non-Current Receivables","Augmentation des créances immobilisées"
"group_172","172","","Decrease in Financing Liabilities","Diminution des dettes de financement"
"group_2","2","","Non-Current Assets","Actif immobilisé"
"group_21","21","","Non-Value Non-Current Assets","Immobilisations en non-valeurs"
"group_211","211","","Start-Up Costs","Frais préliminaires"
"group_212","212","","Deferred Expenses over Several Years","Charges à répartir sur plusieurs exercices"
"group_213","213","","Bond Redemption Premiums","Primes de remboursement des obligations"
"group_22","22","","Intangible Non-Current Assets","Immobilisations incorporelles"
"group_221","221","","Non-Current Assets in Research and Development","Immobilisation en recherche et développement"
"group_222","222","","Patents, Trademarks, Rights and Similar Assets","Brevets, marques, droits et valeurs similaires"
"group_223","223","","Commercial Funds","Fonds commercial"
"group_228","228","","Other Intangible Non-Current Assets","Autres immobilisations incorporelles"
"group_23","23","","Tangible Non-Current Assets","Immobilisations corporelles"
"group_231","231","","Land","Terrains"
"group_232","232","","Constructions","Constructions"
"group_233","233","","Technical Installations, Equipment and Tools","Installations techniques, matériel et outillage"
"group_234","234","","Transport Equipment","Matériel de transport"
"group_235","235","","Furniture, Office Equipment, Miscellaneous Amenities","Mobilier, matériel de bureau, aménagements divers"
"group_238","238","","Other Tangible Non-Current Assets","Autres immobilisations corporelles"
"group_239","239","","Tangible Non-Current Assets in Progress","Immobilisations corporelles en cours"
"group_24","24","25","Financial Non-Current Assets","Immobilisations financières"
"group_241","241","","Non-Current Loans","Prêts immobilisés"
"group_248","248","","Other Financial Receivables","Autres créances financières"
"group_251","251","","Equity Securities","Titres de participation"
"group_258","258","","Other Long-Term Securities","Autres titres immobilisés"
"group_27","27","","Foreign Exchange Assets","Écarts de conversion actif"
"group_271","271","","Decrease in Non-Current Assets","Diminution des créances immobilisées"
"group_272","272","","Increase in Financing Liabilities","Augmentation des dettes de financement"
"group_28","28","","Depreciations of Non-Current Assets","Amortissements des immobilisations"
"group_281","281","","Depreciations of Non-Value Assets","Amortissements des non-valeurs"
"group_282","282","","Depreciations of Intangible Non-Current Assets","Amortissements des immobilisations incorporelles"
"group_283","283","","Depreciations of Tangible Non-Current Assets","Amortissements des immobilisations corporelles"
"group_29","29","","Provisions for Depreciation of Non-Current Assets","Provisions pour dépréciation des immobilisations"
"group_292","292","","Provisions for Depreciation of Intangible Non-Current Assets","Provisions pour dépréciation des immobilisations incorporelles"
"group_293","293","","Provisions for Depreciation of Tangible Non-Current Assets","Provisions pour dépréciation des immobilisations corporelles"
"group_294","294","295","Provisions for Depreciation of Financial Non-Current Assets","Provisions pour dépréciation des immobilisations financières"
"group_3","3","","Current Assets Excluding Cash","Actif circulant hors trésorerie"
"group_31","31","","Inventories","Stocks"
"group_311","311","","Goods","Marchandises"
"group_312","312","","Consumable Materials and Supplies","Matières et fournitures consommables"
"group_313","313","","Products in Progress","Produits en cours"
"group_314","314","","Intermediary and Residual Products","Produits intermédiaires et produits résiduels"
"group_315","315","","Finished Products","Produits finis"
"group_34","34","","Receivables from Current Assets","Créances de l'actif circulant"
"group_341","341","","Supplier Receivables, Advances and Down Payments","Fournisseurs débiteurs, avances et acomptes"
"group_342","342","","Customers and Related Accounts","Clients et comptes rattachés"
"group_343","343","","Personnel - Receivables","Personnel - Débiteur"
"group_345","345","","Government - Receivables","État - Débiteur"
"group_346","346","","Partner Accounts - Receivables","Comptes d'associés - Débiteurs"
"group_348","348","","Other Receivables","Autres débiteurs"
"group_349","349","","Accruals and Deferrals - Assets","Comptes de régularisation - Actif"
"group_35","35","","Securities and Investment Assets","Titres et valeurs de placement"
"group_350","350","","Securities and Investment Assets","Titres et valeurs de placement"
"group_37","37","","Foreign Exchange Assets (Current Part)","Écarts de conversion - Actif (Eléments circulants)"
"group_370","370","","Foreign Exchange Assets (Current Part)","Écarts de conversion - Actif (Eléments circulants)"
"group_39","39","","Provisions for Depreciation of Current Assets Accounts","Provisions pour dépréciation des comptes de l'actif circulant"
"group_391","391","","Provisions for Depreciation of Inventories","Provisions pour dépréciation des stocks"
"group_394","394","","Provisions for Depreciation of Receivables from Current Assets","Provisions pour dépréciation des créances de l'actif circulant"
"group_395","395","","Provisions for Depreciation of Securities and Investment Assets","Provisions pour dépréciation des titres et valeurs de placement"
"group_4","4","","Current Liabilities Excluding Cash","Passif circulant hors trésorerie"
"group_44","44","","Debts of Current Liabilities","Dettes du passif circulant"
"group_441","441","","Suppliers and Related Accounts","Fournisseurs et comptes rattachés"
"group_442","442","","Customer Payables, Advances and Down Payments","Clients créditeurs, avances et acomptes"
"group_443","443","","Personnel - Payables","Personnel - Créditeur"
"group_444","444","","Social Organizations - Payables","Organismes sociaux"
"group_445","445","","Government - Payables","État - Créditeur"
"group_446","446","","Partner Accounts - Payables","Comptes d'associés - Créditeurs"
"group_448","448","","Other Creditors","Autres créanciers"
"group_449","449","","Accruals and Deferrals - Liabilities","Comptes de régularisation - Passif"
"group_45","45","","Other Provisions for Risks and Expenses","Autres provisions pour risques et charges"
"group_450","450","","Other Provisions for Risks and Expenses","Autres provisions pour risques et charges"
"group_47","47","","Foreign Exchange Liabilities (Current Part)","Écarts de conversion - Passif (Eléments circulants)"
"group_470","470","","Foreign Exchange Liabilities (Current Part)","Écarts de conversion - Passif (Eléments circulants)"
"group_5","5","","Cash Accounts","Trésorerie"
"group_51","51","","Cash Accounts - Assets","Trésorerie - Actif"
"group_511","511","","Checks and Securities to be Cashed","Chèques et valeurs à encaisser"
"group_514","514","","Bank, General Treasury and Postal Checks in Debit","Banques, Trésorerie Générale et Chèques Postaux débiteurs"
"group_516","516","","Cash, Imprest Accounts and Letters of Credit","Caisses, régies d'avances et accréditifs"
"group_55","55","","Cash Accounts - Liabilities","Trésorerie - Passif"
"group_552","552","","Discount Credits","Crédits d'escompte"
"group_553","553","","Cash Credits","Crédits de trésorerie"
"group_554","554","","Banks (Credit Balances)","Banques (Soldes créditeurs)"
"group_59","59","","Provisions for Depreciation of Cash Accounts","Provisions pour dépréciation des comptes de trésorerie"
"group_590","590","","Provisions for Depreciation of Cash Accounts","Provisions pour dépréciation des comptes de trésorerie"
"group_6","6","","Expense Accounts","Comptes de charges"
"group_61","61","","Operating Expenses","Charges d'exploitation"
"group_611","611","","Purchases of Goods Resold","Achats revendus de marchandises"
"group_612","612","","Purchases of Materials and Supplies Consumed","Achats consommés de matières et fournitures"
"group_613","613","614","Other External Expenses","Autres charges externes"
"group_616","616","","Taxes and Duties","Impôts et taxes"
"group_617","617","","Personnel Expenses","Charges de personnel"
"group_618","618","","Other Operating Expenses","Autres charges d'exploitation"
"group_619","619","","Operating Allocations","Dotations d'exploitation"
"group_63","63","","Financial Expenses","Charges financières"
"group_631","631","","Interest Expenses","Charges d'intérêts"
"group_633","633","","Foreign Exchange Losses","Pertes de change"
"group_638","638","","Other Financial Expenses","Autres charges financières"
"group_639","639","","Financial Allocations","Dotations financières"
"group_65","65","","Non-Current Expenses","Charges non courantes"
"group_651","651","","Net Depreciation Value of Non-Current Assets Sold","Valeurs nettes d'amortissements des immobilisations cédées"
"group_656","656","","Subsidies Granted","Subventions accordées"
"group_658","658","","Other Non-Current Expenses","Autres charges non courantes"
"group_659","659","","Non-Current Allocations","Dotations non courantes"
"group_67","67","","Income Taxes","Impôts sur les résultats"
"group_670","670","","Income Taxes","Impôts sur les résultats"
"group_7","7","","Income Accounts","Comptes de produits"
"group_71","71","","Operating Income","Produits d'exploitation"
"group_711","711","","Sales of Goods","Ventes de marchandises"
"group_712","712","","Sales of Goods and Services Produced","Ventes de biens et services produits"
"group_713","713","","Change in Inventory of Products","Variations des stocks de produits"
"group_714","714","","Non-Current Assets Produced by the Company for its Own Use","Immobilisations produites par l'entreprise pour elle-même"
"group_716","716","","Operating Subsidies","Subventions d'exploitation"
"group_718","718","","Other Operating Income","Autres produits d'exploitation"
"group_719","719","","Operating Reversals; Expense Transfers","Reprises d'exploitation; Transferts de charges"
"group_73","73","","Financial Income","Produits financiers"
"group_732","732","","Income from Equity Securities and Other Non-Current Assets","Produits des titres de participation et des autres titres immobilisés"
"group_733","733","","Foreign Exchange Gains","Gains de change"
"group_738","738","","Interest and Other Financial Income","Intérêts et autres produits financiers"
"group_739","739","","Financial Reversals; Expense Transfers","Reprises financières; Transferts de charges"
"group_75","75","","Non-Current Income","Produits non courants"
"group_751","751","","Income from Disposal of Non-Current Assets","Produits des cessions d'immobilisations"
"group_756","756","","Balancing Subsidies","Subventions d'équilibre"
"group_757","757","","Reversals of Investment Subsidies","Reprises sur subventions d'investissement"
"group_758","758","","Other Non-Current Income","Autres produits non courants"
"group_759","759","","Non-Current Reversals; Expense Transfers","Reprises non courantes; Transferts de charges"

```

## File: data\template\account.tax-ma.csv

```csv
"id","sequence","description","invoice_label","name","amount","amount_type","type_tax_use","tax_group_id","active","tax_exigibility","cash_basis_transition_account_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@fr","invoice_label@fr"
"vat_out_20_80","10","Production and distribution operations","20%","20% 80","20","percent","sale","tax_group_vat_20","","on_payment","pcg_44551","base","invoice","+80B","","","20% 80","Opérations de production et de distribution"
"","","","","","","","","","","","","tax","invoice","+80T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-80B","","","",""
"","","","","","","","","","","","","tax","refund","-80T","pcg_4455","","",""
"vat_out_20_81","20","Services","20%","20% 81","20","percent","sale","tax_group_vat_20","","on_payment","pcg_44551","base","invoice","+81B","","","20% 81","Prestations de services"
"","","","","","","","","","","","","tax","invoice","+81T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-81B","","","",""
"","","","","","","","","","","","","tax","refund","-81T","pcg_4455","","",""
"vat_out_20_82","30","Liberal professions referred to in Article 89-I-12°-(b) of the CGI","20%","20% 82","20","percent","sale","tax_group_vat_20","False","on_payment","pcg_44551","base","invoice","+82B","","","20% 82","Professions libérales visées à l'article 89-I-12°-(b) du CGI"
"","","","","","","","","","","","","tax","invoice","+82T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-82B","","","",""
"","","","","","","","","","","","","tax","refund","-82T","pcg_4455","","",""
"vat_out_20_83","40","Leasing operations","20%","20% 83","20","percent","sale","tax_group_vat_20","False","on_payment","pcg_44551","base","invoice","+83B","","","20% 83","Opérations de crédit-bail "
"","","","","","","","","","","","","tax","invoice","+83T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-83B","","","",""
"","","","","","","","","","","","","tax","refund","-83T","pcg_4455","","",""
"vat_out_20_87","50","Operations of real estate works companies","20%","20% 87","20","percent","sale","tax_group_vat_20","False","on_payment","pcg_44551","base","invoice","+87B","","","20% 87","Opérations d'entreprises de travaux immobiliers"
"","","","","","","","","","","","","tax","invoice","+87T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-87B","","","",""
"","","","","","","","","","","","","tax","refund","-87T","pcg_4455","","",""
"vat_out_20_93","60","Sales and delivery of used movable property","20%","20% 93","20","percent","sale","tax_group_vat_20","False","on_payment","pcg_44551","base","invoice","+93B","","","20% 93","Opérations de vente et de livraison de biens meubles d'occasion"
"","","","","","","","","","","","","tax","invoice","+93T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-93B","","","",""
"","","","","","","","","","","","","tax","refund","-93T","pcg_4455","","",""
"vat_out_20_94","70","Self-supply of constructions","20%","20% 94","20","percent","sale","tax_group_vat_20","False","on_payment","pcg_44551","base","invoice","+94B","","","20% 94","Opérations de livraison à soi-même de constructions"
"","","","","","","","","","","","","tax","invoice","+94T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-94B","","","",""
"","","","","","","","","","","","","tax","refund","-94T","pcg_4455","","",""
"vat_out_20_95","80","Margin regime referred to in Articles 125 bis and 125 quater of the CGI","20%","20% 95","20","percent","sale","tax_group_vat_20","False","on_payment","pcg_44551","base","invoice","+95B","","","20% 95","Régime de marge visé aux articles 125 bis et 125 quater du CGI"
"","","","","","","","","","","","","tax","invoice","+95T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-95B","","","",""
"","","","","","","","","","","","","tax","refund","-95T","pcg_4455","","",""
"vat_out_20_102","90","Other","20%","20% 102","20","percent","sale","tax_group_vat_20","","on_payment","pcg_44551","base","invoice","+102B","","","20% 102","Autres"
"","","","","","","","","","","","","tax","invoice","+102T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-102B","","","",""
"","","","","","","","","","","","","tax","refund","-102T","pcg_4455","","",""
"vat_out_20_129","100","Transactions with non-resident taxpayers","20%","20% EX 129","20","percent","sale","tax_group_vat_20","","on_payment","pcg_44551","base","invoice","+129B","","","20% EX 129","Opérations réalisées avec des contribuables non résidents"
"","","","","","","","","","","","","tax","invoice","+129T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-129T","pcg_4455","","",""
"vat_out_14_84","110","Butter, excluding home-made butter referred to in Article 91 (I-A. 2°)","14%","14% 84","14","percent","sale","tax_group_vat_14","False","on_payment","pcg_44551","base","invoice","+84B","","","14% 84","Beurre, à l'exclusion du beurre de fabrication artisanale visé à l'article 91 (I-A. 2°)"
"","","","","","","","","","","","","tax","invoice","+84T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-84B","","","",""
"","","","","","","","","","","","","tax","refund","-84T","pcg_4455","","",""
"vat_out_14_88","120","Passenger and freight transport operations, excluding rail transport","14%","14% 88","14","percent","sale","tax_group_vat_14","False","on_payment","pcg_44551","base","invoice","+88B","","","14% 88","Opérations de transport de voyageurs et de marchandises, à l'exclusion du transport ferroviaire"
"","","","","","","","","","","","","tax","invoice","+88T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-88B","","","",""
"","","","","","","","","","","","","tax","refund","-88T","pcg_4455","","",""
"vat_out_14_90","130","Electrical energy","14%","14% 90","14","percent","sale","tax_group_vat_14","False","on_payment","pcg_44551","base","invoice","+90B","","","14% 90","Énergie électrique"
"","","","","","","","","","","","","tax","invoice","+90T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-90B","","","",""
"","","","","","","","","","","","","tax","refund","-90T","pcg_4455","","",""
"vat_out_14_104","140","Other","14%","14% 104","14","percent","sale","tax_group_vat_14","False","on_payment","pcg_44551","base","invoice","+104B","","","14% 104","Autres"
"","","","","","","","","","","","","tax","invoice","+104T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-104B","","","",""
"","","","","","","","","","","","","tax","refund","-104T","pcg_4455","","",""
"vat_out_14_91","150","Services rendered by any soliciting agent or insurance broker in respect of contracts brought by him to an insurance company","14%","14% 91","14","percent","sale","tax_group_vat_14","False","on_payment","pcg_44551","base","invoice","+91B","","","14% 91","Prestations de services rendues par tout agentdémarcheur ou courtier d'assurances à raison des contrats apportés par lui à une entreprise d'assurances"
"","","","","","","","","","","","","tax","invoice","+91T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-91B","","","",""
"","","","","","","","","","","","","tax","refund","-91T","pcg_4455","","",""
"vat_out_14_129","160","Transactions with non-resident taxpayers","14%","14% EX 129","14","percent","sale","tax_group_vat_14","False","on_payment","pcg_44551","base","invoice","+129B","","","14% EX 129","Opérations réalisées avec des contribuables non résidents"
"","","","","","","","","","","","","tax","invoice","+129T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-129T","pcg_4455","","",""
"vat_out_10_76","170","Photovoltaic panels","10%","10% 76","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+76B","","","10% 76","Panneaux photovoltaïques"
"","","","","","","","","","","","","tax","invoice","+76T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-76B","","","",""
"","","","","","","","","","","","","tax","refund","-76T","pcg_4455","","",""
"vat_out_10_77","180","Solar water heaters","10%","10% 77","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+77B","","","10% 77","Chauffe-eaux solaires"
"","","","","","","","","","","","","tax","invoice","+77T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-77B","","","",""
"","","","","","","","","","","","","tax","refund","-77T","pcg_4455","","",""
"vat_out_10_78","190","Feed for livestock and farmyard animals as well as oil cakes used in their manufacture, excluding other simple feed","10%","10% 78","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+78B","","","10% 78","Aliments destinés à l'alimentation du bétail et des animaux de basse-cour ainsi que les tourteaux servant à leur fabrication à l'exclusion des autres aliments simples"
"","","","","","","","","","","","","tax","invoice","+78T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-78B","","","",""
"","","","","","","","","","","","","tax","refund","-78T","pcg_4455","","",""
"vat_out_10_79","200","Equipment for exclusive agricultural use","10%","10% 79","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+79B","","","10% 79","Équipements à usage exclusivement agricole"
"","","","","","","","","","","","","tax","invoice","+79T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-79B","","","",""
"","","","","","","","","","","","","tax","refund","-79T","pcg_4455","","",""
"vat_out_10_85","210","Wood in logs, debarked or simply squared, cork in its natural state, firewood in bundles or sawn to short lengths and charcoal","10%","10% 85","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+85B","","","10% 85","Bois en grumes, écorcés ou simplement équarris, le liège à l'état naturel, les bois de feu en fagots ou sciés à petite longueur et le charbon de bois"
"","","","","","","","","","","","","tax","invoice","+85T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-85B","","","",""
"","","","","","","","","","","","","tax","refund","-85T","pcg_4455","","",""
"vat_out_10_86","220","Fishing gear and nets for professional marine fishermen","10%","10% 86","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+86B","","","10% 86","Engins et filets de pêche destinés aux professionnels de la pêche maritime"
"","","","","","","","","","","","","tax","invoice","+86T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-86B","","","",""
"","","","","","","","","","","","","tax","refund","-86T","pcg_4455","","",""
"vat_out_10_89","230","Accommodation and catering operations as well as services provided by cafes","10%","10% 89","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+89B","","","10% 89","Opérations d'hébergement et de restauration ainsi que les prestations fournies par les cafés"
"","","","","","","","","","","","","tax","invoice","+89T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-89B","","","",""
"","","","","","","","","","","","","tax","refund","-89T","pcg_4455","","",""
"vat_out_10_92","240","Rental operations of buildings used as hotels","10%","10% 92","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+92B","","","10% 92","Opérations de location d'immeubles à usage d'hôtels"
"","","","","","","","","","","","","tax","invoice","+92T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-92B","","","",""
"","","","","","","","","","","","","tax","refund","-92T","pcg_4455","","",""
"vat_out_10_96","250","Cooking salt (rock or sea salt)","10%","10% 96","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+96B","","","10% 96","Sel de cuisine (gemme ou marin)"
"","","","","","","","","","","","","tax","invoice","+96T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-96B","","","",""
"","","","","","","","","","","","","tax","refund","-96T","pcg_4455","","",""
"vat_out_10_97","260","Milled rice and pasta","10%","10% 97","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+97B","","","10% 97","Riz usiné et les pâtes alimentaires"
"","","","","","","","","","","","","tax","invoice","+97T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-97B","","","",""
"","","","","","","","","","","","","tax","refund","-97T","pcg_4455","","",""
"vat_out_10_98","270","Food fluid oils excluding palm oil","10%","10% 98","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+98B","","","10% 98","Huiles fluides alimentaires à l'exclusion de l'huile de palme"
"","","","","","","","","","","","","tax","invoice","+98T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-98B","","","",""
"","","","","","","","","","","","","tax","refund","-98T","pcg_4455","","",""
"vat_out_10_99","280","Securities transactions","10%","10% 99","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+99B","","","10% 99","Transactions relatives aux valeurs mobilières"
"","","","","","","","","","","","","tax","invoice","+99T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-99B","","","",""
"","","","","","","","","","","","","tax","refund","-99T","pcg_4455","","",""
"vat_out_10_100","290","Banking and credit transactions and exchange commissions","10%","10% 100","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+100B","","","10% 100","Opérations de banque et de crédit et les commissions de change"
"","","","","","","","","","","","","tax","invoice","+100T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-100B","","","",""
"","","","","","","","","","","","","tax","refund","-100T","pcg_4455","","",""
"vat_out_10_101","300","Transactions involving shares and units shares","10%","10% 101","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+101B","","","10% 101","Transactions portant sur les actions et les parts sociales"
"","","","","","","","","","","","","tax","invoice","+101T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-101B","","","",""
"","","","","","","","","","","","","tax","refund","-101T","pcg_4455","","",""
"vat_out_10_103","310","Operations of sale and delivery of works and objects of art","10%","10% 103","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+103B","","","10% 103","Opérations de vente et de livraison des œuvres et objets d'art"
"","","","","","","","","","","","","tax","invoice","+103T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-103B","","","",""
"","","","","","","","","","","","","tax","refund","-103T","pcg_4455","","",""
"vat_out_10_105","320","Operations of lawyers, interpreters, notaries, adouls, bailiffs and veterinarians","10%","10% 105","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+105B","","","10% 105","Opérations des avocats, interprètes, notaires, adoul, huissiers de justice et vétérinaires"
"","","","","","","","","","","","","tax","invoice","+105T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-105B","","","",""
"","","","","","","","","","","","","tax","refund","-105T","pcg_4455","","",""
"vat_out_10_108","330","Petroleum gas and other gaseous hydrocarbons","10%","10% 108","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+108B","","","10% 108","Gaz de pétrole et les autres hydrocarbures gazeux"
"","","","","","","","","","","","","tax","invoice","+108T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-108B","","","",""
"","","","","","","","","","","","","tax","refund","-108T","pcg_4455","","",""
"vat_out_10_109","340","Petroleum or shale oils, crude or refined","10%","10% 109","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+109B","","","10% 109","Huiles de pétrole ou de schistes, brutes ou raffinées"
"","","","","","","","","","","","","tax","invoice","+109T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-109B","","","",""
"","","","","","","","","","","","","tax","refund","-109T","pcg_4455","","",""
"vat_out_10_112","350","Ticket sales operations for museums, cinema and theater","10%","10% 112","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+112B","","","10% 112","Opérations de vente des billets d'entrée aux musées, cinéma et théâtre"
"","","","","","","","","","","","","tax","invoice","+112T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-112B","","","",""
"","","","","","","","","","","","","tax","refund","-112T","pcg_4455","","",""
"vat_out_10_118","360","Other","10%","10% 118","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+118B","","","10% 118","Autres"
"","","","","","","","","","","","","tax","invoice","+118T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-118B","","","",""
"","","","","","","","","","","","","tax","refund","-118T","pcg_4455","","",""
"vat_out_10_129","370","Transactions with non-resident taxpayers","10%","10% EX 129","10","percent","sale","tax_group_vat_10","False","on_payment","pcg_44551","base","invoice","+129B","","","10% EX 129","Opérations réalisées avec des contribuables non résidents"
"","","","","","","","","","","","","tax","invoice","+129T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-129T","pcg_4455","","",""
"vat_out_7_106","380","Water delivered to the public distribution networks and sanitation services","7%","7% 106","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+106B","","","7% 106","Eau livrée aux réseaux de distribution publique et les prestations d'assainissement"
"","","","","","","","","","","","","tax","invoice","+106T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-106B","","","",""
"","","","","","","","","","","","","tax","refund","-106T","pcg_4455","","",""
"vat_out_7_107","390","Rental of water and electricity meters","7%","7% 107","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+107B","","","7% 107","Location de compteurs d'eau et d'électricité"
"","","","","","","","","","","","","tax","invoice","+107T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-107B","","","",""
"","","","","","","","","","","","","tax","refund","-107T","pcg_4455","","",""
"vat_out_7_110","400","Pharmaceutical products, raw materials and products used in their composition as well as non-returnable packaging of these products and materials","7%","7% 110","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+110B","","","7% 110","Produits pharmaceutiques, matières premières et produits entrant dans leurs compositions ainsi que les emballages non récupérables de ces produits et matières"
"","","","","","","","","","","","","tax","invoice","+110T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-110B","","","",""
"","","","","","","","","","","","","tax","refund","-110T","pcg_4455","","",""
"vat_out_7_111","410","School supplies, products and materials in their composition","7%","7% 111","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+111B","","","7% 111","Fournitures scolaires, produits et matières entrant dans leur composition"
"","","","","","","","","","","","","tax","invoice","+111T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-111B","","","",""
"","","","","","","","","","","","","tax","refund","-111T","pcg_4455","","",""
"vat_out_7_113","420","Refined or agglomerated sugar","7%","7% 113","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+113B","","","7% 113","Sucre raffiné ou aggloméré"
"","","","","","","","","","","","","tax","invoice","+113T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-113B","","","",""
"","","","","","","","","","","","","tax","refund","-113T","pcg_4455","","",""
"vat_out_7_114","430","Canned sardines","7%","7% 114","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+114B","","","7% 114","Conserves de sardines"
"","","","","","","","","","","","","tax","invoice","+114T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-114B","","","",""
"","","","","","","","","","","","","tax","refund","-114T","pcg_4455","","",""
"vat_out_7_115","440","Milk powder","7%","7% 115","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+115B","","","7% 115","Lait en poudre"
"","","","","","","","","","","","","tax","invoice","+115T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-115B","","","",""
"","","","","","","","","","","","","tax","refund","-115T","pcg_4455","","",""
"vat_out_7_116","450","Household soap","7%","7% 116","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+116B","","","7% 116","Savon de ménage"
"","","","","","","","","","","","","tax","invoice","+116T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-116B","","","",""
"","","","","","","","","","","","","tax","refund","-116T","pcg_4455","","",""
"vat_out_7_117","460","Economy car and products and materials used in its in its manufacture","7%","7% 117","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+117B","","","7% 117","Voiture économique et produits et matières entrant dans sa fabrication"
"","","","","","","","","","","","","tax","invoice","+117T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-117B","","","",""
"","","","","","","","","","","","","tax","refund","-117T","pcg_4455","","",""
"vat_out_7_119","470","Other","7%","7% 119","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+119B","","","7% 119","Autres"
"","","","","","","","","","","","","tax","invoice","+119T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-119B","","","",""
"","","","","","","","","","","","","tax","refund","-119T","pcg_4455","","",""
"vat_out_7_129","480","Transactions with non-resident taxpayers","7%","7% EX 129","7","percent","sale","tax_group_vat_7","False","on_payment","pcg_44551","base","invoice","+129B","","","7% EX 129","Opérations réalisées avec des contribuables non résidents"
"","","","","","","","","","","","","tax","invoice","+129T","pcg_4455","","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-129T","pcg_4455","","",""
"vat_out_0_20","490","Operations outside the scope of VAT","0%","0% 20","0","percent","sale","tax_group_vat_0","False","","","base","invoice","+20","","","0% 20","Opérations situées hors champ d'application de la TVA"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-20","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_out_0_30","500","Exempt operations without right of deduction (Article 91 of the CGI)","0%","0% 30","0","percent","sale","tax_group_vat_0","False","","","base","invoice","+30","","","0% 30","Opérations exonérées sans droit à déduction (article 91 du CGI)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-30","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_out_0_40","510","Exempt operations with right of deduction (Article 92 of the CGI)","0%","0% 40","0","percent","sale","tax_group_vat_0","False","","","base","invoice","+40","","","0% 40","Opérations exonérées avec droit à déduction (article 92 du CGI)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-40","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_out_0_50","520","Operations carried out under suspension of VAT (Article 94 of the CGI)","0%","0% 50","0","percent","sale","tax_group_vat_0","False","","","base","invoice","+50","","","0% 50","Opérations réalisées en suspension de la TVA (article 94 du CGI)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-50","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_in_20_146","530","Domestic purchases other than fixed assets","20%","20% 146","20","percent","purchase","tax_group_vat_20","","on_payment","pcg_34554","base","invoice","","","","20% 146","Achats non immobilisés à l'interieur"
"","","","","","","","","","","","","tax","invoice","+146T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-146T","pcg_34552","","",""
"vat_in_20_140","540","Services","20%","20% S 140","20","percent","purchase","tax_group_vat_20","","on_payment","pcg_34554","base","invoice","","","","20% S 140","Prestations de services"
"","","","","","","","","","","","","tax","invoice","+140T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-140T","pcg_34552","","",""
"vat_in_20_155","550","Contract work","20%","20% S 155","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34554","base","invoice","","","","20% S 155","Travaux à façon"
"","","","","","","","","","","","","tax","invoice","+155T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-155T","pcg_34552","","",""
"vat_in_20_156","560","Subcontracting (real estate work)","20%","20% S 156","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34554","base","invoice","","","","20% S 156","Sous–traitance (travaux immobiliers)"
"","","","","","","","","","","","","tax","invoice","+156T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-156T","pcg_34552","","",""
"vat_in_20_145","570","Import purchases other than fixed assets","20%","20% EX 145","20","percent","purchase","tax_group_vat_20","","on_payment","pcg_34554","base","invoice","+129B","","","20% EX 145","Achats non immobilisés à l'importation"
"","","","","","","","","","","","","tax","invoice","+145T","pcg_34552","","",""
"","","","","","","","","","","","","tax","invoice","-129T","pcg_4455","-100","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-145T","pcg_34552","","",""
"","","","","","","","","","","","","tax","refund","+129T","pcg_4455","-100","",""
"vat_in_20_163","580","Domestic purchases of fixed assets","20%","20% IG 163","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34553","base","invoice","","","","20% IG 163","Achats d'immobilisations à l'interieur"
"","","","","","","","","","","","","tax","invoice","+163T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-163T","pcg_34551","","",""
"vat_in_20_162","590","Import purchases of fixed assets","20%","20% EX IG 162","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34553","base","invoice","+129B","","","20% EX IG 162","Achats d'immobilisations à l'importation"
"","","","","","","","","","","","","tax","invoice","+162T","pcg_34551","","",""
"","","","","","","","","","","","","tax","invoice","-129T","pcg_4455","-100","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-162T","pcg_34551","","",""
"","","","","","","","","","","","","tax","refund","+129T","pcg_4455","-100","",""
"vat_in_20_164","600","Self-supply other than constructions","20%","20% IG 164","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34553","base","invoice","","","","20% IG 164","Livraisons à soi-même autre que les constructions"
"","","","","","","","","","","","","tax","invoice","+164T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-164T","pcg_34551","","",""
"vat_in_20_165","610","Facilities and installations","20%","20% IG 165","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34553","base","invoice","","","","20% IG 165","Installations et poses"
"","","","","","","","","","","","","tax","invoice","+165T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-165T","pcg_34551","","",""
"vat_in_20_166","620","Constructions","20%","20% IG 166","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34553","base","invoice","","","","20% IG 166","Constructions"
"","","","","","","","","","","","","tax","invoice","+166T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-166T","pcg_34551","","",""
"vat_in_20_167","630","Self-supply of constructions","20%","20% IG 167","20","percent","purchase","tax_group_vat_20","False","on_payment","pcg_34553","base","invoice","","","","20% IG 167","Livraison à soi-même de constructions"
"","","","","","","","","","","","","tax","invoice","+167T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-167T","pcg_34551","","",""
"vat_in_14_148","640","Domestic purchases other than fixed assets","14%","14% 148","14","percent","purchase","tax_group_vat_14","","on_payment","pcg_34554","base","invoice","","","","14% 148","Achats non immobilisés à l'interieur"
"","","","","","","","","","","","","tax","invoice","+148T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-148T","pcg_34552","","",""
"vat_in_14_141","650","Transportation","14%","14% S 141","14","percent","purchase","tax_group_vat_14","False","on_payment","pcg_34554","base","invoice","","","","14% S 141","Transport"
"","","","","","","","","","","","","tax","invoice","+141T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-141T","pcg_34552","","",""
"vat_in_14_147","660","Import purchases other than fixed assets","14%","14% EX 147","14","percent","purchase","tax_group_vat_14","False","on_payment","pcg_34554","base","invoice","+129B","","","14% EX 147","Achats non immobilisés à l'importation"
"","","","","","","","","","","","","tax","invoice","+147T","pcg_34552","","",""
"","","","","","","","","","","","","tax","invoice","-129T","pcg_4455","-100","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-147T","pcg_34552","","",""
"","","","","","","","","","","","","tax","refund","+129T","pcg_4455","-100","",""
"vat_in_14_168","670","Other fixed assets","14%","14% IG 168","14","percent","purchase","tax_group_vat_14","False","on_payment","pcg_34553","base","invoice","","","","14% IG 168","Autres immobilisations"
"","","","","","","","","","","","","tax","invoice","+168T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-168T","pcg_34551","","",""
"vat_in_10_150","680","Domestic purchases other than fixed assets","10%","10% 150","10","percent","purchase","tax_group_vat_10","","on_payment","pcg_34554","base","invoice","","","","10% 150","Achats non immobilisés à l'interieur"
"","","","","","","","","","","","","tax","invoice","+150T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-150T","pcg_34552","","",""
"vat_in_10_142","690","Banking operation","10%","10% S 142","10","percent","purchase","tax_group_vat_10","False","on_payment","pcg_34554","base","invoice","","","","10% S 142","Opération de banque"
"","","","","","","","","","","","","tax","invoice","+142T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-142T","pcg_34552","","",""
"vat_in_10_143","700","Hotels for travelers, and all real estate for tourism purposes","10%","10% S 143","10","percent","purchase","tax_group_vat_10","False","on_payment","pcg_34554","base","invoice","","","","10% S 143","Hôtels de voyageurs, et ensemble immobilier à destination touristique"
"","","","","","","","","","","","","tax","invoice","+143T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-143T","pcg_34552","","",""
"vat_in_10_144","710","Operations performed by lawyers, interpreters, notaries, adouls, bailiffs and veterinarians","10%","10% S 144","10","percent","purchase","tax_group_vat_10","False","on_payment","pcg_34554","base","invoice","","","","10% S 144","Opérations réalisées par les avocats, interprètes, notaires, adoul, huissiers de justice et vétérinaires"
"","","","","","","","","","","","","tax","invoice","+144T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-144T","pcg_34552","","",""
"vat_in_10_153","720","Other services","10%","10% S 153","10","percent","purchase","tax_group_vat_10","False","on_payment","pcg_34554","base","invoice","","","","10% S 153","Autres prestations de services"
"","","","","","","","","","","","","tax","invoice","+153T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-153T","pcg_34552","","",""
"vat_in_10_149","730","Import purchases other than fixed assets","10%","10% EX 149","10","percent","purchase","tax_group_vat_10","False","on_payment","pcg_34554","base","invoice","+129B","","","10% EX 149","Achats non immobilisés à l'importation"
"","","","","","","","","","","","","tax","invoice","+149T","pcg_34552","","",""
"","","","","","","","","","","","","tax","invoice","-129T","pcg_4455","-100","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-149T","pcg_34552","","",""
"","","","","","","","","","","","","tax","refund","+129T","pcg_4455","-100","",""
"vat_in_10_160","740","Other fixed assets","10%","10% IG 160","10","percent","purchase","tax_group_vat_10","False","on_payment","pcg_34553","base","invoice","","","","10% IG 160","Autres immobilisations"
"","","","","","","","","","","","","tax","invoice","+160T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-160T","pcg_34551","","",""
"vat_in_7_152","750","Domestic purchases other than fixed assets","7%","7% 152","7","percent","purchase","tax_group_vat_7","","on_payment","pcg_34554","base","invoice","","","","7% 152","Achats non immobilisés à l'interieur"
"","","","","","","","","","","","","tax","invoice","+152T","pcg_34552","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-152T","pcg_34552","","",""
"vat_in_7_151","760","Import purchases other than fixed assets","7%","7% EX 151","7","percent","purchase","tax_group_vat_7","False","on_payment","pcg_34554","base","invoice","+129B","","","7% EX 151","Achats non immobilisés à l'importation"
"","","","","","","","","","","","","tax","invoice","+151T","pcg_34552","","",""
"","","","","","","","","","","","","tax","invoice","-129T","pcg_4455","-100","",""
"","","","","","","","","","","","","base","refund","-129B","","","",""
"","","","","","","","","","","","","tax","refund","-151T","pcg_34552","","",""
"","","","","","","","","","","","","tax","refund","+129T","pcg_4455","-100","",""
"vat_in_7_169","770","Other fixed assets","7%","7% IG 169","7","percent","purchase","tax_group_vat_7","False","on_payment","pcg_34553","base","invoice","","","","7% IG 169","Autres immobilisations"
"","","","","","","","","","","","","tax","invoice","+169T","pcg_34551","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-169T","pcg_34551","","",""

```

## File: data\template\account.tax.group-ma.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@fr"
"tax_group_vat_20","VAT 20%","base.ma","pcg_3456","pcg_4456","TVA 20%"
"tax_group_vat_14","VAT 14%","base.ma","pcg_3456","pcg_4456","TVA 14%"
"tax_group_vat_10","VAT 10%","base.ma","pcg_3456","pcg_4456","TVA 10%"
"tax_group_vat_7","VAT 7%","base.ma","pcg_3456","pcg_4456","TVA 7%"
"tax_group_vat_0","VAT 0%","base.ma","pcg_3456","pcg_4456","TVA 0%"

```

## File: models\base_document_layout.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from markupsafe import Markup

from odoo import api, fields, models


class BaseDocumentLayout(models.TransientModel):
    _inherit = 'base.document.layout'

    @api.model
    def _default_company_details(self):
        # OVERRIDE web/models/base_document_layout
        company_details = super()._default_company_details()
        if self.env.company.country_code == 'MA':
            company_details += Markup('<br> ICE: %s') % self.env.company.company_registry
        return company_details

    company_details = fields.Html(default=_default_company_details)

```

## File: models\res_partner.py

```python
from odoo import api, models, _
from odoo.exceptions import ValidationError


class ResPartner(models.Model):
    _inherit = ['res.partner']

    @api.constrains('company_registry')
    def _check_company_registry_ma(self):
        for record in self:
            if record.country_code == 'MA' and record.company_registry and (len(record.company_registry) != 15 or not record.company_registry.isdigit()):
                raise ValidationError(_("ICE number should have exactly 15 digits."))

    def _get_company_registry_labels(self):
        labels = super()._get_company_registry_labels()
        labels['MA'] = _("ICE")
        return labels

```

## File: models\template_ma.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ma')
    def _get_ma_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'pcg_34211',
            'property_account_payable_id': 'pcg_44111',
            'property_account_income_categ_id': 'pcg_7111',
            'property_account_expense_categ_id': 'pcg_6111',
            'display_invoice_amount_total_words': True,
        }

    @template('ma', 'res.company')
    def _get_ma_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ma',
                'bank_account_code_prefix': '5141',
                'cash_account_code_prefix': '51611',
                'transfer_account_code_prefix': '5115',
                'account_default_pos_receivable_account_id': 'pcg_34218',
                'income_currency_exchange_account_id': 'pcg_7331',
                'expense_currency_exchange_account_id': 'pcg_6331',
                'account_journal_suspense_account_id': 'pcg_3497',
                'default_cash_difference_income_account_id': 'pcg_73861',
                'default_cash_difference_expense_account_id': 'pcg_63861',
                'account_journal_early_pay_discount_gain_account_id': 'pcg_73862',
                'account_journal_early_pay_discount_loss_account_id': 'pcg_63862',
                'account_sale_tax_id': 'vat_out_20_80',
                'account_purchase_tax_id': 'vat_in_20_146',
                'tax_exigibility': 'True',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import base_document_layout
from . import template_ma
from . import res_partner

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='partner_vat_address_same_as_shipping']" position="after">
            <div t-if="o.partner_id.country_code == 'MA' and o.partner_id.company_registry">
                ICE: <a t-field="o.partner_id.company_registry"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_company_form" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_ma</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="account.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='company_registry']" position="attributes">
                <attribute name="nolabel">1</attribute>
            </xpath>
            <field name="company_registry" position="before">
                <label for="company_registry" invisible="country_code == 'MA'" />
                <label for="company_registry" string="ICE" invisible="country_code != 'MA'" />
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_property_form" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_ma</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="company_registry" string="ICE" invisible="country_code != 'MA'"></field>
            </xpath>
        </field>
    </record>
</odoo>

```

