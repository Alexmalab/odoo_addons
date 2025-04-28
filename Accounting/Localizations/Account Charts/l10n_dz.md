# Odoo Module: l10n_dz

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
    'name': 'Algeria - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['dz'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the accounting chart for Algeria in Odoo.
======================================================================
This module applies to companies based in Algeria.
""",
    'author': 'Osis',
    'depends': [
        'base_vat',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/tax_report.xml',
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
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.dz"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_total_turnover" model="account.report.column">
                <field name="name">Total turnover</field>
                <field name="expression_label">total_turnover</field>
            </record>
            <record id="tax_report_exempt_turnover" model="account.report.column">
                <field name="name">Exempt turnover</field>
                <field name="expression_label">exempt_turnover</field>
            </record>
            <record id="tax_report_taxable_turnover" model="account.report.column">
                <field name="name">Taxable turnover</field>
                <field name="expression_label">taxable_turnover</field>
            </record>
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Fee amount</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_dz_tax_report_1" model="account.report.line">
                <field name="name">VALUE ADDED TAX</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_dz_tax_report_1_1" model="account.report.line">
                        <field name="name">A / Taxable turnover</field>
                        <field name="hierarchy_level">1</field>
                        <field name="code">l10n_dz_tr_overall_turnover</field>
                        <field name="expression_ids">
                            <record id="l10n_dz_tax_report_overall_turnover_total_turnover_tag" model="account.report.expression">
                                <field name="label">total_turnover</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_dz_tr_overall_turnover.exempt_turnover + l10n_dz_tr_overall_turnover.taxable_turnover</field>
                            </record>
                            <record id="l10n_dz_tax_report_overall_turnover_exempt_turnover_tag" model="account.report.expression">
                                <field name="label">exempt_turnover</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_dz_E3B11.exempt_turnover +
                                                      l10n_dz_E3B12.exempt_turnover +
                                                      l10n_dz_E3B13.exempt_turnover +
                                                      l10n_dz_E3B14.exempt_turnover +
                                                      l10n_dz_E3B15.exempt_turnover +
                                                      l10n_dz_E3B16.exempt_turnover +
                                                      l10n_dz_E3B21.exempt_turnover +
                                                      l10n_dz_E3B22.exempt_turnover +
                                                      l10n_dz_E3B23.exempt_turnover +
                                                      l10n_dz_E3B24.exempt_turnover +
                                                      l10n_dz_E3B25.exempt_turnover +
                                                      l10n_dz_E3B26.exempt_turnover +
                                                      l10n_dz_E3B28.exempt_turnover +
                                                      l10n_dz_E3B31.exempt_turnover +
                                                      l10n_dz_E3B32.exempt_turnover +
                                                      l10n_dz_E3B33.exempt_turnover +
                                                      l10n_dz_E3B34.exempt_turnover +
                                                      l10n_dz_E3B35.exempt_turnover +
                                                      l10n_dz_E3B36.exempt_turnover +
                                                      l10n_dz_E3B37.exempt_turnover</field>
                            </record>
                            <record id="l10n_dz_tax_report_overall_turnover_taxable_turnover_tag" model="account.report.expression">
                                <field name="label">taxable_turnover</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_dz_E3B11.taxable_turnover +
                                                      l10n_dz_E3B12.taxable_turnover +
                                                      l10n_dz_E3B13.taxable_turnover +
                                                      l10n_dz_E3B14.taxable_turnover +
                                                      l10n_dz_E3B15.taxable_turnover +
                                                      l10n_dz_E3B16.taxable_turnover +
                                                      l10n_dz_E3B21.taxable_turnover +
                                                      l10n_dz_E3B22.taxable_turnover +
                                                      l10n_dz_E3B23.taxable_turnover +
                                                      l10n_dz_E3B24.taxable_turnover +
                                                      l10n_dz_E3B25.taxable_turnover +
                                                      l10n_dz_E3B26.taxable_turnover +
                                                      l10n_dz_E3B28.taxable_turnover +
                                                      l10n_dz_E3B31.taxable_turnover +
                                                      l10n_dz_E3B32.taxable_turnover +
                                                      l10n_dz_E3B33.taxable_turnover +
                                                      l10n_dz_E3B34.taxable_turnover +
                                                      l10n_dz_E3B35.taxable_turnover +
                                                      l10n_dz_E3B36.taxable_turnover +
                                                      l10n_dz_E3B37.taxable_turnover</field>
                            </record>
                            <record id="l10n_dz_tax_report_overall_turnover_balance_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_dz_E3B11.balance +
                                                      l10n_dz_E3B12.balance +
                                                      l10n_dz_E3B13.balance +
                                                      l10n_dz_E3B14.balance +
                                                      l10n_dz_E3B15.balance +
                                                      l10n_dz_E3B16.balance +
                                                      l10n_dz_E3B21.balance +
                                                      l10n_dz_E3B22.balance +
                                                      l10n_dz_E3B23.balance +
                                                      l10n_dz_E3B24.balance +
                                                      l10n_dz_E3B25.balance +
                                                      l10n_dz_E3B26.balance +
                                                      l10n_dz_E3B28.balance +
                                                      l10n_dz_E3B31.balance +
                                                      l10n_dz_E3B32.balance +
                                                      l10n_dz_E3B33.balance +
                                                      l10n_dz_E3B34.balance +
                                                      l10n_dz_E3B35.balance +
                                                      l10n_dz_E3B36.balance +
                                                      l10n_dz_E3B37.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="l10n_dz_tax_report_1_1_1" model="account.report.line">
                                <field name="name">1) Operations subject to VAT (9%)</field>
                                <field name="children_ids">
                                    <record id="l10n_dz_tax_report_1_1_1_1" model="account.report.line">
                                        <field name="name">E3B11 - Goods, Products and Commodities Covered by Section 23 of the CTCA</field>
                                        <field name="code">l10n_dz_E3B11</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B11_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B11.exempt_turnover + l10n_dz_E3B11.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B11_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B11_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B11_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B11_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B11_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B11_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_1_2" model="account.report.line">
                                        <field name="name">E3B12 - Services covered by Article 23 of the CTCA</field>
                                        <field name="code">l10n_dz_E3B12</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B12_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B12.exempt_turnover + l10n_dz_E3B12.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B12_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B12_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B12_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B12_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B12_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B12_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_1_3" model="account.report.line">
                                        <field name="name">E3B13 - Real Estate Operations under Section 23 of the CTCA</field>
                                        <field name="code">l10n_dz_E3B13</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B13_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B13.exempt_turnover + l10n_dz_E3B13.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B13_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B13_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B13_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B13_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B13_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B13_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_1_4" model="account.report.line">
                                        <field name="name">E3B14 - Medical Procedures</field>
                                        <field name="code">l10n_dz_E3B14</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B14_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B14.exempt_turnover + l10n_dz_E3B14.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B14_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B14_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B14_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B14_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B14_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B14_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_1_5" model="account.report.line">
                                        <field name="name">E3B15 - Commissionaires and Brokers</field>
                                        <field name="code">l10n_dz_E3B15</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B15_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B15.exempt_turnover + l10n_dz_E3B15.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B15_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B15_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B15_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B15_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B15_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B15_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_1_6" model="account.report.line">
                                        <field name="name">E3B16 - Energy Supply</field>
                                        <field name="code">l10n_dz_E3B16</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B16_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B16.exempt_turnover + l10n_dz_E3B16.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B16_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B16_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B16_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B16_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B16_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B16_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_1_1_2" model="account.report.line">
                                <field name="name">2) Operations subject to VAT (19%)</field>
                                <field name="children_ids">
                                    <record id="l10n_dz_tax_report_1_1_2_1" model="account.report.line">
                                        <field name="name">E3B21 - Productions: goods, products and commodities covered by article 21 of the CTCA</field>
                                        <field name="code">l10n_dz_E3B21</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B21_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B21.exempt_turnover + l10n_dz_E3B21.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B21_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B21_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B21_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B21_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B21_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B21_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_2" model="account.report.line">
                                        <field name="name">E3B22 - Resale as is: goods, products and commodities covered by article 21 of the CTCA</field>
                                        <field name="code">l10n_dz_E3B22</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B22_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B22.exempt_turnover + l10n_dz_E3B22.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B22_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B22_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B22_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B22_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B22_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B22_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_3" model="account.report.line">
                                        <field name="name">E3B23 - Real estate work other than that subject to the 9% rate</field>
                                        <field name="code">l10n_dz_E3B23</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B23_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B23.exempt_turnover + l10n_dz_E3B23.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B23_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B23_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B23_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B23_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B23_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B23_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_4" model="account.report.line">
                                        <field name="name">E3B24 - Liberal Professions</field>
                                        <field name="code">l10n_dz_E3B24</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B24_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B24.exempt_turnover + l10n_dz_E3B24.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B24_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B24_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B24_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B24_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B24_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B24_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_5" model="account.report.line">
                                        <field name="name">E3B25 - Banking and Insurance</field>
                                        <field name="code">l10n_dz_E3B25</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B25_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B25.exempt_turnover + l10n_dz_E3B25.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B25_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B25_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B25_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B25_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B25_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B25_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_6" model="account.report.line">
                                        <field name="name">E3B26 - Telephone and Telex Services</field>
                                        <field name="code">l10n_dz_E3B26</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B26_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B26.exempt_turnover + l10n_dz_E3B26.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B26_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B26_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B26_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B26_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B26_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B26_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_7" model="account.report.line">
                                        <field name="name">E3B28 - Other Services</field>
                                        <field name="code">l10n_dz_E3B28</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B28_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B28.exempt_turnover + l10n_dz_E3B28.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B28_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B28_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B28_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B28_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B28_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B28_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_8" model="account.report.line">
                                        <field name="name">E3B31 - Drinking Places</field>
                                        <field name="code">l10n_dz_E3B31</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B31_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B31.exempt_turnover + l10n_dz_E3B31.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B31_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B31_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B31_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B31_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B31_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B31_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_9" model="account.report.line">
                                        <field name="name">E3B32 - Productions: goods, products and commodities covered by article 21 of the C. TCA</field>
                                        <field name="code">l10n_dz_E3B32</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B32_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B32.exempt_turnover + l10n_dz_E3B32.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B32_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B32_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B32_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B32_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B32_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B32_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_11" model="account.report.line">
                                        <field name="name">E3B33 - Resale as is: goods, products and commodities covered by art. 21 of the C. TCA</field>
                                        <field name="code">l10n_dz_E3B33</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B33_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B33.exempt_turnover + l10n_dz_E3B33.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B33_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B33_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B33_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B33_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B33_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B33_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_12" model="account.report.line">
                                        <field name="name">E3B34 - Tobacco and Matches</field>
                                        <field name="code">l10n_dz_E3B34</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B34_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B34.exempt_turnover + l10n_dz_E3B34.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B34_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B34_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B34_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B34_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B34_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B34_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_13" model="account.report.line">
                                        <field name="name">E3B35 - Shows, games and amusements other than those referred to in art. 21 of the C. TCA</field>
                                        <field name="code">l10n_dz_E3B35</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B35_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B35.exempt_turnover + l10n_dz_E3B35.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B35_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B35_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B35_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B35_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B35_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B35_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_14" model="account.report.line">
                                        <field name="name">E3B36 - Other services referred to in article 21 of the C. TCA</field>
                                        <field name="code">l10n_dz_E3B36</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B36_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B36.exempt_turnover + l10n_dz_E3B36.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B36_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B36_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B36_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B36_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B36_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B36_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_1_2_15" model="account.report.line">
                                        <field name="name">E3B37 - On-site Consumption</field>
                                        <field name="code">l10n_dz_E3B37</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B37_total_turnover_tag" model="account.report.expression">
                                                <field name="label">total_turnover</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_E3B37.exempt_turnover + l10n_dz_E3B37.taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B37_exempt_turnover_tag" model="account.report.expression">
                                                <field name="label">exempt_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B37_exempt_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B37_taxable_turnover_tag" model="account.report.expression">
                                                <field name="label">taxable_turnover</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B37_taxable_turnover</field>
                                            </record>
                                            <record id="l10n_dz_tax_report_E3B37_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">E3B37_balance</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_dz_tax_report_1_2" model="account.report.line">
                        <field name="name">B / Deductions to be made</field>
                        <field name="hierarchy_level">1</field>
                        <field name="code">l10n_dz_tr_recoverable_vat</field>
                        <field name="expression_ids">
                            <record id="l10n_dz_tax_report_recoverable_vat_balance_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_dz_tr_E3B91.balance +
                                                      l10n_dz_tr_E3B92.balance +
                                                      l10n_dz_tr_E3B93.balance +
                                                      l10n_dz_tr_E3B94.balance +
                                                      l10n_dz_tr_E3B95.balance +
                                                      l10n_dz_tr_E3B96.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="l10n_dz_tax_report_E3B91" model="account.report.line">
                                <field name="name">E3B91 - Previous deductions (previous month)</field>
                                <field name="code">l10n_dz_tr_E3B91</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_E3B91_balance_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_E3B92" model="account.report.line">
                                <field name="name">E3B92 - VAT on purchases of goods, materials and services (Art. 29 C. TCA)</field>
                                <field name="code">l10n_dz_tr_E3B92</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_E3B92_balance_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E3B92_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_E3B93" model="account.report.line">
                                <field name="name">E3B93 - VAT on purchases of depreciable goods (Art. 38 C. TCA)</field>
                                <field name="code">l10n_dz_tr_E3B93</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_E3B93_balance_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E3B93_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_E3B94" model="account.report.line">
                                <field name="name">E3B94 - Adjustment of the pro rata (additional deduction) (art. 40 C. TCA)</field>
                                <field name="code">l10n_dz_tr_E3B94</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_E3B94_balance_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_E3B95" model="account.report.line">
                                <field name="name">E3B95 - VAT to be recovered on cancelled or unpaid invoices (Art. 18 C. TCA)</field>
                                <field name="code">l10n_dz_tr_E3B95</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_E3B95_balance_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_E3B96" model="account.report.line">
                                <field name="name">E3B96 - Other deductions (notification of withholding tax, etc. ....)</field>
                                <field name="code">l10n_dz_tr_E3B96</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_E3B96_balance_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_dz_tax_report_1_3" model="account.report.line">
                        <field name="name">C / VAT to be paid</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="l10n_dz_tax_report_1_3_1" model="account.report.line">
                                <field name="name">Total to be recalled (C)</field>
                                <field name="code">l10n_dz_tr_vat_to_be_reported</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_total_to_be_reported_balance_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_dz_tr_total_fees_due.balance + l10n_dz_tr_E3B97.balance + l10n_dz_tr_E3B98.balance</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="l10n_dz_tax_report_1_3_1_1" model="account.report.line">
                                        <field name="name">Total fees due</field>
                                        <field name="code">l10n_dz_tr_total_fees_due</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_total_fees_due_balance_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">l10n_dz_tr_overall_turnover.balance</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_3_1_2" model="account.report.line">
                                        <field name="name">E3B97 - Adjustment of the pro rata (art. 40 C. TCA) (+) (excess deduction)</field>
                                        <field name="code">l10n_dz_tr_E3B97</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B97_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=0</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_dz_tax_report_1_3_1_3" model="account.report.line">
                                        <field name="name">E3B98 - Reversal of the deduction (art. 38 C. TCA)(+)</field>
                                        <field name="code">l10n_dz_tr_E3B98</field>
                                        <field name="expression_ids">
                                            <record id="l10n_dz_tax_report_E3B98_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=0</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_1_3_2" model="account.report.line">
                                <field name="name">Total deductions to be made (B) (-)</field>
                                <field name="code">l10n_dz_tr_vat_to_pay_deduction</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_vat_to_pay_deduction_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_dz_tr_recoverable_vat.balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_1_3_3" model="account.report.line">
                                <field name="name">E3B00 - VAT payable for the month (C - B)</field>
                                <field name="code">l10n_dz_tr_vat_to_pay_total_E3B00</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_vat_to_pay_total_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_dz_tr_vat_to_be_reported.balance - l10n_dz_tr_vat_to_pay_deduction.balance</field>
                                        <field name="subformula">if_above(DZD(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_dz_tax_report_1_3_4" model="account.report.line">
                                <field name="name">E3B99 - Deduction to be carried forward to the next month (B-C)</field>
                                <field name="code">l10n_dz_tr_vat_to_carry_forward_E3B99</field>
                                <field name="expression_ids">
                                    <record id="l10n_dz_tax_report_vat_to_carry_forward_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_dz_tr_vat_to_pay_deduction.balance - l10n_dz_tr_vat_to_be_reported.balance</field>
                                        <field name="subformula">if_above(DZD(0))</field>
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

## File: data\template\account.account-dz.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@fr"
"l10n_dz_101","Issued capital or share capital or endowment fund, or operating fund",101,equity,,False,"Capital émis ou capital social ou fonds de dotation, ou fonds d’exploitation"
l10n_dz_103,Premiums related to share capital,103,equity,,False,Primes liées au capital social
l10n_dz_104,Fair value adjustments,104,equity,,False,Ecart d’évaluation
l10n_dz_105,Revaluation difference,105,equity,,False,Ecart de réévaluation
l10n_dz_106,"Reserves (legal, statutory, ordinary, regulated)",106,equity,,False,"Réserves (légale, statutaire, ordinaire, réglementée)"
l10n_dz_107,Equity differences,107,equity,,False,Écarts d'équivalence
l10n_dz_108,Operator's account,108,equity,,False,Compte de l'exploitant
l10n_dz_109,Shareholders: subscribed capital - uncalled,109,equity,,False,Actionnaires: capital souscrit - non appelé
l10n_dz_11,Retained earnings,11,equity,,False,Report à nouveau
l10n_dz_12,Profit or loss for the year,12,equity_unaffected,,False,Résultat de l’exercice
l10n_dz_131,Equipment grants,131,equity,,False,Subventions d’équipements
l10n_dz_132,Other investment grants,132,equity,,False,Autres subventions d’investissement
l10n_dz_133,Deferred tax assets,133,asset_current,,False,Impôts différés actif
l10n_dz_134,Deferred tax liabilities,134,liability_current,,False,Impôts différés passif
l10n_dz_138,Other deferred income and expenses,138,liability_current,,False,Autres produits et charges différés
l10n_dz_153,Provisions for pensions and similar obligations,153,liability_non_current,,False,Provisions pour pensions et obligations similaires
l10n_dz_155,Provisions for taxes,155,liability_non_current,,False,Provisions pour impôts
l10n_dz_156,Provisions for renewal of fixed assets (concession),156,liability_non_current,,False,Provisions pour renouvellement des immobilisations (concession)
l10n_dz_158,Other provisions for charges - non-current liabilities,158,liability_non_current,,False,Autres provisions pour charges - passifs non courants
l10n_dz_161,Equity securities,161,liability_current,,False,Titres participatifs
l10n_dz_162,Convertible bonds,162,liability_current,,False,Emprunts obligataires convertibles
l10n_dz_163,Other bonds,163,liability_current,,False,Autres emprunts obligataires
l10n_dz_164,Borrowings from credit institutions,164,liability_current,,False,Emprunts auprès des Établissements de crédit
l10n_dz_165,Deposits and guarantees received,165,liability_current,,False,Dépôts et cautionnements reçus
l10n_dz_167,Liabilities under finance leases,167,liability_current,,False,Dettes sur contrat de location-financement
l10n_dz_168,Other loans and similar debts,168,liability_current,,False,Autres emprunts et dettes assimilés
l10n_dz_169,Bond redemption premiums,169,liability_current,,False,Primes de remboursement des obligations
l10n_dz_171,Liabilities related to Group investments,171,liability_current,,False,Dettes rattachées à des participations groupe
l10n_dz_172,Payables to non-group companies,172,liability_current,,False,Dettes rattachées à des participations hors groupe
l10n_dz_173,Payables to joint ventures,173,liability_current,,False,Dettes rattachées à des sociétés en participation
l10n_dz_178,Other liabilities related to equity investments,178,liability_current,,False,Autres dettes rattachées à des participations
l10n_dz_181,Inter-institutional liaison accounts,181,liability_current,,False,Comptes de liaison entre établissements
l10n_dz_188,Liaison accounts between joint ventures,188,liability_current,,False,Comptes de liaison entre sociétés en participation
l10n_dz_203,Capitalizable development costs,203,asset_fixed,,False,Frais de développement immobilisables
l10n_dz_204,Computer software and similar,204,asset_fixed,,False,Logiciels informatiques et assimilés
l10n_dz_205,"Concessions and similar rights, patents, licences, trademarks",205,asset_fixed,,False,"Concessions et droits similaires, brevets, licences, marques"
l10n_dz_207,Goodwill,207,asset_fixed,,False,Écart d’acquisition
l10n_dz_208,Other intangible assets,208,asset_fixed,,False,Autres immobilisations incorporelles
l10n_dz_211,Land,211,asset_fixed,,False,Terrains
l10n_dz_212,Layouts and landscaping,212,asset_fixed,,False,Agencements et aménagements de terrain
l10n_dz_213,Buildings,213,asset_fixed,,False,Constructions
l10n_dz_215,"Technical installations, equipment and industrial tools",215,asset_fixed,,False,"Installations techniques, matériel et outillage industriels"
l10n_dz_218,Other tangible assets,218,asset_fixed,,False,Autres immobilisations corporelles
l10n_dz_221,Land under concession,221,asset_fixed,,False,Terrains en concession
l10n_dz_222,Fixtures and fittings under concession,222,asset_fixed,,False,Agencements et aménagements de terrain en concession
l10n_dz_223,Buildings under concession,223,asset_fixed,,False,Constructions en concession
l10n_dz_225,Technical installations under concession,225,asset_fixed,,False,Installations techniques en concession
l10n_dz_228,Other tangible assets under concession,228,asset_fixed,,False,Autres immobilisations corporelles en concession
l10n_dz_229,Grantor's rights,229,liability_non_current,,False,Droits du concédant
l10n_dz_232,"Property, plant and equipment in progress",232,asset_fixed,,False,Immobilisations corporelles en cours
l10n_dz_237,Intangible assets in progress,237,asset_fixed,,False,Immobilisations incorporelles en cours
l10n_dz_238,Advances and deposits paid on orders for fixed assets,238,asset_fixed,,False,Avances et acomptes versés sur commandes d'immobilisations
l10n_dz_261,Shares in subsidiaries,261,asset_fixed,,False,Titres de filiales
l10n_dz_262,Other investments,262,asset_fixed,,False,Autres titres de participation
l10n_dz_265,Investments in associates,265,asset_fixed,,False,Titres de participation évalués par équivalence (entreprises associées) 
l10n_dz_266,Receivables from Group investments,266,asset_fixed,,False,Créances rattachées à des participations groupe
l10n_dz_267,Receivables from non-group investments,267,asset_fixed,,False,Créances rattachées à des participations hors groupe
l10n_dz_268,Receivables from joint ventures,268,asset_fixed,,False,Créances rattachées à des sociétés en participation
l10n_dz_269,Payments outstanding on equity investments not paid up,269,asset_fixed,,False,Versements restant à effectuer sur titres de participation non libérés
l10n_dz_271,Securities other than portfolio securities,271,asset_fixed,,False,Titres immobilisés autres que les titres immobilisés de l'activité de portefeuille
l10n_dz_272,"Debt securities (bonds, notes)",272,asset_fixed,,False,"Titres représentatifs de droit de créance (obligations, bons)"
l10n_dz_273,Fixed assets of the portfolio activity,273,asset_fixed,,False,Titres immobilisés de l'activité de portefeuille
l10n_dz_274,Loans and receivables under finance leases,274,asset_fixed,,False,Prêts et créances sur contrat de location - financement
l10n_dz_275,Deposits and guarantees paid,275,asset_fixed,,False,Dépôts et cautionnements versés
l10n_dz_276,Other fixed assets,276,asset_fixed,,False,Autres créances immobilisées
l10n_dz_279,Outstanding payments on fixed assets not paid up,279,asset_fixed,,False,Versements restant à effectuer sur titres immobilisés non libérés
l10n_dz_2803,Amortization of capitalizable development costs,2803,asset_fixed,,False,Amortissement des frais de recherche et développement immobilisables
l10n_dz_2804,Amortization of computer and related software,2804,asset_fixed,,False,Amortissement des logiciels informatiques et assimilés
l10n_dz_2805,"Amortization of concessions and similar rights, patents, licenses, trademarks",2805,asset_fixed,,False,"Amortissement concessions et droits similaires, brevets, licences, marques"
l10n_dz_2807,Amortization of goodwill,2807,asset_fixed,,False,Amortissement écart d’acquisition (goodwill)
l10n_dz_2808,Amortization of other intangible assets,2808,asset_fixed,,False,Amortissement autres immobilisations incorporelles
l10n_dz_2812,Amortization of land fixtures and fittings,2812,asset_fixed,,False,Amortissement agencements et aménagements de terrain
l10n_dz_2813,Amortization of buildings,2813,asset_fixed,,False,Amortissement constructions
l10n_dz_2815,Amortization of technical installations,2815,asset_fixed,,False,Amortissement installations techniques
l10n_dz_2818,Amortization of other tangible assets,2818,asset_fixed,,False,Amortissement autres immobilisations corporelles
l10n_dz_282,Amortization of assets under concession,282,asset_fixed,,False,Amortissement des immobilisations mises en concession
l10n_dz_2903,Impairment of capitalizable development costs,2903,asset_fixed,,False,Pertes de valeur sur frais de recherche et développement immobilisables
l10n_dz_2904,Impairment losses on computer software and similar,2904,asset_fixed,,False,Pertes de valeur sur logiciels informatiques et assimilés
l10n_dz_2905,"Impairment of concessions and similar rights, patents, licenses, trademarks",2905,asset_fixed,,False,"Pertes de valeur sur concessions et droits similaires, brevets, licences, marques"
l10n_dz_2907,Impairment of goodwill,2907,asset_fixed,,False,Pertes de valeur sur écart d’acquisition
l10n_dz_2908,Impairment losses on other intangible assets intangible assets,2908,asset_fixed,,False,Pertes de valeur sur autres immobilisations incorporelles
l10n_dz_2912,Impairment of fixtures and fittings,2912,asset_fixed,,False,Pertes de valeur sur agencements et aménagements de terrain
l10n_dz_2913,Impairment of buildings,2913,asset_fixed,,False,Pertes de valeur sur constructions
l10n_dz_2914,Impairment of investment property (fair value),2914,asset_fixed,,False,Pertes de valeur sur immeubles de placement (Juste valeur)
l10n_dz_2915,"Impairment of plant, machinery and equipment",2915,asset_fixed,,False,Pertes de valeur sur installations techniques
l10n_dz_2918,"Impairment of other property, plant and equipment",2918,asset_fixed,,False,Pertes de valeur sur autres immobilisations corporelles
l10n_dz_292,Impairment of assets held under concession,292,asset_fixed,,False,Pertes de valeur sur immobilisations mises en concession
l10n_dz_293,Impairment losses on fixed assets in progress,293,asset_fixed,,False,Pertes de valeur sur immobilisations encours
l10n_dz_296,Impairment losses on investments in subsidiaries and affiliates,296,asset_fixed,,False,Pertes de valeur sur participations et créances rattachées à participations
l10n_dz_297,Impairment losses on other fixed assets,297,asset_fixed,,False,Pertes de valeur sur autres titres immobilisés
l10n_dz_298,Impairment of other financial assets assets,298,asset_fixed,,False,Pertes de valeur sur autres actifs financiers immobilisés
l10n_dz_30,Inventories of goods,30,asset_current,,False,Stocks de marchandises
l10n_dz_31,Raw materials and supplies,31,asset_current,,False,Matières premières et fournitures
l10n_dz_321,Consumable materials,321,asset_current,,False,Matières consommables
l10n_dz_322,Consumable supplies,322,asset_current,,False,Fournitures consommables
l10n_dz_326,Packaging,326,asset_current,,False,Emballages
l10n_dz_331,Product in process,331,asset_current,,False,Produits en cours
l10n_dz_335,Work in Progress,335,asset_current,,False,Travaux en cours
l10n_dz_341,Studies in progress,341,asset_current,,False,Etudes en cours 
l10n_dz_345,Services in progress,345,asset_current,,False,Prestations de services en cours
l10n_dz_351,Intermediate products,351,asset_current,,False,Produits intermédiaires
l10n_dz_355,Finished products,355,asset_current,,False,Produits finis
l10n_dz_358,"Residual or salvaged materials (waste, scrap)",358,asset_current,,False,"Produits résiduels ou matières de récupération (déchets, rebuts)"
l10n_dz_361,Board and accompanying parts,361,asset_current,,False,Lot de bord et pièces d’accompagnement
l10n_dz_362,Dismantled parts and accessories,362,asset_current,,False,Organes et accessoires démantelés
l10n_dz_365,Spare parts recovered from equipment,365,asset_current,,False,Pièces de rechanges récupérées sur matériels
l10n_dz_368,Non-current assets held for sale (reforms),368,asset_non_current,,False,Actifs non courants destinés à être cédés (réformes)
l10n_dz_371,Raw materials in the process of being received,371,asset_current,,False,Matières premières en cours de réception
l10n_dz_372,Other supplies to be received,372,asset_current,,False,Autres approvisionnements à réceptionner
l10n_dz_373,Stocks on deposit or consignment,373,asset_current,,False,Stocks en dépôt ou en consignation
l10n_dz_374,Stocks under customs control,374,asset_current,,False,Stocks sous douanes
l10n_dz_380,Stored goods,380,asset_current,,False,Marchandises stockées
l10n_dz_381,Raw materials and supplies,381,asset_current,,False,Matières premières et fournitures stockées
l10n_dz_382,Other stored supplies,382,asset_current,,False,Autres approvisionnements stockés
l10n_dz_390,Impairment losses on inventories of goods,390,asset_current,,False,Pertes de valeur sur Stocks de marchandises
l10n_dz_391,Impairment losses on raw materials and supplies,391,asset_current,,False,Pertes de valeur sur Matières premières et fournitures
l10n_dz_392,Impairment losses on other supplies,392,asset_current,,False,Pertes de valeur sur Autres approvisionnements
l10n_dz_393,Impairment losses on work in progress of goods,393,asset_current,,False,Pertes de valeur sur En cours de production de biens
l10n_dz_394,Impairment losses on work in progress of services,394,asset_current,,False,Pertes de valeur sur En cours de production de services
l10n_dz_395,Impairment losses on inventories of goods,395,asset_current,,False,Pertes de valeur sur Stocks de marchandises
l10n_dz_397,Impairment losses on external inventories,397,asset_current,,False,Pertes de valeur sur Stocks à l'extérieur
l10n_dz_401,Inventory and service providers,401,liability_payable,,True,Fournisseurs de stocks et services
l10n_dz_403,"Suppliers, notes payable",403,liability_payable,,True,"Fournisseurs, effets à payer"
l10n_dz_404,Suppliers of fixed assets,404,liability_payable,,True,Fournisseurs d'immobilisations
l10n_dz_405,Fixed Assets Suppliers - Notes Payable,405,liability_payable,,True,Fournisseurs d'immobilisations - Effets à payer
l10n_dz_408,Suppliers unpaid invoices,408,liability_payable,,True,Fournisseurs factures non parvenues
l10n_dz_409,"Suppliers - debtors: advances and down payments made, discounts, rebates to be obtained, other receivables",409,asset_current,,False,"Fournisseurs - débiteurs : avances et acomptes versés, rabais, remise, ristourne à obtenir, autres créances"
l10n_dz_411,Clients,411,asset_receivable,,True,Clients
l10n_dz_412,Accounts receivable: Notes receivable (PoS),412,asset_receivable,,True,Clients : effets à recevoir (PoS)
l10n_dz_413,Customers : Notes receivable,413,asset_receivable,,True,Clients : effets à recevoir
l10n_dz_416,Doubtful Customers,416,asset_receivable,,True,Clients douteux
l10n_dz_417,Receivables on work or services in progress,417,asset_receivable,,True,Créances sur travaux ou prestations en cours
l10n_dz_418,Customers - revenue not yet invoiced,418,asset_receivable,,True,Clients - produits non encore facturés
l10n_dz_419,"Accounts payable, advances received, discounts, rebates and other credit notes to be issued",419,liability_current,,False,"Clients créditeurs, avances reçues, rabais, remise, ristourne à accorder et autres avoirs à établir"
l10n_dz_421,"Personnel, salaries due",421,liability_current,,False,"Personnel, rémunérations dues"
l10n_dz_422,Social Work Fund,422,liability_current,,False,Fonds des oeuvres sociales
l10n_dz_423,Employee profit sharing,423,liability_current,,False,Participation des salariés au résultat
l10n_dz_425,"Personnel, advances and deposits granted",425,liability_current,,False,"Personnel, avances et acomptes accordés"
l10n_dz_426,"Personnel, deposits received",426,liability_current,,False,"Personnel, dépôts reçus"
l10n_dz_427,"Personnel, wage oppositions",427,liability_current,,False,"Personnel, oppositions sur salaires"
l10n_dz_428,"Personnel, accrued expenses and accrued income",428,liability_current,,False,"Personnel, charges à payer et produits à recevoir"
l10n_dz_431,Social security,431,liability_current,,False,Sécurité sociale
l10n_dz_432,Other social organizations,432,liability_current,,False,Autres organismes sociaux
l10n_dz_438,Social organizations: Accrued charges and accrued income,438,asset_receivable,,True,Organismes sociaux : charges à payer et produits à recevoir
l10n_dz_441,"State and other public authorities, grants receivable",441,asset_receivable,,True,"Etat et autres collectivités publiques, subventions à recevoir"
l10n_dz_442,"State, taxes and duties recoverable from third parties",442,asset_current,,False,"Etat, impôts et taxes recouvrables sur des tiers"
l10n_dz_443,Special operations with the State and public authorities,443,asset_receivable,,True,Opérations particulières avec l'Etat et les collectivités publiques
l10n_dz_444,"State, income taxes",444,liability_current,,False,"Etat, impôts sur les résultats"
l10n_dz_4455,VAT to be disbursed,4455,liability_current,,False,TVA à décaisser
l10n_dz_4456,Recoverable VAT and withholding tax,4456,asset_current,,False,TVA récupérable et précompte
l10n_dz_4457,Collected VAT,4457,liability_current,,False,TVA collectée
l10n_dz_4458,VAT to be regularized,4458,asset_current,,False,TVA à régulariser
l10n_dz_446,International organizations,446,liability_current,,False,Organismes internationaux
l10n_dz_4471,Tax on professional activity - TAP payable (on G.50),4471,liability_current,,False,Taxe sur l’activité professionnelle – TAP à payer (sur G.50)
l10n_dz_4472,Tax on apprenticeship and professional training,4472,liability_current,,False,Taxe sur l’apprentissage et la formation professionnelle
l10n_dz_4473,Environmental tax - (Ecotax),4473,liability_current,,False,Taxe sur l’environnement – (Écotaxe)
l10n_dz_4474,Registration fees to be paid,4474,liability_current,,False,Droits d’enregistrement à payer
l10n_dz_4475,Property tax to be paid,4475,liability_current,,False,Taxe foncière à payer
l10n_dz_4476,Special (or specific) taxes payable,4476,liability_current,,False,Taxes spéciales (ou spécifiques) à payer
l10n_dz_4478,"Other duties, taxes and fees payable",4478,liability_current,,False,"Autres droits, impôts et taxes à payer"
l10n_dz_448,"State, accrued expenses and accrued income (excluding taxes)",448,liability_current,,False,"Etat, charges à payer et produits à recevoir (hors impôts)"
l10n_dz_451,Group Operations,451,asset_receivable,,True,Opérations groupe
l10n_dz_455,"Partners, current accounts",455,liability_payable,,True,"Associés, Comptes courants"
l10n_dz_456,"Partners, capital transactions",456,asset_receivable,,True,"Associés, opérations sur le capital"
l10n_dz_457,"Partners, dividends to be paid",457,liability_current,,False,"Associés, dividendes à payer"
l10n_dz_458,"Partners, operations carried out in common or in grouping",458,asset_receivable,,True,"Associés, opérations faites en commun ou en groupement"
l10n_dz_462,Receivables on disposal of fixed assets,462,asset_receivable,,True,Créances sur cessions d'immobilisations
l10n_dz_464,Payables on acquisitions of marketable securities and derivative financial instruments,464,liability_payable,,True,Dettes sur acquisitions de valeurs mobilières de placement et instruments financiers dérivés
l10n_dz_465,Receivables on disposals of marketable securities and derivative financial instruments,465,asset_receivable,,True,Créances sur cessions de valeurs mobilières de placement et instruments financiers dérivés
l10n_dz_467,Other accounts receivable or payable,467,liability_current,,False,Autres comptes débiteurs ou créditeurs
l10n_dz_468,Miscellaneous accrued liabilities and accrued income,468,liability_current,,False,Diverses charges à payer et produits à recevoir
l10n_dz_476,Expenditures pending charge,476,liability_current,,False,Dépenses en attente d’imputation
l10n_dz_477,Income pending allocation,477,asset_receivable,,True,Recettes en attente d’imputations
l10n_dz_478,Other operations to be regularized,478,liability_current,,False,Autres opérations à régulariser
l10n_dz_481,"Provisions, current liabilities",481,asset_receivable,,True,"Provisions, passifs courants"
l10n_dz_486,Deferred expenses,486,asset_receivable,,True,Charges constatées d'avance
l10n_dz_487,Deferred income,487,liability_current,,False,Produits constatés d'avance
l10n_dz_491,Impairment losses on trade receivables,491,liability_current,,False,Pertes de valeur sur comptes de clients
l10n_dz_495,Impairment losses on group accounts and on associates,495,liability_current,,False,Pertes de valeur sur comptes du groupe et sur associés
l10n_dz_496,Impairment losses on miscellaneous accounts receivable,496,asset_receivable,,True,Pertes de valeur sur comptes de débiteurs divers
l10n_dz_498,Impairment losses on other third party accounts,498,asset_receivable,,True,Pertes de valeur sur autres comptes de tiers
l10n_dz_501,Shares in affiliated companies,501,asset_current,,False,Parts dans entreprises liées
l10n_dz_502,Own shares,502,asset_current,,False,Actions propres
l10n_dz_503,Other shares or securities conferring a right of ownership,503,asset_current,,False,Autres actions ou titres conférant un droit de propriété
l10n_dz_506,"Bonds, treasury bills and short-term bills",506,asset_current,,False,"Obligations, bons du Trésor et bons de caisse à court terme"
l10n_dz_508,Other marketable securities and similar receivables,508,asset_current,,False,Autres valeurs mobilières de placement et créances assimilées
l10n_dz_509,Payments still to be made on unpaid investment securities,509,asset_current,,False,Versements restant à effectuer sur valeurs mobilières de placement non libérées
l10n_dz_511,Cash in hand,511,asset_cash,,False,Valeur à l'encaissement
l10n_dz_512,Banks current accounts,512,asset_cash,,False,Banques comptes courants
l10n_dz_515,Treasury and public institutions,515,asset_cash,,False,Trésor public et établissements publics
l10n_dz_517,Other financial organizations,517,asset_cash,,False,Autres organismes financiers
l10n_dz_518,Accrued interests,518,asset_cash,,False,Intérêts courus
l10n_dz_519,Current bank loans,519,asset_cash,,False,Concours bancaires courants
l10n_dz_521,Derivative financial instruments: Assets,521,asset_cash,,False,Instruments financiers dérivés : Actifs
l10n_dz_529,Derivative financial instruments: Liabilities,529,asset_cash,,False,Instruments financiers dérivés : Passifs
l10n_dz_530,Cash register,530,asset_cash,,False,Caisse
l10n_dz_541,Imprest Accounts,541,asset_cash,,False,Régies d'avances
l10n_dz_542,Credentials,542,asset_cash,,False,Accréditifs
l10n_dz_581,Transfer of funds,581,asset_cash,,False,Virements de fonds
l10n_dz_588,Other internal transfers,588,asset_cash,,False,Autres virements internes
l10n_dz_591,Loss of value on securities in banks and financial institutions,591,asset_cash,,False,Pertes de valeur sur valeurs en banque et Etablissements financiers
l10n_dz_594,Loss of value on imprest accounts and letters of credit,594,asset_cash,,False,Pertes de valeurs sur régies d'avances et accréditifs
l10n_dz_600,Purchases of goods sold,600,expense,account.account_tag_operating,False,Achats de marchandises vendues
l10n_dz_601,Raw materials,601,expense,account.account_tag_operating,False,Matières premières
l10n_dz_602,Other supplies,602,expense,account.account_tag_operating,False,Autres approvisionnements
l10n_dz_603,Changes in inventory,603,expense,account.account_tag_operating,False,Variations des stocks
l10n_dz_604,Purchases of studies and services,604,expense,account.account_tag_operating,False,Achats d'études et de prestations de services
l10n_dz_605,"Purchase of materials, equipment and works",605,expense,account.account_tag_operating,False,"Achats de matériels, équipements et travaux"
l10n_dz_607,Purchases of non-stock materials and supplies,607,expense,account.account_tag_operating,False,Achats non stockés de matières et fournitures
l10n_dz_608,Incidental purchase costs,608,expense,account.account_tag_operating,False,Frais accessoires d’achat
l10n_dz_609,"Discounts, rebates, discounts obtained on purchases",609,expense,account.account_tag_operating,False,"Rabais, remises, ristournes obtenus sur achats"
l10n_dz_611,General subcontracting,611,expense,account.account_tag_operating,False,Sous traitance générale
l10n_dz_613,Rentals,613,expense,account.account_tag_operating,False,Locations
l10n_dz_614,Rental charges and condominium fees,614,expense,account.account_tag_operating,False,Charges locatives et charges de copropriété
l10n_dz_615,"Care, repairs and maintenance",615,expense,account.account_tag_operating,False,"Entretien, réparations et maintenance"
l10n_dz_616,Insurance premiums,616,expense,account.account_tag_operating,False,Primes d'assurances
l10n_dz_617,Studies and research,617,expense,account.account_tag_operating,False,Etudes et recherches
l10n_dz_618,Documentation and miscellaneous,618,expense,account.account_tag_operating,False,Documentation et divers
l10n_dz_619,"Discounts, rebates and discounts obtained on external services",619,expense,account.account_tag_operating,False,"Rabais, remises et ristournes obtenus sur services extérieurs"
l10n_dz_621,Personnel from outside the company,621,expense,account.account_tag_operating,False,Personnel extérieur à l'entreprise
l10n_dz_622,Remuneration of intermediaries and fees,622,expense,account.account_tag_operating,False,Rémunérations d'intermédiaires et honoraires
l10n_dz_623,"Advertising, publishing, public relations",623,expense,account.account_tag_operating,False,"Publicité, publication, relations publiques"
l10n_dz_624,Transportation of goods and collective transportation of personnel,624,expense,account.account_tag_operating,False,Transports de biens et transport collectif du personnel
l10n_dz_625,"Travel, missions and receptions",625,expense,account.account_tag_operating,False,"Déplacements, missions et réceptions"
l10n_dz_626,Postal and telecommunication expenses,626,expense,account.account_tag_operating,False,Frais postaux et de télécommunications
l10n_dz_627,Banking and related services,627,expense,account.account_tag_operating,False,Services bancaires et assimilés
l10n_dz_628,Dues and miscellaneous,628,expense,account.account_tag_operating,False,Cotisations et divers
l10n_dz_629,"Discounts, rebates and refunds obtained on other external services",629,expense,account.account_tag_operating,False,"Rabais, remises et ristournes obtenus sur autres services extérieurs"
l10n_dz_631,Personnel remuneration,631,expense,account.account_tag_operating,False,Rémunérations du personnel
l10n_dz_634,Operator's remuneration,634,expense,account.account_tag_operating,False,Rémunération de l'exploitant
l10n_dz_635,Contributions to social organizations,635,expense,account.account_tag_operating,False,Cotisations aux organismes sociaux
l10n_dz_636,Social charges for the sole proprietor,636,expense,account.account_tag_operating,False,Charges sociales de l'exploitant individuel
l10n_dz_637,Other social charges,637,expense,account.account_tag_operating,False,Autres charges sociales
l10n_dz_638,Other personnel expenses,638,expense,account.account_tag_operating,False,Autres charges de personnel
l10n_dz_641,"Taxes, duties and similar payments on salaries",641,expense,account.account_tag_operating,False,"Impôts, taxes et versements assimilés sur rémunérations"
l10n_dz_642,Non-recoverable taxes on revenues,642,expense,account.account_tag_operating,False,Impôts et taxes non récupérables sur chiffre d'affaires
l10n_dz_645,Other taxes (excluding income taxes),645,expense,account.account_tag_operating,False,Autres impôts et taxes (hors impôts sur les résultats)
l10n_dz_651,"Royalties for concessions, patents, licenses, software, rights and similar assets",651,expense,account.account_tag_operating,False,"Redevances pour concessions, brevets, licences, logiciels, droits et valeurs similaires"
l10n_dz_652,Losses on disposal of non-financial fixed assets,652,expense,account.account_tag_operating,False,Moins-values sur sortie d'actifs immobilisés non financiers
l10n_dz_653,Attendance fees,653,expense,account.account_tag_operating,False,Jetons de présence
l10n_dz_654,Losses on uncollectible receivables,654,expense,account.account_tag_operating,False,Pertes sur créances irrécouvrables
l10n_dz_655,Share of profit of joint ventures,655,expense,account.account_tag_operating,False,Quote-part de résultat sur opérations faites en commun
l10n_dz_656,"Fines and penalties, grants awarded, donations and gifts",656,expense,account.account_tag_operating,False,"Amendes et pénalités, subventions accordées, dons et libéralités"
l10n_dz_657,Exceptional expenses of current management,657,expense,account.account_tag_operating,False,Charges exceptionnelles de gestion courante
l10n_dz_658,Other operating expenses,658,expense,account.account_tag_operating,False,Autres charges de gestion courante
l10n_dz_661,Interest expenses,661,expense,account.account_tag_financing,False,charges d'intérêts
l10n_dz_664,Losses on receivables from investments,664,expense,account.account_tag_financing,False,Pertes sur créances liées à des participations
l10n_dz_665,Valuation differences on financial assets - losses,665,expense,account.account_tag_financing,False,Ecart d'évaluation sur actifs financiers - moins-values
l10n_dz_666,Losses on foreign exchange,666,expense,account.account_tag_financing,False,Pertes de change
l10n_dz_667,Net losses on disposal of financial assets,667,expense,account.account_tag_financing,False,Pertes nettes sur cessions d’actifs financiers
l10n_dz_668,Other financial expenses,668,expense,account.account_tag_financing,False,Autres charges financières
l10n_dz_67,Extraordinary items (Expenses),67,expense,account.account_tag_investing,False,Eléments extraordinaires (Charges)
l10n_dz_681,"Depreciation, amortization, provisions and impairment impairment losses, non-current assets",681,expense,account.account_tag_operating,False,"Dotations aux amortissements, provisions et pertes de valeur, actifs non courants"
l10n_dz_682,"Depreciation, provisions and impairment of assets under concession",682,expense,account.account_tag_operating,False,"Dotations aux amortissements, provisions et PDV des biens mis en concession"
l10n_dz_685,"Depreciation, amortization, provisions and impairment on current assets",685,expense,account.account_tag_operating,False,"Dotations aux amortissements, provisions et pertes de valeur, actifs courants"
l10n_dz_686,"Depreciation, amortization, provisions and impairment impairment losses, financial items",686,expense,account.account_tag_financing,False,"Dotations aux amortissements, provisions et pertes de valeur, éléments financiers"
l10n_dz_692,Deferred tax assets - Income,692,expense,,False,Imposition différée actif – Produits
l10n_dz_693,Deferred tax liabilities - Expenses,693,expense,,False,Imposition différée passif – Charges
l10n_dz_695,Income taxes based on profit or loss from ordinary activities,695,expense,,False,Impôts sur les bénéfices basés sur le résultat des activités ordinaires
l10n_dz_698,Other income taxes,698,expense,,False,Autres impôts sur les résultats
l10n_dz_700,Sales of merchandise,700,income,account.account_tag_operating,False,Ventes de marchandises
l10n_dz_701,Sales of finished goods,701,income,account.account_tag_operating,False,Ventes de Produits finis
l10n_dz_702,Sales of intermediate products,702,income,account.account_tag_operating,False,Ventes de produits intermédiaires
l10n_dz_703,Sales of residual products,703,income,account.account_tag_operating,False,Ventes de produits résiduels
l10n_dz_704,Sale of works,704,income,account.account_tag_operating,False,Vente de travaux
l10n_dz_705,Sale of studies,705,income,account.account_tag_operating,False,Vente d'études
l10n_dz_706,Other services provided,706,income,account.account_tag_operating,False,Autres prestations de services
l10n_dz_708,Income from auxiliary activities,708,income,account.account_tag_operating,False,Produits des activités annexes
l10n_dz_709,"Discounts, rebates and discounts granted",709,income,account.account_tag_operating,False,"Rabais, remises et ristournes accordés"
l10n_dz_723,Change in inventories of work in progress,723,income,account.account_tag_operating,False,Variation de stocks d'encours
l10n_dz_724,Change in inventory of products,724,income,account.account_tag_operating,False,Variation de stocks de produits
l10n_dz_731,Capitalized production of intangible assets,731,income,account.account_tag_operating,False,Production immobilisée d'actifs incorporels
l10n_dz_732,Capitalized production of tangible assets,732,income,account.account_tag_operating,False,Production immobilisée d'actifs corporels
l10n_dz_741,Balancing subsidy,741,income,account.account_tag_operating,False,Subvention d'équilibre
l10n_dz_748,Other operating subsidies,748,income,account.account_tag_operating,False,Autres subventions d'exploitation
l10n_dz_751,"Royalties for concessions, patents, licenses, software and similar assets",751,income,account.account_tag_operating,False,"Redevances pour concessions, brevets, licences, logiciels et valeurs similaires"
l10n_dz_752,Capital gains on disposal of non-financial fixed assets,752,income,account.account_tag_operating,False,Plus-values sur sorties d'actifs immobilisés non financiers
l10n_dz_753,Directors' and officers' fees and remuneration,753,income,account.account_tag_operating,False,Jetons de présence et rémunérations d'administrateurs ou de gérants
l10n_dz_754,Share of investment grants transferred to income for the year,754,income,account.account_tag_operating,False,Quotes-parts de subventions d’investissement virées au résultat de l’exercice
l10n_dz_755,Share of profit of joint ventures,755,income,account.account_tag_operating,False,Quote-part de résultat sur opérations faites en commun
l10n_dz_756,Receipts from amortized receivables,756,income,account.account_tag_operating,False,Rentrées sur créances amorties
l10n_dz_757,Extraordinary income from management operations,757,income,account.account_tag_operating,False,Produits exceptionnels sur opérations de gestion
l10n_dz_758,Other current management income,758,income,account.account_tag_operating,False,Autres produits de gestion courante
l10n_dz_761,Income from participations,761,income,account.account_tag_financing,False,Produits de participations
l10n_dz_762,Income from financial assets,762,income,account.account_tag_financing,False,Revenus des actifs financiers
l10n_dz_763,Income from receivables,763,income,account.account_tag_financing,False,Revenus de créances
l10n_dz_765,Valuation differences on financial assets - Capital gains,765,income,account.account_tag_financing,False,Ecart d’évaluation sur actifs financiers – Plus-values
l10n_dz_766,Foreign exchange gains,766,income,account.account_tag_financing,False,Gains de change
l10n_dz_767,Net gains on disposal of financial assets,767,income,account.account_tag_financing,False,Profits nets sur cessions d’actifs financiers
l10n_dz_768,Other financial income,768,income,account.account_tag_financing,False,Autres produits financiers
l10n_dz_77,Extraordinary items (Income),77,income,account.account_tag_investing,False,Éléments extraordinaires (revenus)
l10n_dz_781,Operating reversals of impairment losses and provisions - non-current assets,781,income,account.account_tag_operating,False,Reprises d'exploitation sur pertes de valeur et provisions - actifs non courants
l10n_dz_785,Operating reversals of impairment losses and provisions - current assets,785,income,account.account_tag_operating,False,Reprises d'exploitation sur pertes de valeur et provisions - actifs courants
l10n_dz_786,Financial reversals of impairment losses and provisions,786,income,account.account_tag_financing,False,Reprises financières sur pertes de valeur et provisions

```

## File: data\template\account.fiscal.position-dz.csv

```csv
id,sequence,name,auto_apply,vat_required,country_id,country_group_id,tax_ids/tax_src_id,tax_ids/tax_dest_id,name@fr
fiscal_position_template_national,"1",National Regime,"1","1","base.dz",,,,"Régime National"
fiscal_position_template_exo,,Exemption,"1",,,,,,"Exonération"
,,,,,,,l10n_dz_vat_sale_19_resale,l10n_dz_vat_sale_0_resale
,,,,,,,l10n_dz_vat_sale_9_g,l10n_dz_vat_sale_0_g
,,,,,,,l10n_dz_vat_sale_9_s,l10n_dz_vat_sale_0_s
,,,,,,,l10n_dz_vat_sale_9_immo,l10n_dz_vat_sale_0_immo_from_9
,,,,,,,l10n_dz_vat_sale_9_med,l10n_dz_vat_sale_0_med
,,,,,,,l10n_dz_vat_sale_9_cb,l10n_dz_vat_sale_0_cb
,,,,,,,l10n_dz_vat_sale_9_energy,l10n_dz_vat_sale_0_energy
,,,,,,,l10n_dz_vat_sale_19_prod,l10n_dz_vat_sale_0_prod
,,,,,,,l10n_dz_vat_sale_19_immo,l10n_dz_vat_sale_0_immo_from_19
,,,,,,,l10n_dz_vat_sale_19_liberal_professions,l10n_dz_vat_sale_0_liberal_professions
,,,,,,,l10n_dz_vat_sale_19_bank_insurance,l10n_dz_vat_sale_0_bank_insurance
,,,,,,,l10n_dz_vat_sale_19_telephone,l10n_dz_vat_sale_0_telephone
,,,,,,,l10n_dz_vat_sale_19_other_services,l10n_dz_vat_sale_0_other_services
,,,,,,,l10n_dz_vat_sale_19_drink,l10n_dz_vat_sale_0_drink
,,,,,,,l10n_dz_vat_sale_19_tobacco_matches,l10n_dz_vat_sale_0_tobacco_matches
,,,,,,,l10n_dz_vat_sale_19_shows_games,l10n_dz_vat_sale_0_shows_games
,,,,,,,l10n_dz_vat_sale_19_on_site_consumption,l10n_dz_vat_sale_0_on_site_consumption
,,,,,,,l10n_dz_vat_purchase_19,l10n_dz_vat_purchase_0
,,,,,,,l10n_dz_vat_purchase_9,l10n_dz_vat_purchase_0
,,,,,,,l10n_dz_vat_purchase_19_depr,l10n_dz_vat_purchase_0_depr
,,,,,,,l10n_dz_vat_purchase_9_depr,l10n_dz_vat_purchase_0_depr
fiscal_position_template_export_import,,Export/Import,"1",,,,,
,,,,,,,l10n_dz_vat_sale_19_resale,l10n_dz_vat_sale_0_resale
,,,,,,,l10n_dz_vat_sale_9_g,l10n_dz_vat_sale_0_g
,,,,,,,l10n_dz_vat_sale_9_s,l10n_dz_vat_sale_0_s
,,,,,,,l10n_dz_vat_sale_9_immo,l10n_dz_vat_sale_0_immo_from_9
,,,,,,,l10n_dz_vat_sale_9_med,l10n_dz_vat_sale_0_med
,,,,,,,l10n_dz_vat_sale_9_cb,l10n_dz_vat_sale_0_cb
,,,,,,,l10n_dz_vat_sale_9_energy,l10n_dz_vat_sale_0_energy
,,,,,,,l10n_dz_vat_sale_19_prod,l10n_dz_vat_sale_0_prod
,,,,,,,l10n_dz_vat_sale_19_immo,l10n_dz_vat_sale_0_immo_from_19
,,,,,,,l10n_dz_vat_sale_19_liberal_professions,l10n_dz_vat_sale_0_liberal_professions
,,,,,,,l10n_dz_vat_sale_19_bank_insurance,l10n_dz_vat_sale_0_bank_insurance
,,,,,,,l10n_dz_vat_sale_19_telephone,l10n_dz_vat_sale_0_telephone
,,,,,,,l10n_dz_vat_sale_19_other_services,l10n_dz_vat_sale_0_other_services
,,,,,,,l10n_dz_vat_sale_19_drink,l10n_dz_vat_sale_0_drink
,,,,,,,l10n_dz_vat_sale_19_tobacco_matches,l10n_dz_vat_sale_0_tobacco_matches
,,,,,,,l10n_dz_vat_sale_19_shows_games,l10n_dz_vat_sale_0_shows_games
,,,,,,,l10n_dz_vat_sale_19_on_site_consumption,l10n_dz_vat_sale_0_on_site_consumption
,,,,,,,l10n_dz_vat_purchase_19,l10n_dz_vat_purchase_0
,,,,,,,l10n_dz_vat_purchase_9,l10n_dz_vat_purchase_0
,,,,,,,l10n_dz_vat_purchase_19_depr,l10n_dz_vat_purchase_0_depr
,,,,,,,l10n_dz_vat_purchase_9_depr,l10n_dz_vat_purchase_0_depr
fiscal_position_template_eu_free_trade,,"EU Free Trade","1",,,"base.europe",,,"Accord de libre échange avec l'Union Européenne"
,,,,,,,l10n_dz_vat_sale_19_resale,l10n_dz_vat_sale_0_resale
,,,,,,,l10n_dz_vat_sale_9_g,l10n_dz_vat_sale_0_g
,,,,,,,l10n_dz_vat_sale_9_s,l10n_dz_vat_sale_0_s
,,,,,,,l10n_dz_vat_sale_9_immo,l10n_dz_vat_sale_0_immo_from_9
,,,,,,,l10n_dz_vat_sale_9_med,l10n_dz_vat_sale_0_med
,,,,,,,l10n_dz_vat_sale_9_cb,l10n_dz_vat_sale_0_cb
,,,,,,,l10n_dz_vat_sale_9_energy,l10n_dz_vat_sale_0_energy
,,,,,,,l10n_dz_vat_sale_19_prod,l10n_dz_vat_sale_0_prod
,,,,,,,l10n_dz_vat_sale_19_immo,l10n_dz_vat_sale_0_immo_from_19
,,,,,,,l10n_dz_vat_sale_19_liberal_professions,l10n_dz_vat_sale_0_liberal_professions
,,,,,,,l10n_dz_vat_sale_19_bank_insurance,l10n_dz_vat_sale_0_bank_insurance
,,,,,,,l10n_dz_vat_sale_19_telephone,l10n_dz_vat_sale_0_telephone
,,,,,,,l10n_dz_vat_sale_19_other_services,l10n_dz_vat_sale_0_other_services
,,,,,,,l10n_dz_vat_sale_19_drink,l10n_dz_vat_sale_0_drink
,,,,,,,l10n_dz_vat_sale_19_tobacco_matches,l10n_dz_vat_sale_0_tobacco_matches
,,,,,,,l10n_dz_vat_sale_19_shows_games,l10n_dz_vat_sale_0_shows_games
,,,,,,,l10n_dz_vat_sale_19_on_site_consumption,l10n_dz_vat_sale_0_on_site_consumption
,,,,,,,l10n_dz_vat_purchase_19,l10n_dz_vat_purchase_0
,,,,,,,l10n_dz_vat_purchase_9,l10n_dz_vat_purchase_0
,,,,,,,l10n_dz_vat_purchase_19_depr,l10n_dz_vat_purchase_0_depr
,,,,,,,l10n_dz_vat_purchase_9_depr,l10n_dz_vat_purchase_0_depr

```

## File: data\template\account.group-dz.csv

```csv
id,code_prefix_start,name,name@fr
l10n_dz_account_group_1,1,CLASS 1: CAPITAL,CLASSE 1 : CAPITAUX
l10n_dz_account_group_10,10,"Capital, reserves and similar","Capital, réserves et assimilés"
l10n_dz_account_group_11,11,Retained earnings,Report à nouveau
l10n_dz_account_group_12,12,Net income for the year,Résultat de l'exercice
l10n_dz_account_group_13,13,Deferred income and expenses - non-operating cycle,Produits et charges différés – hors cycle d’exploitation
l10n_dz_account_group_15,15,Provisions for charges - non-current liabilities,Provisions pour charges - passifs non courants
l10n_dz_account_group_16,16,Borrowings and similar liabilities,Emprunts et dettes assimilés
l10n_dz_account_group_17,17,Debts related to participating interests,Dettes rattachées à des participations
l10n_dz_account_group_18,18,Liaison accounts of establishments and joint ventures,Comptes de liaison des établissements et sociétés en participation
l10n_dz_account_group_2,2,CLASS 2: FIXED ASSETS,CLASSE 2 : IMMOBILISATIONS
l10n_dz_account_group_20,20,Intangible fixed assets,Immobilisations incorporelles
l10n_dz_account_group_21,21,Tangible fixed assets,Immobilisations corporelles
l10n_dz_account_group_22,22,Fixed assets under concession,Immobilisations en concession
l10n_dz_account_group_23,23,Assets under construction,Immobilisations en cours
l10n_dz_account_group_26,26,Participating interests and receivables related to participating interests,Participations et créances rattachées à des participations
l10n_dz_account_group_27,27,Other financial assets,Autres immobilisations financières
l10n_dz_account_group_28,28,Depreciation of fixed assets,Amortissements des immobilisations
l10n_dz_account_group_29,29,Impairment losses on fixed assets,Pertes de valeur sur immobilisations
l10n_dz_account_group_3,3,CLASS 3: INVENTORIES AND WORK IN PROGRESS,CLASSE 3 : STOCKS ET EN COURS
l10n_dz_account_group_30,30,Inventories of goods,Stocks de marchandises
l10n_dz_account_group_31,31,Raw materials and supplies,Matières premières et fournitures stockées
l10n_dz_account_group_32,32,Other supplies,Pertes de valeur sur Autres approvisionnements
l10n_dz_account_group_33,33,In the course of production of goods,En cours de production de biens
l10n_dz_account_group_34,34,In process of production of services,En cours de production de services
l10n_dz_account_group_35,35,Inventories of products,Stocks de produits
l10n_dz_account_group_36,36,Inventories from fixed assets,Stocks provenant d’immobilisations
l10n_dz_account_group_37,37,"Outside inventories (in process, on deposit or on consignment)","Stocks à l'extérieur (en cours de route, en dépôt ou en consignation)"
l10n_dz_account_group_38,38,Purchases in stock,Achats stockés
l10n_dz_account_group_39,39,Value adjustments to inventories and work in progress,Pertes de valeur sur stocks et en cours
l10n_dz_account_group_4,4,CLASS 4: THIRD PARTIES,CLASSE 4 : TIERS
l10n_dz_account_group_40,40,Suppliers and related accounts,Fournisseurs et comptes rattachés
l10n_dz_account_group_41,41,Trade receivables and related accounts,Clients et comptes rattachés
l10n_dz_account_group_42,42,Personnel and related accounts,Personnel et comptes rattachés
l10n_dz_account_group_43,43,Social organizations and related accounts,Organismes sociaux et comptes rattachés
l10n_dz_account_group_44,44,"State, public authorities, international organizations and related accounts","Etat, collectivités publiques, organismes internationaux et comptes rattachés"
l10n_dz_account_group_45,45,Group and associates,Groupe et Associés
l10n_dz_account_group_46,46,Sundry debtors and creditors,Débiteurs divers et créditeurs divers
l10n_dz_account_group_47,47,Suspense accounts,Comptes transitoires ou d'attente
l10n_dz_account_group_48,48,Prepaid expenses or income and provisions,Charges ou produits constatés d'avance et provisions
l10n_dz_account_group_49,49,Impairment losses on third party accounts,Pertes de valeur sur comptes de tiers
l10n_dz_account_group_5,5,CLASS 5: FINANCIAL ACCOUNTS,CLASSE 5 : COMPTES FINANCIERS
l10n_dz_account_group_50,50,Marketable securities,Valeurs mobilières de placement
l10n_dz_account_group_51,51,"Banks, financial institutions and similar","Banque, établissements financiers et assimilés"
l10n_dz_account_group_52,52,Derivative financial instruments,Instruments financiers dérivés
l10n_dz_account_group_53,53,Cash,Caisse
l10n_dz_account_group_54,54,Imprest accounts and letters of credit,Régies d'avances et accréditifs
l10n_dz_account_group_58,58,Internal transfers,Virements internes
l10n_dz_account_group_59,59,Impairment losses on current financial assets,Pertes de valeur sur actifs financiers courants
l10n_dz_account_group_6,6,CLASS 6: EXPENSES,CLASSE 6 : CHARGES
l10n_dz_account_group_60,60,Purchases consumed,Achats consommés
l10n_dz_account_group_61,61,External services,Services extérieurs
l10n_dz_account_group_62,62,Other external services,Autres services extérieurs
l10n_dz_account_group_63,63,Personnel expenses,Charges de personnel
l10n_dz_account_group_64,64,Taxes and similar payments,"Impôts, Taxes et versements assimilés"
l10n_dz_account_group_65,65,Other operating expenses,Autres charges de gestion courante
l10n_dz_account_group_66,66,Financial expenses,Charges financières différées
l10n_dz_account_group_67,67,Extraordinary items (Expenses),Eléments extraordinaires (Charges)
l10n_dz_account_group_68,68,"Depreciation, amortization, provisions and impairment","Dotations aux amortissements, provisions et pertes de valeur"
l10n_dz_account_group_69,69,Income tax and similar taxes,Impôts sur les résultats et assimilés
l10n_dz_account_group_7,7,CLASS 7: REVENUE,CLASSE 7 : PRODUITS
l10n_dz_account_group_70,70,"Sales of goods and manufactured products, sales of services and related income","Ventes de marchandises et de produits fabriqués, vente de prestations de services et produits annexes"
l10n_dz_account_group_72,72,Production in stock or destocked,Production stockée ou déstockée
l10n_dz_account_group_73,73,Production capitalised,Production immobilisée
l10n_dz_account_group_74,74,Operating subsidies,Subventions d'exploitation
l10n_dz_account_group_75,75,Other operating income,Autres produits opérationnels
l10n_dz_account_group_76,76,Financial income,Autres produits financiers
l10n_dz_account_group_77,77,Extraordinary items (Income),Éléments extraordinaires (revenus)
l10n_dz_account_group_78,78,Reversal of impairment losses and provisions,Reprise sur pertes de valeur et provisions

```

## File: data\template\account.tax-dz.csv

```csv
"id","sequence","description","name","invoice_label","amount","amount_type","type_tax_use","tax_group_id","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@fr","description@fr"
"l10n_dz_vat_sale_19_prod","1","19% Goods Production","19% G Prod","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","","base","invoice","+E3B21_taxable_turnover","","","19% B Prod.","19% Production de biens"
"","","","","","","","","","","tax","invoice","+E3B21_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B21_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B21_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_prod","","0% Goods Production","0% G Prod","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","","base","invoice","+E3B21_exempt_turnover","","","0% B Prod.","0% Production de biens"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B21_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_9_g","","9% Goods, Products, Commodities","9% G","9%","9.0","percent","sale","l10n_dz_tax_group_vat_9","","base","invoice","+E3B11_taxable_turnover","","","9% B","9% Biens, Produits et Denrées"
"","","","","","","","","","","tax","invoice","+E3B11_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B11_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B11_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_g","","0% Goods, Products, Commodities","0% G","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","","base","invoice","+E3B11_exempt_turnover","","","0% B","0% Biens, Produits et Denrées"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B11_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_9_s","","9% Services","9% S","9%","9.0","percent","sale","l10n_dz_tax_group_vat_9","","base","invoice","+E3B12_taxable_turnover","","","9% S",""
"","","","","","","","","","","tax","invoice","+E3B12_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B12_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B12_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_s","","0% Services","0% S","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","","base","invoice","+E3B12_exempt_turnover","","","0% S",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B12_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_9_immo","","9% Real Estate","9% Immo","9%","9.0","percent","sale","l10n_dz_tax_group_vat_9","False","base","invoice","+E3B13_taxable_turnover","","","9% Immo","19% Opérations immobilières"
"","","","","","","","","","","tax","invoice","+E3B13_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B13_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B13_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_immo_from_9","","0% Real Estate","0% Immo 9","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B13_exempt_turnover","","","0% Immo","0% Opérations immobilières"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B13_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_9_med","","9% Medical procedures","9% Med","9%","9.0","percent","sale","l10n_dz_tax_group_vat_9","False","base","invoice","+E3B14_taxable_turnover","","","9% Med","9% Actes médicaux"
"","","","","","","","","","","tax","invoice","+E3B14_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B14_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B14_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_med","","0% Medical procedures","0% Med","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B14_exempt_turnover","","","0% Med","0% Actes médicaux"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B14_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_9_cb","","9% Commissionaires and brokers","9% C&B","9%","9.0","percent","sale","l10n_dz_tax_group_vat_9","False","base","invoice","+E3B15_taxable_turnover","","","9% C&C","9%  - Commissionnaires et courtiers"
"","","","","","","","","","","tax","invoice","+E3B15_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B15_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B15_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_cb","","0% Commissionaires and brokers","0% C&B","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B15_exempt_turnover","","","0% C&C","0%  - Commissionnaires et courtiers"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B15_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_9_energy","","9% Energy supply","9% En","9%","9.0","percent","sale","l10n_dz_tax_group_vat_9","False","base","invoice","+E3B16_taxable_turnover","","","9% En","9% Fourniture d'énergie"
"","","","","","","","","","","tax","invoice","+E3B16_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B16_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B16_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_energy","","0% Energy supply","0% En","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B16_exempt_turnover","","","0% En","0% Fourniture d'énergie"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B16_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_resale","","19% Resale as is","19% G Resale","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","","base","invoice","+E3B22_taxable_turnover","","","19% B Revente","19% Revente en l'état"
"","","","","","","","","","","tax","invoice","+E3B22_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B22_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B22_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_resale","","0% Resale as is","0% G Resale","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","","base","invoice","+E3B22_exempt_turnover","","","0% B Revente","0% Revente en l'état"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B22_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_immo","","19% Real Estate","19% Immo","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B23_taxable_turnover","","","19% Immo","19% Opérations immobilières"
"","","","","","","","","","","tax","invoice","+E3B23_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B23_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B23_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_immo_from_19","","0% Real Estate","0% Immo 19","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B23_exempt_turnover","","","0% Immo","0% Opérations immobilières"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B23_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_liberal_professions","","19% Liberal professions","19% LB","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B24_taxable_turnover","","","19% PL","19% Professions libérales"
"","","","","","","","","","","tax","invoice","+E3B24_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B24_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B24_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_liberal_professions","","0% Liberal professions","0% LB","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B24_exempt_turnover","","","0% PL","0% Professions libérales"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B24_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_bank_insurance","","19% Banking and insurance","19% BI","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B25_taxable_turnover","","","19% BA","19% Opérations de banques et d'assurances"
"","","","","","","","","","","tax","invoice","+E3B25_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B25_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B25_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_bank_insurance","","0% Banking and insurance","0% BI","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B25_exempt_turnover","","","0% BA","0% Opérations de banques et d'assurances"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B25_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_telephone","","19% Telephone and telex services","19% Tel","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B26_taxable_turnover","","","19% Tel","19% Prestations de téléphones et de télex"
"","","","","","","","","","","tax","invoice","+E3B26_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B26_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B26_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_telephone","","0% Telephone and telex services","0% Tel","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B26_exempt_turnover","","","0% Tel","0% Prestations de téléphones et de télex"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B26_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_other_services","","19% Other services","19% OS","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","","base","invoice","+E3B28_taxable_turnover","","","19% AS","19% Autres services"
"","","","","","","","","","","tax","invoice","+E3B28_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B28_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B28_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_other_services","","0% Other services","0% OS","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","","base","invoice","+E3B28_exempt_turnover","","","0% AS","0% Autres services"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B28_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_drink","","19% Drinking places","19% Drink","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B31_taxable_turnover","","","19% Boissons","19% Débits de boissons"
"","","","","","","","","","","tax","invoice","+E3B31_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B31_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B31_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_drink","","0% Drinking places","0% Drink","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B31_exempt_turnover","","","0% Boissons","0% Débits de boissons"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B31_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_tobacco_matches","","19% Tobacco and matches","19% Tob","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B34_taxable_turnover","","","19% TA","19% Tabacs et allumettes"
"","","","","","","","","","","tax","invoice","+E3B34_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B34_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B34_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_tobacco_matches","","0% Tobacco and matches","0% Tob","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B34_exempt_turnover","","","0% TA","0% Tabacs et allumettes"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B34_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_shows_games","","19% Entertainments","19% Shows","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B35_taxable_turnover","","","19% Divertis.","19% Divertissements"
"","","","","","","","","","","tax","invoice","+E3B35_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B35_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B35_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_shows_games","","0% Entertainments","0% Shows","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B35_exempt_turnover","","","0% Divertis.","0% Divertissements"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B35_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_other_services_art21","","19% Other services art. 21","19% OS art 21","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B36_taxable_turnover","","","19% AS art 21","19% Autres services art.21"
"","","","","","","","","","","tax","invoice","+E3B36_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B36_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B36_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_other_services_art21","","0% Other services art. 21","0% OS art 21","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B36_exempt_turnover","","","0% AS art 21","0% Autres services art. 21"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B36_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_sale_19_on_site_consumption","","19% On-site consumption","19% On-Site","19%","19.0","percent","sale","l10n_dz_tax_group_vat_19","False","base","invoice","+E3B37_taxable_turnover","","","19% Sur Site","19% Consommation sur place"
"","","","","","","","","","","tax","invoice","+E3B37_balance","l10n_dz_4457","","",""
"","","","","","","","","","","base","refund","-E3B37_taxable_turnover","","","",""
"","","","","","","","","","","tax","refund","-E3B37_balance","l10n_dz_4457","","",""
"l10n_dz_vat_sale_0_on_site_consumption","","0% On-site consumption","0% On-Site","0%","0.0","percent","sale","l10n_dz_tax_group_vat_0","False","base","invoice","+E3B37_exempt_turnover","","","0% Sur Site","0% Consommation sur place"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-E3B37_exempt_turnover","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_purchase_19","","19% Purchases of goods, materials and services","19%","19%","19.0","percent","purchase","l10n_dz_tax_group_vat_19","","base","invoice","","","","19%","19% Achats de biens, matières et services"
"","","","","","","","","","","tax","invoice","+E3B92_balance","l10n_dz_4456","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","-E3B92_balance","l10n_dz_4456","","",""
"l10n_dz_vat_purchase_9","","9% Purchases of goods, materials and services","9%","9%","9.0","percent","purchase","l10n_dz_tax_group_vat_9","","base","invoice","","","","9%","9% Achats de biens, matières et services"
"","","","","","","","","","","tax","invoice","+E3B92_balance","l10n_dz_4456","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","-E3B92_balance","l10n_dz_4456","","",""
"l10n_dz_vat_purchase_0","","0% Purchases of goods, materials and services","0%","0%","0.0","percent","purchase","l10n_dz_tax_group_vat_0","","base","invoice","","","","0%","0% Achats de biens, matières et services"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"l10n_dz_vat_purchase_19_depr","","19% Purchases of depreciable goods","19% G","19%","19.0","percent","purchase","l10n_dz_tax_group_vat_19","","base","invoice","","","","19% G","19% Achats de biens amortissables"
"","","","","","","","","","","tax","invoice","+E3B93_balance","l10n_dz_4456","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","-E3B93_balance","l10n_dz_4456","","",""
"l10n_dz_vat_purchase_9_depr","","9% Purchases of depreciable goods","9% G","9%","9.0","percent","purchase","l10n_dz_tax_group_vat_9","","base","invoice","","","","9% G","9% Achats de biens amortissables"
"","","","","","","","","","","tax","invoice","+E3B93_balance","l10n_dz_4456","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","-E3B93_balance","l10n_dz_4456","","",""
"l10n_dz_vat_purchase_0_depr","","0% Purchases of depreciable goods","0% G","0%","0.0","percent","purchase","l10n_dz_tax_group_vat_0","","base","invoice","","","","0% G","0% Achats de biens amortissables"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-dz.csv

```csv
"id","country_id","name","name@fr",
"l10n_dz_tax_group_vat_0","base.dz","VAT 0%","TVA 0%",
"l10n_dz_tax_group_vat_9","base.dz","VAT 9%","TVA 9%",
"l10n_dz_tax_group_vat_19","base.dz","VAT 19%","TVA 19%",

```

## File: models\template_dz.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('dz')
    def _get_dz_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_dz_413',
            'property_account_payable_id': 'l10n_dz_401',
            'property_account_expense_categ_id': 'l10n_dz_600',
            'property_account_income_categ_id': 'l10n_dz_700',
            'code_digits': 6,
            'display_invoice_amount_total_words': True,
        }

    @template('dz', 'res.company')
    def _get_dz_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.dz',
                'bank_account_code_prefix': '512',
                'cash_account_code_prefix': '53',
                'transfer_account_code_prefix': '58',
                'account_default_pos_receivable_account_id': 'l10n_dz_412',
                'income_currency_exchange_account_id': 'l10n_dz_766',
                'expense_currency_exchange_account_id': 'l10n_dz_666',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_dz_709',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_dz_609',
                'default_cash_difference_income_account_id': 'l10n_dz_758',
                'default_cash_difference_expense_account_id': 'l10n_dz_657',
                'account_sale_tax_id': 'l10n_dz_vat_sale_19_prod',
                'account_purchase_tax_id': 'l10n_dz_vat_purchase_19',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_dz

```

