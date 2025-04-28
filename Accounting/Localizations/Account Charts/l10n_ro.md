# Odoo Module: l10n_ro

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2022 Odoo S.A.
# @author -  Fekete Mihai <feketemihai@gmail.com>, Tatár Attila <atta@nvm.ro>
# Copyright (C) 2015 Tatár Attila
# Copyright (C) 2015 Forest and Biomass Services Romania (http://www.forbiom.eu).
# Copyright (C) 2011 TOTAL PC SYSTEMS (http://www.erpsystems.ro).
# Copyright (C) 2009 (<http://www.filsystem.ro>)

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Romania - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/romania.html',
    'author': 'Fekete Mihai (NextERP Romania SRL), Odoo S.A.',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ro'],
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.0',
    'depends': [
        'account',
        'base_vat',
    ],
    'auto_install': ['account'],
    'description': """
This is the module to manage the Accounting Chart, VAT structure, Fiscal Position and Tax Mapping.
It also adds the Registration Number for Romania in Odoo.
================================================================================================================

Romanian accounting chart and localization.
    """,
    'data': [
        'views/res_partner_view.xml',
        'data/account_tax_report_data.xml',
        'data/res.bank.csv',
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
        <field name="name">Romanian Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ro"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_ro_baza_intracom_eu" model="account.report.line">
                <field name="name">TAX BASE TRADE WITHIN AND OUTSIDE THE EU</field>
                <field name="code">tax_ro_baza_intracom_eu</field>
                <field name="aggregation_formula">tax_ro_baza_rd1.balance + tax_ro_baza_rd2.balance + tax_ro_baza_rd3.balance + tax_ro_baza_rd4.balance + tax_ro_baza_rd5.balance + tax_ro_baza_rd6.balance + tax_ro_baza_rd7.balance + tax_ro_baza_rd8.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd1" model="account.report.line">
                        <field name="name">1 - TAX BASE - Intra-Community supplies of goods, exempt under Article 294(2)(a) and (d) of the Tax Code</field>
                        <field name="code">tax_ro_baza_rd1</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">01 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd2" model="account.report.line">
                        <field name="name">2 - TAX BASE - Regularisation of intra-Community supplies exempted under Article 294(2)(a) and (d) of the Tax Code</field>
                        <field name="code">tax_ro_baza_rd2</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">02 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd3" model="account.report.line">
                        <field name="name">3 - TAX BASE - Supplies of goods/services for which the place of supply is outside Romania, as well as intracom. supplies of goods, shield. under Art. 294 (2) (b) and (c) of the CF, of which:</field>
                        <field name="code">tax_ro_baza_rd3</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">03 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd31" model="account.report.line">
                                <field name="name">3.1 - TAX BASE - Intra-Community supplies of services not exempt in the Member State where the tax is due</field>
                                <field name="code">tax_ro_baza_rd31</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd31_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">03_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd4" model="account.report.line">
                        <field name="name">4 - TAX BASE - Adjustments for intra-Community supplies of services which are not exempt in the Member State where the tax is due</field>
                        <field name="code">tax_ro_baza_rd4</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">04 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd5" model="account.report.line">
                        <field name="name">5 - TAX BASE - Intra-Community acquisitions of goods for which the purchaser is liable for VAT (reverse charge), of which:</field>
                        <field name="code">tax_ro_baza_rd5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">05 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd51" model="account.report.line">
                                <field name="name">5.1 - TAX BASE - Intracom. purchases for which the purchaser is liable for VAT (IT) and the supplier is registered for VAT in the Member State from which the intracom. supply took place.</field>
                                <field name="code">tax_ro_baza_rd51</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd51_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">05_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd6" model="account.report.line">
                        <field name="name">6 - TAX BASE - Adjustments for intra-Community acquisitions of goods for which the purchaser is liable for VAT (reverse charge)</field>
                        <field name="code">tax_ro_baza_rd6</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">06 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd7" model="account.report.line">
                        <field name="name">7 - TAX BASE - Purchases of goods other than those under items 5 and 6 and purchases of services for which the recipient in Romania is liable to pay VAT (reverse charge) of which:</field>
                        <field name="code">tax_ro_baza_rd7</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">07 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd71" model="account.report.line">
                                <field name="name">7.1 - TAX BASE - Intra-Community purchases of services for which the recipient is liable for VAT (reverse charge)</field>
                                <field name="code">tax_ro_baza_rd71</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd71_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">07_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd8" model="account.report.line">
                        <field name="name">8 - TAX BASE - Adjustments for intra-Community purchases of services for which the beneficiary is liable for VAT (reverse charge)</field>
                        <field name="code">tax_ro_baza_rd8</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">08 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_intracom_eu" model="account.report.line">
                <field name="name">VAT INTRA AND EXTRA EU TRADE</field>
                <field name="code">tax_ro_tva_intracom_eu</field>
                <field name="aggregation_formula">tax_ro_tva_rd5.balance + tax_ro_tva_rd6.balance + tax_ro_tva_rd7.balance + tax_ro_tva_rd8.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd5" model="account.report.line">
                        <field name="name">5 - VAT - Intra-Community acquisitions of goods for which the purchaser is liable to pay VAT (reverse charge), of which:</field>
                        <field name="code">tax_ro_tva_rd5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">05 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd51" model="account.report.line">
                                <field name="name">5.1 - VAT - Intra-Community acquisitions for which the purchaser is liable for VAT (IT) and the supplier is registered for VAT in the Member State from which the intra-Community supply took place</field>
                                <field name="code">tax_ro_tva_rd51</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd51_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">05_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd6" model="account.report.line">
                        <field name="name">6 - VAT - Adjustments for intra-Community acquisitions of goods for which the purchaser is liable for VAT (reverse charge)</field>
                        <field name="code">tax_ro_tva_rd6</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">06 - VAT</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd7" model="account.report.line">
                        <field name="name">7 - VAT - Purchases of goods other than those under items 5 and 6 and purchases of services for which the recipient in Romania is liable to pay VAT (reverse charge) of which:</field>
                        <field name="code">tax_ro_tva_rd7</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">07 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd71" model="account.report.line">
                                <field name="name">7.1 - VAT - Intra-Community purchases of services for which the recipient is liable to pay VAT (reverse charge)</field>
                                <field name="code">tax_ro_tva_rd71</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd71_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">07_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd8" model="account.report.line">
                        <field name="name">8 - VAT - Adjustments relating to purchases of intra-Community services for which the beneficiary is liable to pay VAT (reverse charge)</field>
                        <field name="code">tax_ro_tva_rd8</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">08 - VAT</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_livrari" model="account.report.line">
                <field name="name">TAX BASE ON DOMESTIC SUPPLIES OF GOODS/SERVICES AND EXPORTS</field>
                <field name="code">tax_ro_baza_livrari</field>
                <field name="aggregation_formula">tax_ro_baza_rd9.balance + tax_ro_baza_rd10.balance + tax_ro_baza_rd11.balance + tax_ro_baza_rd12.balance + tax_ro_baza_rd13.balance + tax_ro_baza_rd14.balance + tax_ro_baza_rd15.balance + tax_ro_baza_rd16.balance + tax_ro_baza_rd17.balance + tax_ro_baza_rd18.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd9" model="account.report.line">
                        <field name="name">9 - TAX BASE - Supplies of goods and services taxable at 19% rate</field>
                        <field name="code">tax_ro_baza_rd9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">09 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd91" model="account.report.line">
                                <field name="name">9.1 - TAX BASE - Supplies of goods and services taxable at 19% rate</field>
                                <field name="code">tax_ro_baza_rd91</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd91_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd92" model="account.report.line">
                                <field name="name">9.2 - TAX BASE - Non-deductible purchases of goods and services 50% taxable at 19% rate</field>
                                <field name="code">tax_ro_baza_rd92</field>
                                <field name="aggregation_formula">0.5 * tax_ro_baza_rd242.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd10" model="account.report.line">
                        <field name="name">10 - TAX BASE - Supplies of goods and services taxable at 9% rate</field>
                        <field name="code">tax_ro_baza_rd10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd101" model="account.report.line">
                                <field name="name">10.1 - TAX BASE - Supplies of goods and services taxable at 9% rate</field>
                                <field name="code">tax_ro_baza_rd101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd101_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd102" model="account.report.line">
                                <field name="name">10_2 - TAX BASE - Non-deductible purchases of goods and services 50% taxable at 9% rate</field>
                                <field name="code">tax_ro_baza_rd102</field>
                                <field name="aggregation_formula">0.5 * tax_ro_baza_rd252.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd11" model="account.report.line">
                        <field name="name">11 - TAX BASE - Supplies of taxable goods at 5% rate</field>
                        <field name="code">tax_ro_baza_rd11</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd111" model="account.report.line">
                                <field name="name">11.1 - TAX BASE - Supplies of goods and services taxable at 5% rate</field>
                                <field name="code">tax_ro_baza_rd111</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd111_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd112" model="account.report.line">
                                <field name="name">11.2 - TAX BASE - Non-deductible purchases of goods and services 50% taxable at 5% rate</field>
                                <field name="code">tax_ro_baza_rd112</field>
                                <field name="aggregation_formula">0.5 * tax_ro_baza_rd262.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd12" model="account.report.line">
                        <field name="name">12 - TAX BASE - Purchases of goods and services subject to simplification measures for which the beneficiary is liable to pay VAT (reverse charge) , of which:</field>
                        <field name="code">tax_ro_baza_rd12</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd121" model="account.report.line">
                                <field name="name">12.1 - TAX BASE - Purchases of goods and services, taxable at 19% rate</field>
                                <field name="code">tax_ro_baza_rd121</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd121_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd122" model="account.report.line">
                                <field name="name">12.2 - TAX BASE - Purchases of goods and services, taxable at 9% rate</field>
                                <field name="code">tax_ro_baza_rd122</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd122_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_2 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd123" model="account.report.line">
                                <field name="name">12.3 - TAX BASE - Purchases of goods and services, taxable at 5% rate</field>
                                <field name="code">tax_ro_baza_rd123</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd123_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_3 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd13" model="account.report.line">
                        <field name="name">13 - TAX BASE - Supplies of goods and services subject to simplification measures (reverse charge)</field>
                        <field name="code">tax_ro_baza_rd13</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd14" model="account.report.line">
                        <field name="name">14 - TAX BASE - Exempt supplies of goods and services with the right to deduct, other than those under headings 1-3</field>
                        <field name="code">tax_ro_baza_rd14</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">14 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd15" model="account.report.line">
                        <field name="name">15 - TAX BASE - Supplies of goods and services exempt without deduction</field>
                        <field name="code">tax_ro_baza_rd15</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd16" model="account.report.line">
                        <field name="name">16 - TAX BASE - Regularisations collected tax</field>
                        <field name="code">tax_ro_baza_rd16</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">16 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd17" model="account.report.line">
                        <field name="name">17 - TAX BASE - Intra-Community supply of services under Article 278(8) of the Tax Code for which the place of supply is in Romania</field>
                        <field name="code">tax_ro_baza_rd17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">17 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd18" model="account.report.line">
                        <field name="name">18 - TAX BASE - Adjustments for intra-Community supplies of services under Article 278(8) of the Tax Code for which the place of supply is in Romania</field>
                        <field name="code">tax_ro_baza_rd18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">18 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_livrari" model="account.report.line">
                <field name="name">VAT ON DOMESTIC SUPPLIES OF GOODS/SERVICES AND EXPORTS</field>
                <field name="code">tax_ro_tva_livrari</field>
                <field name="aggregation_formula">tax_ro_tva_rd9.balance + tax_ro_tva_rd10.balance + tax_ro_tva_rd11.balance + tax_ro_tva_rd12.balance + tax_ro_tva_rd16.balance + tax_ro_tva_rd17.balance + tax_ro_tva_rd18.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd9" model="account.report.line">
                        <field name="name">9 - VAT - Supplies of goods and services taxable at 19% rate</field>
                        <field name="code">tax_ro_tva_rd9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">09 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd91" model="account.report.line">
                                <field name="name">9.1 - VAT - Supplies of goods and services taxable at 19% rate</field>
                                <field name="code">tax_ro_tva_rd91</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd91_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd92" model="account.report.line">
                                <field name="name">9.2 - VAT - Non-deductible purchases of goods and services 50% taxable at 19% rate</field>
                                <field name="code">tax_ro_tva_rd92</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd92_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd10" model="account.report.line">
                        <field name="name">10 - VAT - Supplies of goods and services taxable at 9% rate</field>
                        <field name="code">tax_ro_tva_rd10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd101" model="account.report.line">
                                <field name="name">10.1 - VAT - Supplies of goods and services taxable at 9% rate</field>
                                <field name="code">tax_ro_tva_rd101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd101_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd102" model="account.report.line">
                                <field name="name">10.2 - VAT - Non-deductible purchases of goods and services 50% taxable at 9% rate</field>
                                <field name="code">tax_ro_tva_rd102</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd102_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd11" model="account.report.line">
                        <field name="name">11 - VAT - Supplies of taxable goods at 5% rate</field>
                        <field name="code">tax_ro_tva_rd11</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd111" model="account.report.line">
                                <field name="name">11.1 - VAT - Supplies of goods and services taxable at 5% rate</field>
                                <field name="code">tax_ro_tva_rd111</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd111_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd112" model="account.report.line">
                                <field name="name">11.2 - VAT - Non-deductible purchases of goods and services 50% taxable at 5% rate</field>
                                <field name="code">tax_ro_tva_rd112</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd112_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd12" model="account.report.line">
                        <field name="name">12 - VAT - Purchases of goods and services subject to simplification measures for which the beneficiary is liable to pay VAT (reverse charge) , of which:</field>
                        <field name="code">tax_ro_tva_rd12</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd121" model="account.report.line">
                                <field name="name">12.1 - VAT - Purchases of goods and services, taxable at 19% rate</field>
                                <field name="code">tax_ro_tva_rd121</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd121_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd122" model="account.report.line">
                                <field name="name">12.2 - VAT - Purchases of goods and services, taxable at 9% rate</field>
                                <field name="code">tax_ro_tva_rd122</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd122_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd123" model="account.report.line">
                                <field name="name">12.3 - VAT - Purchases of goods and services, taxable at 5% rate</field>
                                <field name="code">tax_ro_tva_rd123</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd123_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_3 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd16" model="account.report.line">
                        <field name="name">16 - VAT - Regularisations collected tax</field>
                        <field name="code">tax_ro_tva_rd16</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">16 - VAT</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd17" model="account.report.line">
                        <field name="name">17 - VAT - Intra-Community supply of services under Article 278(8) of the Tax Code for which the place of supply is in Romania</field>
                        <field name="code">tax_ro_tva_rd17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">17 - VAT</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd18" model="account.report.line">
                        <field name="name">18 - VAT - Adjustments for intra-Community supplies of services under Article 278(8) of the Tax Code for which the place of supply is in Romania</field>
                        <field name="code">tax_ro_tva_rd18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">18 - VAT</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_col" model="account.report.line">
                <field name="name">Tax Base Total Fee COLLECTED</field>
                <field name="code">total_tax_ro_baza_col</field>
                <field name="aggregation_formula">tax_ro_baza_intracom_eu.balance + tax_ro_baza_livrari.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_col" model="account.report.line">
                <field name="name">VAT Total Tax COLLECTED</field>
                <field name="code">total_tax_ro_tva_col</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu.balance + tax_ro_tva_livrari.balance</field>
            </record>
            <record id="account_tax_report_ro_baza_intracom_eu_achiz" model="account.report.line">
                <field name="name">TAX BASIS OF INTRA-COMMUNITY ACQUISITIONS OF GOODS AND OTHER TAXABLE ACQUISITIONS OF GOODS AND SERVICES IN ROMANIA</field>
                <field name="code">tax_ro_baza_intracom_eu_a</field>
                <field name="aggregation_formula">tax_ro_baza_rd20.balance + tax_ro_baza_rd21.balance + tax_ro_baza_rd22.balance + tax_ro_baza_rd23.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd20" model="account.report.line">
                        <field name="name">20 - TAX BASE - Intra-Community acquisitions of goods for which the purchaser is liable to pay VAT (reverse charge) (row 18=row 5), of which:</field>
                        <field name="code">tax_ro_baza_rd20</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">20 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd201" model="account.report.line">
                                <field name="name">20.1 - TAX BASE - Intracom. purchases for which the purchaser is liable for VAT (IT) and the supplier is registered for VAT in the Member State from which the supply took place (row 18.1=row 5.1)</field>
                                <field name="code">tax_ro_baza_rd201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd201_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">20_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd21" model="account.report.line">
                        <field name="name">21 - TAX BASE - Adjustments for intra-Community acquisitions of goods for which the purchaser is liable for VAT (reverse charge) (row 19=row 6)</field>
                        <field name="code">tax_ro_baza_rd21</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">21 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd22" model="account.report.line">
                        <field name="name">22 - TAX BASE - Purchases of goods, other than those under headings 18 and 19, and purchases of services for which the recipient in Romania is liable to pay VAT (reverse charge) (heading 20 = heading 7), of which:</field>
                        <field name="code">tax_ro_baza_rd22</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd221" model="account.report.line">
                                <field name="name">22.1 - TAX BASE - Intra-Community purchases of services for which the recipient is liable for VAT (reverse charge) (row 20.1=row 7.1)</field>
                                <field name="code">tax_ro_baza_rd221</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd221_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd23" model="account.report.line">
                        <field name="name">23 - TAX BASE - Adjustments for intra-Community purchases of services for which the beneficiary in Romania is liable to pay VAT (reverse charge) (row 21=row 8)</field>
                        <field name="code">tax_ro_baza_rd23</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23 - TAX BASE</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_intracom_eu_achiz" model="account.report.line">
                <field name="name">VAT ON INTRA-COMMUNITY ACQUISITIONS OF GOODS AND OTHER TAXABLE ACQUISITIONS OF GOODS AND SERVICES IN ROMANIA</field>
                <field name="code">tax_ro_tva_intracom_eu_a</field>
                <field name="aggregation_formula">tax_ro_tva_rd20.balance + tax_ro_tva_rd21.balance + tax_ro_tva_rd22.balance + tax_ro_tva_rd23.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd20" model="account.report.line">
                        <field name="name">20 - VAT - Intra-Community acquisitions of goods for which the purchaser is liable to pay VAT (reverse charge) (row 18=row 5), of which:</field>
                        <field name="code">tax_ro_tva_rd20</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">20 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd201" model="account.report.line">
                                <field name="name">20.1 - VAT - Intracom. purchases for which the purchaser is liable for VAT (IT) and the supplier is registered for VAT in the Member State from which the supply took place (row 18.1=row 5.1)</field>
                                <field name="code">tax_ro_tva_rd201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd201_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">20_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd21" model="account.report.line">
                        <field name="name">21 - VAT - Adjustments for intra-Community acquisitions of goods for which the purchaser is liable for VAT (reverse charge) (row 19=row 6)</field>
                        <field name="code">tax_ro_tva_rd21</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">21 - VAT</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd22" model="account.report.line">
                        <field name="name">22 - VAT - Purchases of goods, other than those under headings 18 and 19, and purchases of services for which the recipient in Romania is liable to pay VAT (reverse charge) (heading 20 = heading 7), of which:</field>
                        <field name="code">tax_ro_tva_rd22</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd221" model="account.report.line">
                                <field name="name">22.1 - VAT - Intra-Community purchases of services for which the recipient is liable for VAT (reverse charge) (row 20.1=row 7.1)</field>
                                <field name="code">tax_ro_tva_rd221</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd221_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd23" model="account.report.line">
                        <field name="name">23 - VAT - Adjustments for intra-Community purchases of services for which the beneficiary in Romania is liable to pay VAT (reverse charge) (row 21=row 8)</field>
                        <field name="code">tax_ro_tva_rd23</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23 - VAT</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_achiz" model="account.report.line">
                <field name="name">TAX BASE ON DOMESTIC PURCHASES OF GOODS/SERVICES AND IMPORTS, EXEMPT OR NON-TAXABLE INTRA-COMMUNITY PURCHASES</field>
                <field name="code">tax_ro_baza_achiz</field>
                <field name="aggregation_formula">tax_ro_baza_rd24.balance + tax_ro_baza_rd25.balance + tax_ro_baza_rd26.balance + tax_ro_baza_rd27.balance + tax_ro_baza_rd30.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd24" model="account.report.line">
                        <field name="name">24 - TAX BASE - Purchases of goods and services taxable at 19%, other than those under heading 27</field>
                        <field name="code">tax_ro_baza_rd24</field>
                        <field name="aggregation_formula">tax_ro_baza_rd241.balance + 0.5 * tax_ro_baza_rd242.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd241" model="account.report.line">
                                <field name="name">24.1 - TAX BASE - Purchases of goods and services taxable at 19%, other than those under heading 27</field>
                                <field name="code">tax_ro_baza_rd241</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd241_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd242" model="account.report.line">
                                <field name="name">24.2 - TAX BASE - Purchases of goods and services taxable at 19%, non-deductible 50%.</field>
                                <field name="code">tax_ro_baza_rd242</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd242_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_2 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd25" model="account.report.line">
                        <field name="name">25 - TAX BASE - Purchases of goods and services taxable at 9% rate</field>
                        <field name="code">tax_ro_baza_rd25</field>
                        <field name="aggregation_formula">tax_ro_baza_rd251.balance + 0.5 * tax_ro_baza_rd252.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd251" model="account.report.line">
                                <field name="name">25.1 - TAX BASE - Purchases of goods and services taxable at 9% rate</field>
                                <field name="code">tax_ro_baza_rd251</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd251_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd252" model="account.report.line">
                                <field name="name">25.2 - TAX BASE - Purchases of goods and services taxable at 9%, non-deductible at 50%</field>
                                <field name="code">tax_ro_baza_rd252</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd252_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_2 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd26" model="account.report.line">
                        <field name="name">26 - TAX BASE - Purchases of taxable goods at 5% rate</field>
                        <field name="code">tax_ro_baza_rd26</field>
                        <field name="aggregation_formula">tax_ro_baza_rd261.balance + 0.5 * tax_ro_baza_rd262.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd261" model="account.report.line">
                                <field name="name">26.1 - TAX BASE - Purchases of taxable goods at 5% rate</field>
                                <field name="code">tax_ro_baza_rd261</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd261_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd262" model="account.report.line">
                                <field name="name">26.2 - TAX BASE - Purchases of goods taxable at 5%, non-deductible 50%</field>
                                <field name="code">tax_ro_baza_rd262</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd262_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_2 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd27" model="account.report.line">
                        <field name="name">27 - TAX BASE - Purchases of goods and services subject to simplification measures for which the beneficiary is liable to pay VAT (reverse charge), of which (row 25=row 12)</field>
                        <field name="code">tax_ro_baza_rd27</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd27_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">27 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd271" model="account.report.line">
                                <field name="name">27.1 - TAX BASE - Purchases of goods and services, taxable at 19% (row 25.1=row 12.1)</field>
                                <field name="code">tax_ro_baza_rd271</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd271_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd272" model="account.report.line">
                                <field name="name">27.2 - TAX BASE - Purchases of goods, taxable at 9% (row 25.2=row 12.2)</field>
                                <field name="code">tax_ro_baza_rd272</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd272_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_2 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd273" model="account.report.line">
                                <field name="name">27.3 - TAX BASE - Purchases of goods, taxable at 5% (row 25.3=row 12.3)</field>
                                <field name="code">tax_ro_baza_rd273</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd273_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_3 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd30" model="account.report.line">
                        <field name="name">30 - TAX BASE - Purchases of tax-exempt or non-taxable goods and services, of which:</field>
                        <field name="code">tax_ro_baza_rd30</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd30_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">30 - TAX BASE</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd301" model="account.report.line">
                                <field name="name">30.1 - TAX BASE - Tax-exempt intra-Community purchases of services (not completed under the simplified method)</field>
                                <field name="code">tax_ro_baza_rd301</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd301_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">30_1 - TAX BASE</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_achiz" model="account.report.line">
                <field name="name">VAT ON DOMESTIC PURCHASES OF GOODS/SERVICES AND IMPORTS, EXEMPT OR NON-TAXABLE INTRA-COMMUNITY PURCHASES</field>
                <field name="code">tax_ro_tva_achiz</field>
                <field name="aggregation_formula">tax_ro_tva_rd24.balance + tax_ro_tva_rd25.balance + tax_ro_tva_rd26.balance + tax_ro_tva_rd27.balance + tax_ro_tva_rd28.balance + tax_ro_tva_rd29.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd24" model="account.report.line">
                        <field name="name">24 - VAT - Purchases of goods and services taxable at 19%, other than those under heading 27</field>
                        <field name="code">tax_ro_tva_rd24</field>
                        <field name="aggregation_formula">tax_ro_tva_rd241.balance + tax_ro_tva_rd242.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd241" model="account.report.line">
                                <field name="name">24.1 - VAT - Purchases of goods and services taxable at 19%, other than those under heading 27</field>
                                <field name="code">tax_ro_tva_rd241</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd241_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd242" model="account.report.line">
                                <field name="name">24.2 - VAT - Purchases of goods and services taxable at 19%, non-deductible 50%.</field>
                                <field name="code">tax_ro_tva_rd242</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd242_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd25" model="account.report.line">
                        <field name="name">25 - VAT - Purchases of goods and services taxable at 9% rate</field>
                        <field name="code">tax_ro_tva_rd25</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd25_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd251" model="account.report.line">
                                <field name="name">25.1 - VAT - Purchases of goods and services taxable at 9% rate</field>
                                <field name="code">tax_ro_tva_rd251</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd251_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd252" model="account.report.line">
                                <field name="name">25.2 - VAT - Purchases of goods and services taxable at 9%, non-deductible at 50%</field>
                                <field name="code">tax_ro_tva_rd252</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd252_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd26" model="account.report.line">
                        <field name="name">26 - VAT - Purchases of taxable goods at 5% rate</field>
                        <field name="code">tax_ro_tva_rd26</field>
                        <field name="aggregation_formula">tax_ro_tva_rd261.balance + tax_ro_tva_rd262.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd261" model="account.report.line">
                                <field name="name">26.1 - VAT - Purchases of taxable goods at 5% rate</field>
                                <field name="code">tax_ro_tva_rd261</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd261_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd262" model="account.report.line">
                                <field name="name">26.2 - VAT - Purchases of goods taxable at 5%, non-deductible 50%</field>
                                <field name="code">tax_ro_tva_rd262</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd262_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd27" model="account.report.line">
                        <field name="name">27 - VAT - Purchases of goods and services subject to simplification measures for which the beneficiary is liable to pay VAT (reverse charge), of which (row 25=row 12)</field>
                        <field name="code">tax_ro_tva_rd27</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd27_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">27 - VAT</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd271" model="account.report.line">
                                <field name="name">27.1 - VAT - Purchases of goods and services, taxable at 19% (row 25.1=row 12.1)</field>
                                <field name="code">tax_ro_tva_rd271</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd271_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_1 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd272" model="account.report.line">
                                <field name="name">27.2 - VAT - Purchases of goods, taxable at 9% (row 25.2=row 12.2)</field>
                                <field name="code">tax_ro_tva_rd272</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd272_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_2 - VAT</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd273" model="account.report.line">
                                <field name="name">27.3 - VAT - Purchases of goods, taxable at 5% (row 25.3=row 12.3)</field>
                                <field name="code">tax_ro_tva_rd273</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd273_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_3 - VAT</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd28" model="account.report.line">
                        <field name="name">28 - VAT - Flat-rate compensation for purchases of agricultural products and services from suppliers applying the special scheme for farmers</field>
                        <field name="code">tax_ro_tva_rd28</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd28_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">28 - VAT</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd29" model="account.report.line">
                        <field name="name">29 - VAT - Flat-rate compensation adjustments</field>
                        <field name="code">tax_ro_tva_rd29</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd29_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">29 - VAT</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_total_rd31" model="account.report.line">
                <field name="name">31 - TAX BASE - TOTAL DEDUCTIBLE TAX (amount from row 20 to row 29, except row 20.1,22.1, 27.1, 27.2, 27.3)</field>
                <field name="code">tax_ro_baza_total_rd31</field>
                <field name="aggregation_formula">tax_ro_baza_intracom_eu_a.balance + tax_ro_baza_achiz.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_total_rd31" model="account.report.line">
                <field name="name">31 - VAT - TOTAL DEDUCTIBLE TAX (amount from row 20 to row 29, except row 20.1,22.1, 27.1, 27.2, 27.3)</field>
                <field name="code">tax_ro_tva_total_rd31</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu_a.balance + tax_ro_tva_achiz.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_rd32" model="account.report.line">
                <field name="name">32 - VAT - SUB-TOTAL TAX DEDUCTED PURSUANT TO ARTICLE 297 AND ARTICLE 298 OR ARTICLE 300 AND ARTICLE 298 (row 30&lt;=row 29)</field>
                <field name="code">tax_ro_tva_rd32</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu_a.balance + tax_ro_tva_achiz.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_rd33" model="account.report.line">
                <field name="name">33 - VAT - VAT actually refunded to foreign purchasers, including commission to authorised establishments</field>
                <field name="code">tax_ro_tva_rd33</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd33_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">33 - VAT</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_rd34" model="account.report.line">
                <field name="name">34 - TAX BASE - Deducted tax adjustments</field>
                <field name="code">tax_ro_baza_rd34</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_baza_rd34_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">34 - TAX BASE</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd34" model="account.report.line">
                <field name="name">34 - VAT - Deducted tax adjustments</field>
                <field name="code">tax_ro_tva_rd34</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd34_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">34 - VAT</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd35" model="account.report.line">
                <field name="name">35 - VAT - Pro-rata adjustments / adjustments for capital goods</field>
                <field name="code">tax_ro_tva_rd35</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd35_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">35 - VAT</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd36" model="account.report.line">
                <field name="name">36 - VAT - TOTAL TAX DEDUCTED (row 32+row 33+row 34+row 35)</field>
                <field name="code">tax_ro_tva_rd36</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu_a.balance + tax_ro_tva_achiz.balance + tax_ro_tva_rd33.balance + tax_ro_tva_rd34.balance + tax_ro_tva_rd35.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_rd40" model="account.report.line">
                <field name="name">40 - VAT - VAT differences to be paid established by the tax inspection authorities by means of a communicated decision and not paid by the date of submission of the VAT return</field>
                <field name="code">tax_ro_tva_rd40</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd40_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">40 - VAT</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd43" model="account.report.line">
                <field name="name">43 - VAT - Negative VAT differences established by the tax inspection authorities by decision communicated by the date of submission of the VAT return</field>
                <field name="code">tax_ro_tva_rd43</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd43_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">43 - VAT</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\res.bank.csv

```csv
"id","active","name","bic","street","city","state/id","country/id","phone"
"res_bank_1","TRUE","Alpha Bank","BUCUROBU","Calea Dorobantilor 237 B, sector1","BUCURESTI","base.RO_B","base.ro","(021) 209.99.99"
"res_bank_2","TRUE","Anglo-Romanian Bank Limited, Anglia Londra","ARBLROBU","Bd. Carol I nr.34-36, sector 2","BUCURESTI","base.RO_B","base.ro","+44(0)207 826 4200"
"res_bank_3","TRUE","Ate Bank","MINDROBU","Bd.Grivitei, nr. 24, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 30.30.732"
"res_bank_4","TRUE","Banca Comercială Carpatica","CARPRO22","str. Autogarii nr.1","SIBIU","base.RO_SB","base.ro","(0269) 23.39.85"
"res_bank_5","TRUE","Banca Comercială Română","RNCBROBU","Bd.Regina Elisabeta nr.5, sector 3","BUCURESTI","base.RO_B","base.ro","0801 0801 227"
"res_bank_6","TRUE","Banca C.R. Firenze","DAROROBU","Bd. Unirii nr.55, bl.E4a, Tronson 1, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 201.19.30"
"res_bank_7","TRUE","Banca de Export-Import a României Eximbank","EXIMROBU","Spl. Independentei nr.15, sector 5","BUCURESTI","base.RO_B","base.ro","(021) 336.61.62"
"res_bank_8","TRUE","Banca di Roma","BROMROBU","Intrarea Murmurului nr. 2-4, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 232.08.18"
"res_bank_9","TRUE","Banca Italo Romena","BITRROBU","Bd. Dimitrie Cantemir nr.1, bl.B2, sc.2, parter si mezanin, sector 4","BUCURESTI","base.RO_B","base.ro","(021) 330.78.76"
"res_bank_10","TRUE","Banca Românească","BRMAROBU","bd.Unirii nr.35, bl.A3, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 321.16.01"
"res_bank_11","TRUE","Banca Transilvania","BTRLRO22","George Baritiu nr.8","CLUJ-NAPOCA","base.RO_CJ","base.ro","(0264) 407 150"
"res_bank_12","TRUE","Banc Post","BPOSROBU","Calea Vitan nr.6, 6A, Tronson B si C, et.3-7, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 308.09.01"
"res_bank_13","TRUE","Blom Bank Egypt","MIRBROBU","Bd. Unirii nr.66 bl. K 3 sector 3","BUCURESTI","base.RO_B","base.ro","(021) 302.72.00"
"res_bank_14","TRUE","BRD - Groupe Société Générale","BRDEROBU","Bd. Ion Mihalache nr.1-7, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 301.61.00"
"res_bank_15","TRUE","C.E.C.","CECEROBU","Calea Victoriei nr.13, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 311.11.19."
"res_bank_16","TRUE","Citibank România","CITIROBU","bd. Iancu de Hunedoara nr. 8, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 210.18.50"
"res_bank_17","TRUE","Credit Coop Casa Centrală","CRCOROBU","Calea Plevnei nr.200","BUCURESTI","base.RO_B","base.ro","(021) 317.74.05"
"res_bank_18","TRUE","Credit Europe Bank","FNNBROBU","Bd. Timisoara Nr. 26Z, Sector 6","BUCURESTI","base.RO_B","base.ro","(021) 406.40.00"
"res_bank_19","TRUE","Egnatia Bank","EGNAROBX","str. General Constantin Budisteanu nr.28C, P+1, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 303.21.00"
"res_bank_20","TRUE","Emporiki Bank - România","BSEAROBU","str.Berzei nr.19, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 310.39.55"
"res_bank_21","TRUE","GarantiBank International NV","UGBIROBU","str.Paris nr.30, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 230.84.30"
"res_bank_22","TRUE","HVB Banca pentru Locuinţe","HVBLROBU","str.Dr.Grigore Mora nr.37, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 300 11 22"
"res_bank_23","TRUE","ING Bank N.V., Amsterdam","INGBROBU","sos.Kiseleff nr.11-13, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 222.16.00"
"res_bank_24","TRUE","Leumi Bank România","DAFBRO22","B-dul Aviatorilor nr.45, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 206.70.75"
"res_bank_25","TRUE","Libra Bank","BRELROBU","str. dr. Grigore Mora nr.11, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 20.88.000"
"res_bank_26","TRUE","Millenium Bank","MILBROBU","Piaţa Presei Libere nr. 3-5, Clădirea City Gate, Turnul Sudic, parter si et. 13-17","BUCURESTI","base.RO_B","base.ro","(021) 308 13 00"
"res_bank_27","TRUE","OTP Bank România S.A.","OTPVROBU","str.Buzesti nr.66-68, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 307.57.00"
"res_bank_28","TRUE","Piraeus Bank România","PIRBROBU","bd.Carol I nr.34-36, et. VI, sector 2","BUCURESTI","base.RO_B","base.ro","(021) 250.67.98"
"res_bank_29","TRUE","Porsche Bank România","PORLROBU","sos.Pipera-Tunari nr.2, cladirea PORSCHE, parter, etaj 1 si 2","VOLUNTARI","base.RO_B","base.ro","(021) 208.26.00"
"res_bank_30","TRUE","ProCredit Bank","MIROROBU","str.Buzesti nr.62-64, et.1 si et.2, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 201.60.00"
"res_bank_31","TRUE","Raiffeisen Banca pentru Locuinţe","RZBLROBU","str. Nicolae Caramfil nr.79, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 233.30.00"
"res_bank_32","TRUE","Raiffeisen Bank","RZBRROBU","Piata Charles de Gaulle nr.15, et.4,5,6,7 si 8, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 323.00.31"
"res_bank_33","TRUE","RBS Bank România","ABNAROBU","Piata Montreal nr.10, WTCB - E etajul 2, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 20.20.400"
"res_bank_34","TRUE","Romanian International Bank","ROINROBU","bd.Unirii nr.68, bl. K2, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 323.10.35."
"res_bank_35","TRUE","Romexterra Bank","CRDZROBU","Bdul 1 Decembrie 1918 nr.93","TARGU MURES","base.RO_MS","base.ro","(0265) 16.66.41"
"res_bank_36","TRUE","Sanpaolo Imi Bank România","WBANRO22","str.Revolutiei nr.88","ARAD","base.RO_AR","base.ro","(0257) 30.82.00"
"res_bank_37","TRUE","Trezoreria Statului","TREZROBU","Splaiul Unirii 6-8","BUCURESTI","base.RO_B","base.ro","(021)  317.27.70"
"res_bank_38","TRUE","UniCredit Tiriac Bank","BACXROBU","Str. Ghetarilor nr.23-25, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 200.20.20"
"res_bank_39","TRUE","Volksbank Romania","VBBUROBU","sos. Mihai Bravu nr.171, sector 2","BUCURESTI","base.RO_B","base.ro","(021) 303.93.00"
"res_bank_40","TRUE","Banca Comercială Feroviară","BFERROBU","Strada Popa Tatu nr. 62A, Tronson A, Sector 1,","BUCURESTI","base.RO_B","base.ro","(021) 303.40.00"
"res_bank_41","TRUE","BCR Banca pentru Locuinţe","BCRLROBU","B-dul Lascar Catargiu, nr.47-53, sector 1","BUCURESTI","base.RO_B","base.ro","0801 080.275"
"res_bank_42","TRUE","Bank of Cyprus plc Nicosia","BCYPROBU","Calea Dorobanti nr.187B, sector 1","BUCURESTI","base.RO_B","base.ro","08000 877.777"
"res_bank_43","TRUE","BNP Paribas Fortis SA/NV","FTSBROBU","str. Banul Antonache nr.40-44, etaj 5, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 310.09.70"
"res_bank_44","TRUE","TBI Bank EAD Sofia","TBIBROBU","Str. Putul lui Zamfir, nr. 8-12, etaj 4, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 529.86.00"

```

## File: data\template\account.account-ro.csv

```csv
"id","name","code","account_type","reconcile","name@ro"
"pcg_1011","Unpaid subscribed capital","1011","equity","False","Capital subscris nevărsat"
"pcg_1012","Paid subscribed capital","1012","equity","False","Capital subscris vărsat"
"pcg_1015","Stat-owned patrimony","1015","equity","False","Patrimoniul regiei"
"pcg_1016","Public patrimony","1016","equity","False","Patrimoniul public"
"pcg_1017","Private patrimony","1017","equity","False","Patrimoniul privat"
"pcg_1018","Heritage national research and development institutes","1018","equity","False","Patrimoniul institutelor naționale de cercetare-dezvoltare"
"pcg_1031","Employee benefits in the form of equity instruments","1031","equity","False","Beneficii acordate angajaților sub forma instrumentelor de capitaluri proprii"
"pcg_1033","Exchange rate differences in relation to the net investment in a foreign entity","1033","equity","False","Diferențe de curs valutar în relație cu investiția netă într-o entitate străină"
"pcg_1038","Differences from changes in the fair value of available-for-sale financial assets and other equity items","1038","equity","False","Diferențe din modificarea valorii juste a activelor financiare disponibile în vederea vânzării și alte elemente de capitaluri proprii"
"pcg_1041","Premiums Issued","1041","equity","False","Prime de emisiune"
"pcg_1042","Merger/division premiums","1042","equity","False","Prime de fuziune/divizare"
"pcg_1043","Intake premiums","1043","equity","False","Prime de aport"
"pcg_1044","Bond-to-stock conversion premiums","1044","equity","False","Prime de conversie a obligațiunilor în acțiuni"
"pcg_105","Revaluation reserves","105","equity","False","Rezerve din reevaluare"
"pcg_1061","Legal reserves","1061","equity","False","Rezerve legale"
"pcg_1063","Statutory or contractual reserves","1063","equity","False","Rezerve statutare sau contractuale"
"pcg_1068","Other reservations","1068","equity","False","Alte rezerve"
"pcg_107","Exchange rate differences from conversion","107","equity","False","Diferențe de curs valutar din conversie"
"pcg_1081","Non-controlling interests - result of the financial year","1081","equity","False","Interese care nu controlează - rezultatul exercițiului financiar"
"pcg_1082","Non-controlling interests - other equity","1082","equity","False","Interese care nu controlează - alte capitaluri proprii"
"pcg_1091","Short-term own shares","1091","equity","False","Acțiuni proprii deținute pe termen scurt"
"pcg_1092","Long-term own shares","1092","equity","False","Acțiuni proprii deținute pe termen lung"
"pcg_1095","Own shares representing securities held by the absorbed company at the absorbing company","1095","equity","False","Acțiuni proprii reprezentând titluri deținute de societatea absorbită la societatea absorbantă"
"pcg_1171","Retained earnings representing undistributed profit or uncovered loss","1171","equity","False","Rezultatul reportat reprezentând profitul nerepartizat sau pierderea neacoperită"
"pcg_1172","Retained earnings from first-time adoption of IAS, less IAS 29","1172","equity","False","Rezultatul reportat provenit din adoptarea pentru prima dată a IAS, mai puțin IAS 29"
"pcg_1173","The carried forward result from changes in ng policies","1173","equity","False",""
"pcg_1174","The carried forward result from the correction of ng errors","1174","equity","False",""
"pcg_1175","The carried forward result representing the surplus realized from revaluation reserves","1175","equity","False","Rezultatul reportat reprezentând surplusul realizat din rezerve din reevaluare"
"pcg_1176","The carried over result from the transition to the application of ng regulations in accordance with European directives","1176","equity","False",""
"pcg_121","Profit and loss","121","equity_unaffected","False","Profit sau pierdere"
"pcg_129","Profit ditstribution","129","equity","False","Repartizarea profitului"
"pcg_1411","Gains on sale of equity instruments","1411","equity","False","Câștiguri legate de vânzarea instrumentelor de capitaluri proprii"
"pcg_1412","Gains on cancellation of equity instruments","1412","equity","False","Câștiguri legate de anularea instrumentelor de capitaluri proprii"
"pcg_1491","Losses from the sale of equity instruments","1491","equity","False","Pierderi rezultate din vânzarea instrumentelor de capitaluri proprii"
"pcg_1495","Losses resulting from reorganizations, which are determined by the cancellation of securities held","1495","equity","False","Pierderi rezultate din reorganizări, care sunt determinate de anularea titlurilor deținute"
"pcg_1498","Other losses related to equity instruments","1498","equity","False","Alte pierderi legate de instrumentele de capitaluri proprii"
"pcg_1511","Provisions for litigation","1511","liability_non_current","False","Provizioane pentru litigii"
"pcg_1512","Provisions for guarantees granted to customers","1512","liability_non_current","False","Provizioane pentru garanții acordate clienților"
"pcg_1513","Provisions for decommissioning tangible assets and other similar actions related to them","1513","liability_non_current","False","Provizioane pentru dezafectare imobilizări corporale și alte acțiuni similare legate de acestea"
"pcg_1514","Provisions for restructuring","1514","liability_non_current","False","Provizioane pentru restructurare"
"pcg_1515","Provisions for pensions and similar obligations","1515","liability_non_current","False","Provizioane pentru pensii și obligații similare"
"pcg_1516","Provisions for taxes","1516","liability_non_current","False","Provizioane pentru impozite"
"pcg_1517","Provisions for the termination of the employment contract","1517","liability_non_current","False","Provizioane pentru terminarea contractului de muncă"
"pcg_1518","Other provisions","1518","liability_non_current","False","Alte provizioane"
"pcg_16141","External loans from government-guaranteed bond issues - resumed in a period of up to one year","16141","liability_current","False","Împrumuturi externe din emisiuni de obligațiuni garantate de stat - reluate intr-o perioada de pana la un an"
"pcg_16142","External loans from government-guaranteed bond issues - resumed in a period of at least one year","16142","liability_current","False","Împrumuturi externe din emisiuni de obligațiuni garantate de stat - reluate intr-o perioada mal mare de un an"
"pcg_16151","Foreign loans from bond issues guaranteed by banks - resumed in a period of up to one year","16151","liability_current","False","Împrumuturi externe din emisiuni de obligațiuni garantate de bănci - reluate intr-o perioada de pana la un an"
"pcg_16152","Foreign loans from bond issues guaranteed by banks - resumed in a period of at least one year","16152","liability_current","False","Împrumuturi externe din emisiuni de obligațiuni garantate de bănci - reluate intr-o perioada mal mare de un an"
"pcg_16171","Domestic borrowing from government-guaranteed bond issues - resumed in a period of up to one year","16171","liability_current","False","Împrumuturi interne din emisiuni de obligațiuni garantate de stat - reluate intr-o perioada de pana la un an"
"pcg_16172","Domestic borrowing from government-guaranteed bond issues - resumed in a period of at least one year","16172","liability_current","False","Împrumuturi interne din emisiuni de obligațiuni garantate de stat - reluate intr-o perioada mal mare de un an"
"pcg_16181","Other loans from bond issues - resumed in a period of up to one year","16181","liability_current","False","Alte împrumuturi din emisiuni de obligațiuni - reluate intr-o perioada de pana la un an"
"pcg_16182","Other loans from bond issues - resumed in a period of at least one year","16182","liability_current","False","Alte împrumuturi din emisiuni de obligațiuni - reluate intr-o perioada mal mare de un an"
"pcg_16211","Long-term bank loans - resumed in a period of up to one year","16211","liability_current","False","Credite bancare pe termen lung - reluate intr-o perioada de pana la un an"
"pcg_16212","Long-term bank loans - resumed in a period of at least one year","16212","liability_current","False","Credite bancare pe termen lung - reluate intr-o perioada mal mare de un an"
"pcg_16221","Long-term bank loans outstanding at maturity - resumed in a period of up to one year","16221","liability_current","False","Credite bancare pe termen lung nerambursate la scadență - reluate intr-o perioada de pana la un an"
"pcg_16222","Long-term bank loans outstanding at maturity - resumed in a period of at least one year","16222","liability_current","False","Credite bancare pe termen lung nerambursate la scadență - reluate intr-o perioada mal mare de un an"
"pcg_16231","Foreign government loans - resumed in a period of up to one year","16231","liability_current","False","Credite externe guvernamentale - reluate intr-o perioada de pana la un an"
"pcg_16232","Foreign government loans - resumed in a period of at least one year","16232","liability_current","False","Credite externe guvernamentale - reluate intr-o perioada mal mare de un an"
"pcg_16241","Foreign bank loans guaranteed by the state - resumed in a period of up to one year","16241","liability_current","False","Credite bancare externe garantate de stat - reluate intr-o perioada de pana la un an"
"pcg_16242","Foreign bank loans guaranteed by the state - resumed in a period of at least one year","16242","liability_current","False","Credite bancare externe garantate de stat - reluate intr-o perioada mal mare de un an"
"pcg_16251","External bank loans guaranteed by banks - resumed in a period of up to one year","16251","liability_current","False","Credite bancare externe garantate de bănci - reluate intr-o perioada de pana la un an"
"pcg_16252","External bank loans guaranteed by banks - resumed in a period of at least one year","16252","liability_current","False","Credite bancare externe garantate de bănci - reluate intr-o perioada mal mare de un an"
"pcg_16261","Loans from the state treasury - resumed in a period of up to one year","16261","liability_current","False","Credite de la trezoreria statului - reluate intr-o perioada de pana la un an"
"pcg_16262","Loans from the state treasury - resumed in a period of at least one year","16262","liability_current","False","Credite de la trezoreria statului - reluate intr-o perioada mal mare de un an"
"pcg_16271","Domestic bank loans guaranteed by the state - resumed in a period of up to one year","16271","liability_current","False","Credite bancare interne garantate de stat - reluate intr-o perioada de pana la un an"
"pcg_16272","Domestic bank loans guaranteed by the state - resumed in a period of at least one year","16272","liability_current","False","Credite bancare interne garantate de stat - reluate intr-o perioada mal mare de un an"
"pcg_16611","Liabilities to related entities - resumed in a period of up to one year","16611","liability_current","False","Datorii față de entitățile afiliate - reluate intr-o perioada de pana la un an"
"pcg_16612","Liabilities to related entities - resumed in a period of at least one year","16612","liability_current","False","Datorii față de entitățile afiliate - reluate intr-o perioada mal mare de un an"
"pcg_16631","Liabilities to associates and jointly controlled entities - resumed in a period of up to one year","16631","liability_current","False","Datorii față de entitățile asociate și entitățile controlate în comun - reluate intr-o perioada de pana la un an"
"pcg_16632","Liabilities to associates and jointly controlled entities - resumed in a period of at least one year","16632","liability_current","False","Datorii față de entitățile asociate și entitățile controlate în comun - reluate intr-o perioada mal mare de un an"
"pcg_1671","Other loans and similar liabilities - resumed in a period of up to one year","1671","liability_current","False","Alte împrumuturi și datorii asimilate - reluate intr-o perioada de pana la un an"
"pcg_1672","Other loans and similar liabilities - resumed in a period of at least one year","1672","liability_current","False","Alte împrumuturi și datorii asimilate - reluate intr-o perioada mal mare de un an"
"pcg_16811","Interest on loans from bond issues - resumed in a period of up to one year","16811","liability_current","False","Dobânzi aferente împrumuturilor din emisiuni de obligațiuni - reluate intr-o perioada de pana la un an"
"pcg_16812","Interest on loans from bond issues - resumed in a period of at least one year","16812","liability_current","False","Dobânzi aferente împrumuturilor din emisiuni de obligațiuni - reluate intr-o perioada mal mare de un an"
"pcg_16821","Interest related to long-term bank loans - resumed in a period of up to one year","16821","liability_current","False","Dobânzi aferente creditelor bancare pe termen lung - reluate intr-o perioada de pana la un an"
"pcg_16822","Interest related to long-term bank loans - resumed in a period of at least one year","16822","liability_current","False","Dobânzi aferente creditelor bancare pe termen lung - reluate intr-o perioada mal mare de un an"
"pcg_16851","Interest related to debts to related entities - resumed in a period of up to one year","16851","liability_current","False","Dobânzi aferente datoriilor față de entitățile afiliate - reluate intr-o perioada de pana la un an"
"pcg_16852","Interest related to debts to related entities - resumed in a period of at least one year","16852","liability_current","False","Dobânzi aferente datoriilor față de entitățile afiliate - reluate intr-o perioada mal mare de un an"
"pcg_16861","Interest on liabilities to associates and jointly controlled entities - resumed in a period of up to one year","16861","liability_current","False","Dobânzi aferente datoriilor fată de entitătile asociate și entitățile controlate în comun - reluate intr-o perioada de pana la un an"
"pcg_16862","Interest on liabilities to associates and jointly controlled entities - resumed in a period of at least one year","16862","liability_current","False","Dobânzi aferente datoriilor fată de entitătile asociate și entitățile controlate în comun - reluate intr-o perioada mal mare de un an"
"pcg_16871","Interest related to other loans and similar liabilities - resumed in a period of up to one year","16871","liability_current","False","Dobânzi aferente altor împrumuturi și datorii asimilate - reluate intr-o perioada de pana la un an"
"pcg_16872","Interest related to other loans and similar liabilities - resumed in a period of at least one year","16872","liability_current","False","Dobânzi aferente altor împrumuturi și datorii asimilate - reluate intr-o perioada de pana la un an"
"pcg_16911","Bond redemption premiums - resumed in a period of up to one year","16911","liability_current","False","Prime privind rambursarea obligațiunilor - reluate intr-o perioada de pana la un an"
"pcg_16912","Bond redemption premiums - resumed in a period of at least one year","16912","liability_current","False","Prime privind rambursarea obligațiunilor - reluate intr-o perioada de pana la un an"
"pcg_16921","Premiums on repayment of other liabilities - resumed in a period of up to one year","16921","liability_current","False","Prime privind rambursarea altor datorii - reluate intr-o perioada de pana la un an"
"pcg_16922","Premiums on repayment of other liabilities - resumed in a period of at least one year","16922","liability_current","False","Prime privind rambursarea altor datorii - reluate intr-o perioada mal mare de un an"
"pcg_201","Formation expenses","201","asset_non_current","False","Cheltuieli de constituire"
"pcg_203","Development expenses","203","asset_non_current","False","Cheltuieli de dezvoltare"
"pcg_205","Concessions, patents, licenses, trademarks, rights and similar assets","205","asset_non_current","False","Concesiuni, brevete, licențe, mărci comerciale, drepturi și active similare"
"pcg_206","Intangible assets for exploration and evaluation of mineral resources","206","asset_non_current","False","Active necorporale de explorare și evaluare a resurselor minerale"
"pcg_2071","Positive goodwill","2071","asset_non_current","False","Fond comercial pozitiv"
"pcg_2075","Negative goodwill","2075","liability_non_current","False","Fond comercial negativ"
"pcg_208","Other intangible assets","208","asset_non_current","False","Alte imobilizări necorporale"
"pcg_2111","Land","2111","asset_fixed","False","Terenuri"
"pcg_2112","landscaping","2112","asset_fixed","False","Amenajări de terenuri"
"pcg_212","Construction","212","asset_fixed","False","Construcții"
"pcg_2131","Technological equipment (machines, machines and work installations)","2131","asset_fixed","False","Echipamente tehnologice (mașini, utilaje și instalații de lucru)"
"pcg_2132","Measuring, control and regulation apparatus and installations","2132","asset_fixed","False","Aparate și instalații de măsurare, control și reglare"
"pcg_2133","Means of transport","2133","asset_fixed","False","Mijloace de transport"
"pcg_214","Furniture, office equipment, human asset protection equipment and materials and other tangible assets","214","asset_fixed","False","Mobilier, aparatură birotică, echipamente de protecție a valorilor umane și materiale și alte active corporale"
"pcg_215","Real estate investments","215","asset_fixed","False","Investiții imobiliare"
"pcg_216","Tangible assets for exploration and evaluation of mineral resources","216","asset_fixed","False","Active corporale de explorare și evaluare a resurselor minerale"
"pcg_217","Productive biological assets","217","asset_fixed","False","Active biologice productive"
"pcg_223","Technical installations and means of transport being supplied","223","asset_fixed","False","Instalații tehnice și mijloace de transport în curs de aprovizionare"
"pcg_224","Furniture, office equipment, human asset protection equipment and materials and other tangible assets being supplied","224","asset_fixed","False","Mobilier, aparatură birotică, echipamente de protecție a valorilor umane și materiale și alte active corporale în curs de aprovizionare"
"pcg_227","Productive biological assets being supplied","227","asset_non_current","False","Active biologice productive în curs de aprovizionare"
"pcg_231","Property, plant and equipment in progress","231","asset_non_current","False","Imobilizări corporale în curs de execuție"
"pcg_235","Real estate investments in progress","235","asset_non_current","False","Investiții imobiliare în curs de execuție"
"pcg_261","Shares held in affiliated entities","261","asset_fixed","False","Acțiuni deținute la entitățile afiliate"
"pcg_262","Shares held in associated entities","262","asset_fixed","False","Acțiuni deținute la entități asociate"
"pcg_263","Shares held in jointly controlled entities","263","asset_fixed","False","Acțiuni deținute la entități controlate în comun"
"pcg_264","Equivalent securities","264","asset_fixed","False","Titluri puse în echivalență"
"pcg_265","Other fixed assets","265","asset_fixed","False","Alte titluri imobilizate"
"pcg_266","Deferred Green Certificates","266","asset_fixed","False","Certificate verzi amânate"
"pcg_2671","Amounts receivable from affiliated entities","2671","asset_non_current","False","Sume de încasat de la entitățile afiliate"
"pcg_2672","Interest on receivables from affiliated entities","2672","asset_non_current","False","Dobânda aferentă sumelor de încasat de la entitățile afiliate"
"pcg_2673","Receivables from associates and jointly controlled entities","2673","liability_non_current","False","Creanțe față de entitățile asociate și entitățile controlate în comun"
"pcg_2674","Interest related to receivables from associated entities and jointly controlled entities","2674","liability_non_current","False","Dobânda aferentă creanțelor față de entitățile asociate și entitățile controlate în comun"
"pcg_2675","Long-term loans","2675","asset_non_current","False","Împrumuturi acordate pe termen lung"
"pcg_26751","Long-term loans - receivables","26751","asset_non_current","True","Împrumuturi acordate pe termen lung - creanțe"
"pcg_2676","Interest related to long-term loans","2676","asset_non_current","False","Dobânda aferentă împrumuturilor acordate pe termen lung"
"pcg_26761","Interest related to long-term loans - receivables","26761","asset_non_current","True","Dobânda aferentă împrumuturilor acordate pe termen lung - creanțe"
"pcg_2677","Bonds purchased on the occasion of issues carried out by third parties","2677","asset_non_current","False","Obligațiuni achiziționate cu ocazia emisiunilor efectuate de terți"
"pcg_2678","Other immobilized receivables","2678","liability_non_current","False","Alte creanțe imobilizate"
"pcg_2679","Interest related to other immobilized receivables","2679","liability_non_current","False","Dobânzi aferente altor creanțe imobilizate"
"pcg_26911","Payments to be made regarding shares held in affiliated entities - resumed in a period of up to one year","26911","liability_non_current","False","Vărsăminte de efectuat privind acțiunile deținute la entitățile afiliate - reluate intr-o perioada de pana la un an"
"pcg_26912","Payments to be made regarding shares held in affiliated entities - resumed in a period of at least one year","26912","liability_non_current","False","Vărsăminte de efectuat privind acțiunile deținute la entitățile afiliate - reluate intr-o perioada mal mare de un an"
"pcg_26921","Payments to be made regarding shares held in associated entities - resumed in a period of up to one year","26921","liability_non_current","False","Vărsăminte de efectuat privind acțiunile deținute la entități asociate - reluate intr-o perioada de pana la un an"
"pcg_26922","Payments to be made regarding shares held in associated entities - resumed in a period of at least one year","26922","liability_non_current","False","Vărsăminte de efectuat privind acțiunile deținute la entități asociate - reluate intr-o perioada mal mare de un an"
"pcg_26931","Payments to be made regarding shares held in jointly controlled entities - resumed in a period of up to one year","26931","liability_non_current","False","Vărsăminte de efectuat privind acțiunile deținute la entități controlate în comun - reluate intr-o perioada de pana la un an"
"pcg_26932","Payments to be made regarding shares held in jointly controlled entities - resumed in a period of at least one year","26932","liability_non_current","False","Vărsăminte de efectuat privind acțiunile deținute la entități asociate - reluate intr-o perioada mal mare de un an"
"pcg_26951","Payments to be made for other financial assets - resumed in a period of up to one year","26951","liability_non_current","False","Vărsăminte de efectuat pentru alte imobilizări financiare - reluate intr-o perioada de pana la un an"
"pcg_26952","Payments to be made for other financial assets - resumed in a period of at least one year","26952","liability_non_current","False","Vărsăminte de efectuat pentru alte imobilizări financiare - reluate intr-o perioada mal mare de un an"
"pcg_2801","Depreciation of formation expenses","2801","expense_depreciation","False","Amortizarea cheltuielilor de constituire"
"pcg_2803","Amortization of development expenses","2803","expense_depreciation","False","Amortizarea cheltuielilor de dezvoltare"
"pcg_2805","Amortization of concessions, patents, licenses, trademarks, rights and similar assets","2805","expense_depreciation","False","Amortizarea concesiunilor, brevetelor, licențelor, mărcilor comerciale, drepturilor și activelor similare"
"pcg_2806","Amortization of mineral resource exploration and evaluation intangible assets","2806","expense_depreciation","False","Amortizarea activelor necorporale de explorare și evaluare a resurselor minerale"
"pcg_2807","Amortization of goodwill","2807","expense_depreciation","False","Amortizarea fondului comercial"
"pcg_2808","Amortization of other intangible assets","2808","expense_depreciation","False","Amortizarea altor imobilizări necorporale"
"pcg_2811","Depreciation of land improvements","2811","expense_depreciation","False","Amortizarea amenajărilor de terenuri"
"pcg_2812","Depreciation of buildings","2812","expense_depreciation","False","Amortizarea construcțiilor"
"pcg_2813","Depreciation of installations and means of transport","2813","expense_depreciation","False","Amortizarea instalațiilor și mijloacelor de transport"
"pcg_2814","Depreciation of other tangible assets","2814","expense_depreciation","False","Amortizarea altor imobilizări corporale"
"pcg_2815","Depreciation of real estate investments","2815","expense_depreciation","False","Amortizarea investițiilor imobiliare"
"pcg_2816","Depreciation of property, plant and equipment for exploration and evaluation of mineral resources","2816","expense_depreciation","False","Amortizarea activelor corporale de explorare și evaluare a resurselor minerale"
"pcg_2817","Depreciation of productive biological assets","2817","expense_depreciation","False","Amortizarea activelor biologice productive"
"pcg_2903","Adjustments for impairment of development expenses","2903","expense_depreciation","False","Ajustări pentru deprecierea cheltuielilor de dezvoltare"
"pcg_2905","Adjustments for impairment of concessions, patents, licenses, trademarks, rights and similar assets","2905","expense_depreciation","False","Ajustări pentru deprecierea concesiunilor, brevetelor, licențelor, mărcilor comerciale, drepturilor și activelor similare"
"pcg_2906","Adjustments for impairment of mineral resource exploration and evaluation intangible assets","2906","expense_depreciation","False","Ajustări pentru deprecierea activelor necorporale de explorare și evaluare a resurselor minerale"
"pcg_2908","Adjustments for impairment of other intangible assets","2908","expense_depreciation","False","Ajustări pentru deprecierea altor imobilizări necorporale"
"pcg_2911","Adjustments for depreciation of land and land improvements","2911","expense_depreciation","False","Ajustări pentru deprecierea terenurilor și amenajărilor de terenuri"
"pcg_2912","Adjustments for the depreciation of buildings","2912","expense_depreciation","False","Ajustări pentru deprecierea construcțiilor"
"pcg_2913","Adjustments for depreciation of plant and equipment","2913","expense_depreciation","False","Ajustări pentru deprecierea instalațiilor și mijloacelor de transport"
"pcg_2914","Adjustments for impairment of other tangible fixed assets","2914","expense_depreciation","False","Ajustări pentru deprecierea altor imobilizări corporale"
"pcg_2915","Adjustments for impairment of investment properties","2915","expense_depreciation","False","Ajustări pentru deprecierea investițiilor imobiliare"
"pcg_2916","Adjustments for impairment of property, plant and equipment for exploration and evaluation of mineral resources","2916","expense_depreciation","False","Ajustări pentru deprecierea activelor corporale de explorare și evaluare a resurselor minerale"
"pcg_2917","Adjustments for impairment of productive biological assets","2917","expense_depreciation","False","Ajustări pentru deprecierea activelor biologice productive"
"pcg_2931","Adjustments for impairment of property, plant and equipment in progress","2931","expense_depreciation","False","Ajustări pentru deprecierea imobilizărilor corporale în curs de execuție"
"pcg_2935","Adjustments for the depreciation of their real estate investments in progress","2935","expense_depreciation","False","Ajustări pentru deprecierea investiții lor imobiliare în curs de execuție"
"pcg_2961","Adjustments for impairment of shares held in affiliated entities","2961","expense_depreciation","False","Ajustări pentru pierderea de valoare a acțiunilor deținute la entitățile afiliate"
"pcg_2962","Adjustments for impairment of shares held in associates and jointly controlled entities","2962","expense_depreciation","False","Ajustări pentru pierderea de valoare a acțiunilor deținute la entități asociate și entități controlate în comun"
"pcg_2963","Impairment adjustments for other fixed assets","2963","expense_depreciation","False","Ajustări pentru pierderea de valoare a altor titluri imobilizate"
"pcg_2964","Adjustments for impairment of receivables from related entities","2964","expense_depreciation","False","Ajustări pentru pierderea de valoare a sumelor de încasat de la entitățile afiliate"
"pcg_2965","Adjustments for impairment of receivables from associates and jointly controlled entities","2965","expense_depreciation","False","Ajustări pentru pierderea de valoare a creanțelor față de entitățile asociate și entitățile controlate în comun"
"pcg_2966","Adjustments for impairment of long-term loans","2966","expense_depreciation","False","Ajustări pentru pierderea de valoare a împrumuturilor acordate pe termen lung"
"pcg_29661","Adjustments for impairment of long-term loans - receivables","29661","expense_depreciation","True","Ajustări pentru pierderea de valoare a împrumuturilor acordate pe termen lung - creanțe"
"pcg_2968","Adjustments for impairment of other receivables","2968","expense_depreciation","False","Ajustări pentru pierderea de valoare a altor creanțe"
"pcg_29681","Adjustments for impairment of other receivables - receivables","29681","expense_depreciation","False","Ajustări pentru pierderea de valoare a altor creanțe - creanțe"
"pcg_301","Raw materials","301","asset_current","False","Materii prime"
"pcg_3021","Auxiliary materials","3021","asset_current","False","Materiale auxiliare"
"pcg_3022","Fuels","3022","asset_current","False","Combustibili"
"pcg_3023","Packaging materials","3023","asset_current","False","Materiale pentru ambalat"
"pcg_3024","Spare parts","3024","asset_current","False","Piese de schimb"
"pcg_3025","Seeds and planting material","3025","asset_current","False","Semințe și materiale de plantat"
"pcg_3026","Animal feed","3026","asset_current","False","Furaje"
"pcg_3028","Other consumables","3028","asset_current","False","Alte materiale consumabile"
"pcg_303","Inventory items materials","303","asset_current","False","Materiale de natura obiectelor de inventar"
"pcg_308","Price differences in raw materials and materials","308","asset_current","False","Diferențe de preț la materii prime și materiale"
"pcg_321","Raw materials being supplied","321","asset_current","False","Materii prime în curs de aprovizionare"
"pcg_322","Consumables in stock","322","asset_current","False","Materiale consumabile în curs de aprovizionare"
"pcg_323","Inventory items materials being supplied","323","asset_current","False","Materiale de natura obiectelor de inventar în curs de aprovizionare"
"pcg_326","Biological assets being supplied","326","asset_current","False","Active biologice de natura stocurilor în curs de aprovizionare"
"pcg_327","Goods being supplied","327","asset_current","False","Mărfuri în curs de aprovizionare"
"pcg_328","Packaging in progress","328","asset_current","False","Ambalaje în curs de aprovizionare"
"pcg_331","Products in progress","331","asset_current","False","Produse în curs de execuție"
"pcg_332","Services in progress","332","asset_current","False","Servicii în curs de execuție"
"pcg_341","Semi-finished products","341","asset_current","False","Semifabricate"
"pcg_345","Finished products","345","asset_current","False","Produse finite"
"pcg_346","Products residues","346","asset_current","False","Produse reziduale"
"pcg_347","Agricultural products","347","asset_current","False","Produse agricole"
"pcg_348","Product price differences","348","asset_current","False","Diferențe de preț la produse"
"pcg_3481","Product price differences - in progress","3481","asset_current","False","Diferențe de preț la produse - în curs"
"pcg_3482","Product price differences - completed","3482","asset_current","False","Diferențe de preț la produse - terminat"
"pcg_351","Materials and third-party materials","351","asset_current","False","Materii și materiale aflate la terți"
"pcg_354","Third-party products","354","asset_current","False","Produse aflate la terți"
"pcg_356","Biological assets held by third parties","356","asset_current","False","Active biologice de natura stocurilor aflate la terți"
"pcg_357","Goods held by third parties","357","asset_current","False","Mărfuri aflate la terți"
"pcg_358","Third-party packaging","358","asset_current","False","Ambalaje aflate la terți"
"pcg_361","Biological assets","361","asset_current","False","Active biologice de natura stocurilor"
"pcg_368","Price differences in biological assets of a stock nature","368","asset_current","False","Diferențe de preț la active biologice de natura stocurilor"
"pcg_371","Merchandise","371","asset_current","False","Mărfuri"
"pcg_378","Price differences in goods","378","asset_current","False","Diferențe de preț la mărfuri"
"pcg_381","Packaging","381","asset_current","False","Ambalaje"
"pcg_388","Price differences in packaging","388","asset_current","False","Diferențe de preț la ambalaje"
"pcg_391","Adjustments for depreciation of raw materials","391","asset_current","False","Ajustări pentru deprecierea materiilor prime"
"pcg_3921","Adjustments for the depreciation of consumables","3921","asset_current","False","Ajustări pentru deprecierea materialelor consumabile"
"pcg_3922","Adjustments for the depreciation of materials in the nature of inventory items","3922","asset_current","False","Ajustări pentru deprecierea materialelor de natura obiectelor de inventar"
"pcg_393","Adjustments for impairment of work in progress","393","asset_current","False","Ajustări pentru deprecierea producției în curs de execuție"
"pcg_3941","Adjustments for depreciation of semi-finished products","3941","asset_current","False","Ajustări pentru deprecierea semifabricatelor"
"pcg_3945","Adjustments for the depreciation of finished goods","3945","asset_current","False","Ajustări pentru deprecierea produselor finite"
"pcg_3946","Adjustments for the depreciation of residual products","3946","asset_current","False","Ajustări pentru deprecierea produselor reziduale"
"pcg_3947","Adjustments for the depreciation of agricultural products","3947","asset_current","False","Ajustări pentru deprecierea produselor agricole"
"pcg_3951","Adjustments for the impairment of materials and materials held by third parties","3951","asset_current","False","Ajustări pentru deprecierea materiilor și materialelor aflate la terți"
"pcg_3952","Adjustments for the depreciation of semi-finished products held by third parties","3952","asset_current","False","Ajustări pentru deprecierea semifabricatelor aflate la terți"
"pcg_3953","Adjustments for impairment of finished goods held by third parties","3953","asset_current","False","Ajustări pentru deprecierea produselor finite aflate la terți"
"pcg_3954","Adjustments for impairment of residual products held by third parties","3954","asset_current","False","Ajustări pentru deprecierea produselor reziduale aflate la terți"
"pcg_3955","Adjustments for the depreciation of agricultural products held by third parties","3955","asset_current","False","Ajustări pentru deprecierea produselor agricole aflate la terți"
"pcg_3956","Adjustments for the impairment of biological assets held by third parties","3956","asset_current","False","Ajustări pentru deprecierea activelor biologice de natura stocurilor aflate la terți"
"pcg_3957","Adjustments for impairment of goods held by third parties","3957","asset_current","False","Ajustări pentru deprecierea mărfurilor aflate la terți"
"pcg_3958","Adjustments for impairment of third-party packaging","3958","asset_current","False","Ajustări pentru deprecierea ambalajelor aflate la terți"
"pcg_396","Adjustments for the impairment of biological assets","396","asset_current","False","Ajustări pentru deprecierea activelor biologice de natura stocurilor"
"pcg_397","Adjustments for depreciation of goods","397","asset_current","False","Ajustări pentru deprecierea mărfurilor"
"pcg_398","Adjustments for depreciation of packaging","398","asset_current","False","Ajustări pentru deprecierea ambalajelor"
"pcg_4011","Providers - resumed in a period of up to one year","4011","liability_payable","True","Furnizori - reluate intr-o perioada de pana la un an"
"pcg_4012","Providers - resumed in a period of at least one year","4012","liability_payable","True","Furnizori - reluate intr-o perioada mal mare de un an"
"pcg_4031","Notes to be paid - resumed in a period of up to one year","4031","liability_payable","True","Efecte de plătit - reluate intr-o perioada de pana la un an"
"pcg_4032","Notes to be paid - resumed in a period of at least one year","4032","liability_payable","True","Efecte de plătit - reluate intr-o perioada mal mare de un an"
"pcg_4041","Fixed asset providers - resumed in a period of up to one year","4041","liability_payable","True","Furnizori de imobilizări - reluate intr-o perioada de pana la un an"
"pcg_4042","Fixed asset providers - resumed in a period of at least one year","4042","liability_payable","True","Furnizori de imobilizări - reluate intr-o perioada mal mare de un an"
"pcg_4051","Notes payable for fixed assets - resumed in a period of up to one year","4051","liability_current","True","Efecte de plătit pentru imobilizări - reluate intr-o perioada de pana la un an"
"pcg_4052","Notes payable for fixed assets - resumed in a period of at least one year","4052","liability_current","True","Efecte de plătit pentru imobilizări - reluate intr-o perioada mal mare de un an"
"pcg_4081","Providers - unpaid invoices - resumed in a period of up to one year","4081","liability_current","True","Furnizori - facturi nesosite - reluate intr-o perioada de pana la un an"
"pcg_4082","Providers - unpaid invoices - resumed in a period of at least one year","4082","liability_current","True","Furnizori - facturi nesosite - reluate intr-o perioada mal mare de un an"
"pcg_4091","Providers - debtors for purchases of goods of the nature of stocks","4091","liability_current","True","Furnizori - debitori pentru cumpărări de bunuri de natura stocurilor"
"pcg_4092","Providers - debtors for services","4092","liability_current","True","Furnizori - debitori pentru prestări de servicii"
"pcg_4093","Advances granted for tangible assets","4093","liability_current","True","Avansuri acordate pentru imobilizări corporale"
"pcg_4094","Advance payments for intangible assets","4094","liability_current","True","Avansuri acordate pentru imobilizări necorporale"
"ro_pcg_recv","Customers","4111","asset_receivable","True","Clienți"
"pcg_4118","Uncertain or litigious customers","4118","asset_current","True","Clienți incerți sau în litigiu"
"pcg_413","Notes to be received from customers","413","asset_current","True","Efecte de primit de la clienți"
"pcg_418","Customers - invoices to be issued","418","asset_current","True","Clienți - facturi de întocmit"
"pcg_419","Customers","419","asset_current","True","Clienți"
"pcg_4191","Customers - creditor - resumed in a period of up to one year","4191","asset_current","True","Clienți - creditori - reluate intr-o perioada de pana la un an"
"pcg_4192","Customers - creditor - resumed in a period of at least one year","4192","asset_current","True","Clienți - creditori - reluate intr-o perioada mal mare de un an"
"pcg_421","Staff - wages due","421","liability_payable","True","Personal - salarii datorate"
"pcg_4211","Staff - wages due - resumed in a period of up to one year","4211","liability_payable","True","Personal - salarii datorate - reluate intr-o perioada de pana la un an"
"pcg_4212","Staff - wages due - resumed in a period of at least one year","4212","liability_payable","True","Personal - salarii datorate - reluate intr-o perioada mal mare de un an"
"pcg_4231","Staff - material aid owed - resumed in a period of up to one year","4231","liability_current","False","Personal - ajutoare materiale datorate - reluate intr-o perioada de pana la un an"
"pcg_4232","Staff - material aid owed - resumed in a period of at least one year","4232","liability_current","False","Personal - ajutoare materiale datorate - reluate intr-o perioada mal mare de un an"
"pcg_4241","Premiums representing staff profit sharing - resumed in a period of up to one year","4241","liability_current","False","Prime reprezentând participarea personalului la profit - reluate intr-o perioada de pana la un an"
"pcg_4242","Premiums representing staff profit sharing - resumed in a period of at least one year","4242","liability_current","False","Prime reprezentând participarea personalului la profit - reluate intr-o perioada mal mare de un an"
"pcg_425","Advances granted to staff","425","asset_receivable","True","Avansuri acordate personalului"
"pcg_4261","Personnel rights not raised - resumed in a period of up to one year","4261","liability_current","False","Drepturi de personal neridicate - reluate intr-o perioada de pana la un an"
"pcg_4262","Personnel rights not raised - resumed in a period of at least one year","4262","liability_current","False","Drepturi de personal neridicate - reluate intr-o perioada mal mare de un an"
"pcg_4271","Withholdings from wages owed to third parties - resumed in a period of up to one year","4271","liability_current","True","Rețineri din salarii datorate terților - reluate intr-o perioada de pana la un an"
"pcg_4272","Withholdings from wages owed to third parties - resumed in a period of at least one year","4272","liability_current","True","Rețineri din salarii datorate terților - reluate intr-o perioada mal mare de un an"
"pcg_42811","Other liabilities related to personnel - resumed in a period of up to one year","42811","liability_current","False","Alte datorii în legătură cu personalul - reluate intr-o perioada de pana la un an"
"pcg_42812","Other liabilities related to personnel - resumed in a period of at least one year","42812","liability_current","False","Alte datorii în legătură cu personalul - reluate intr-o perioada mal mare de un an"
"pcg_4282","Other receivables related to personnel","4282","asset_current","False","Alte creanțe în legătură cu personalul"
"pcg_43111","Unit contribution to social security - resumed in a period of up to one year","43111","liability_current","False","Contribuția unității la asigurările sociale - reluate intr-o perioada de pana la un an"
"pcg_43112","Unit contribution to social security - resumed in a period of at least one year","43112","liability_current","False","Contribuția unității la asigurările sociale - reluate intr-o perioada mal mare de un an"
"pcg_43121","Staff contribution to social security - resumed in a period of up to one year","43121","liability_current","False","Contribuția personalului la asigurările sociale - reluate intr-o perioada de pana la un an"
"pcg_43122","Staff contribution to social security - resumed in a period of at least one year","43122","liability_current","False","Contribuția personalului la asigurările sociale - reluate intr-o perioada mal mare de un an"
"pcg_43131","Employer contribution for social health insurance - resumed in a period of up to one year","43131","liability_current","False","Contribuția angajatorului pentru asigurările sociale de sănătate - reluate intr-o perioada de pana la un an"
"pcg_43132","Employer contribution for social health insurance - resumed in a period of at least one year","43132","liability_current","False","Contribuția angajatorului pentru asigurările sociale de sănătate - reluate intr-o perioada mal mare de un an"
"pcg_43141","Employee contribution for social health insurance - resumed in a period of up to one year","43141","liability_current","False","Contribuția angajaților pentru asigurările sociale de sănătate - reluate intr-o perioada de pana la un an"
"pcg_43142","Employee contribution for social health insurance - resumed in a period of at least one year","43142","liability_current","False","Contribuția angajaților pentru asigurările sociale de sănătate - reluate intr-o perioada mal mare de un an"
"pcg_43151","Social security contribution - resumed in a period of up to one year","43151","liability_current","False","Contribuția de asigurări sociale - reluate intr-o perioada de pana la un an"
"pcg_43152","Social security contribution - resumed in a period of at least one year","43152","liability_current","False","Contribuția de asigurări sociale - reluate intr-o perioada mal mare de un an"
"pcg_43161","Social insurance health contribution - resumed in a period of up to one year","43161","liability_current","False","Contribuția de asigurări sociale de sănătate - reluate intr-o perioada de pana la un an"
"pcg_43162","Social insurance health contribution - resumed in a period of at least one year","43162","liability_current","False","Contribuția de asigurări sociale de sănătate - reluate intr-o perioada mal mare de un an"
"pcg_43181","Other social health insurance contributions - resumed in a period of up to one year","43181","liability_current","False","Alte contribuții pentru asigurările sociale de sănătate - reluate intr-o perioada de pana la un an"
"pcg_43182","Other social health insurance contributions - resumed in a period of at least one year","43182","liability_current","False","Alte contribuții pentru asigurările sociale de sănătate - reluate intr-o perioada mal mare de un an"
"pcg_4361","Insurance contribution for work - resumed in a period of up to one year","4361","liability_current","False","Contributia asiguratorie pentru munca - reluate intr-o perioada de pana la un an"
"pcg_4362","Insurance contribution for work - resumed in a period of at least one year","4362","liability_current","False","Contributia asiguratorie pentru munca - reluate intr-o perioada mal mare de un an"
"pcg_4371","The unit's contribution to the unemployment fund","4371","liability_current","False","Contribuția unității la fondul de șomaj"
"pcg_4372","Staff contribution to the unemployment fund","4372","liability_current","False","Contribuția personalului la fondul de șomaj"
"pcg_43811","Other social liabilities - resumed in a period of up to one year","43811","liability_current","False","Alte datorii sociale - reluate intr-o perioada de pana la un an"
"pcg_43812","Other social liabilities - resumed in a period of at least one year","43812","liability_current","False","Alte datorii sociale - reluate intr-o perioada mal mare de un an"
"pcg_4382","Other social receivables","4382","asset_current","False","Alte creanțe sociale"
"pcg_44111","Profit tax - resumed in a period of up to one year","44111","liability_current","False","Impozitul pe profit - reluate intr-o perioada de pana la un an"
"pcg_44112","Profit tax - resumed in a period of at least one year","44112","liability_current","False","Impozitul pe profit - reluate intr-o perioada mal mare de un an"
"pcg_44151","Tax specific to activities - resumed in a period of up to one year","44151","liability_current","False","Impozitul specific unor activități - reluate intr-o perioada de pana la un an"
"pcg_44152","Tax specific to activities - resumed in a period of at least one year","44152","liability_current","False","Impozitul specific unor activități - reluate intr-o perioada mal mare de un an"
"pcg_44181","Income tax - resumed in a period of up to one year","44118","liability_current","False","Impozitul pe venit - reluate intr-o perioada de pana la un an"
"pcg_44182","Income tax - resumed in a period of at least one year","44182","liability_current","False","Impozitul pe venit - reluate intr-o perioada mal mare de un an"
"pcg_44231","Payment VAT - resumed in a period of up to one year","44231","liability_current","False","TVA de plată - reluate intr-o perioada de pana la un an"
"pcg_44232","Payment VAT - resumed in a period of at least one year","44232","liability_current","False","TVA de plată - reluate intr-o perioada mal mare de un an"
"pcg_4424","VAT to be recovered","4424","liability_current","False","TVA de recuperat"
"pcg_4426","VAT deductible","4426","asset_current","False","TVA deductibilă"
"pcg_4427","VAT collected","4427","liability_current","False","TVA colectată"
"pcg_44281","Non-chargeable VAT - Collected","44281","liability_current","False","TVA neexigibilă - Colectată"
"pcg_442811","Non-chargeable VAT - Collected - resumed in a period of up to one year","442811","liability_current","False","TVA neexigibilă - Colectată - reluate intr-o perioada de pana la un an"
"pcg_442812","Non-chargeable VAT - Collected - resumed in a period of at least one year","442812","liability_non_current","False","TVA neexigibilă - Colectată - reluate intr-o perioada mal mare de un an"
"pcg_44282","Non-chargeable VAT - Deducted","44282","asset_current","False","TVA neexigibilă - Deductibilă"
"pcg_442821","Non-chargeable VAT - Deducted - resumed in a period of up to one year","442821","asset_current","False","TVA neexigibilă - Deductibilă - reluate intr-o perioada de pana la un an"
"pcg_442822","Non-chargeable VAT - Deducted - resumed in a period of at least one year","442822","asset_current","False","TVA neexigibilă - Deductibilă - reluate intr-o perioada mal mare de un an"
"pcg_4441","Income tax on wages - resumed in a period of up to one year","4441","liability_current","True","Impozitul pe venituri de natura salariilor - reluate intr-o perioada de pana la un an"
"pcg_4442","Income tax on wages - resumed in a period of at least one year","4442","liability_current","True","Impozitul pe venituri de natura salariilor - reluate intr-o perioada mal mare de un an"
"pcg_4451","Government subsidies","4451","asset_non_current","False","Subvenții guvernamentale"
"pcg_4452","Grant-in-kind loans","4452","asset_current","False","Împrumuturi nerambursabile cu caracter de subvenții"
"pcg_4458","Other amounts received as subsidies","4458","asset_current","False","Alte sume primite cu caracter de subvenții"
"pcg_4461","Other taxes, fees and similar payments - resumed in a period of at least one year","4461","liability_current","True","Alte impozite, taxe și vărsăminte asimilate - reluate intr-o perioada mal mare de un an"
"pcg_4462","Other taxes, fees and similar payments - resumed in a period of up to one year","4462","liability_current","True","Alte impozite, taxe și vărsăminte asimilate - reluate intr-o perioada de pana la un an"
"pcg_4471","Special funds - taxes and similar payments - resumed in a period of up to one year","4471","liability_current","False","Fonduri speciale - taxe și vărsăminte asimilate - reluate intr-o perioada de pana la un an"
"pcg_4472","Special funds - taxes and similar payments - resumed in a period of at least one year","4472","liability_current","False","Fonduri speciale - taxe și vărsăminte asimilate - reluate intr-o perioada mal mare de un an"
"pcg_44811","Other liabilities to the state budget - resumed in a period of up to one year","44811","liability_current","False","Alte datorii față de bugetul statului - reluate intr-o perioada de pana la un an"
"pcg_44812","Other liabilities to the state budget - resumed in a period of at least one year","44812","liability_current","False","Alte datorii față de bugetul statului - reluate intr-o perioada mal mare de un an"
"pcg_4482","Other receivables regarding the state budget","4482","asset_current","False","Alte creanțe privind bugetul statului"
"pcg_45111","Settlements between related entities - resumed in a period of up to one year","45111","liability_current","False","Decontări între entitățile afiliate - reluate intr-o perioada de pana la un an"
"pcg_45112","Settlements between related entities - resumed in a period of at least one year","45112","liability_current","False","Decontări între entitățile afiliate - reluate intr-o perioada mal mare de un an"
"pcg_45181","Interest related to settlements between related entities - resumed in a period of up to one year","45181","liability_current","False","Dobânzi aferente decontărilor între entitățile afiliate - reluate intr-o perioada de pana la un an"
"pcg_45182","Interest related to settlements between related entities - resumed in a period of at least one year","45182","liability_current","False","Dobânzi aferente decontărilor între entitățile afiliate - reluate intr-o perioada mal mare de un an"
"pcg_45311","Settlements with associates and jointly controlled entities - resumed in a period of up to one year","45311","liability_current","False","Decontări cu entitățile asociate și entitățile controlate în comun - reluate intr-o perioada de pana la un an"
"pcg_45312","Settlements with associates and jointly controlled entities - resumed in a period of at least one year","45312","liability_current","False","Decontări cu entitățile asociate și entitățile controlate în comun - reluate intr-o perioada mal mare de un an"
"pcg_45381","Interest related to settlements with associated entities and jointly controlled entities - resumed in a period of up to one year","45381","liability_current","False","Dobânzi aferente decontărilor cu entitățile asociate și entitățile controlate în comun - reluate intr-o perioada de pana la un an"
"pcg_45382","Interest related to settlements with associated entities and jointly controlled entities - resumed in a period of at least one year","45382","liability_current","False","Dobânzi aferente decontărilor cu entitățile asociate și entitățile controlate în comun - reluate intr-o perioada mal mare de un an"
"pcg_45511","Associated shareholders - current  - resumed within a period of up to one year","45511","liability_payable","True",""
"pcg_45512","Associated shareholders - current  - resumed within a period of at least one year","45512","liability_payable","True",""
"pcg_45581","Associate shareholders - interest on current  - resumed in a period of up to one year","45581","liability_current","True",""
"pcg_45582","Associate shareholders - interests on current  - resumed in a period of at least one year","45582","liability_current","True",""
"pcg_4561","Settlements with shareholders regarding the capital - resumed in a period of up to one year","4561","liability_current","False","Decontări cu acționarii/asociații privind capitalul - reluate intr-o perioada de pana la un an"
"pcg_4562","Settlements with shareholders regarding the capital - resumed in a period of at least one year","4562","liability_current","False","Decontări cu acționarii/asociații privind capitalul - reluate intr-o perioada mal mare de un an"
"pcg_4571","Dividend payment - resumed in a period of up to one year","4571","liability_current","False","Dividende de plată - reluate intr-o perioada de pana la un an"
"pcg_4572","Dividend payment - resumed in a period of at least one year","4572","liability_current","False","Dividende de plată - reluate intr-o perioada mal mare de un an"
"pcg_45811","Settlements from joint ventures - asset - resumed in a period of up to one year","45811","asset_current","False","Decontări din operațiuni în participație - activ - reluate intr-o perioada de pana la un an"
"pcg_45812","Settlements from joint ventures - asset - resumed in a period of at least one year","45812","asset_current","False","Decontări din operațiuni în participație - activ - reluate intr-o perioada mal mare de un an"
"pcg_4582","Settlements from joint ventures - liability","4582","asset_current","False","Decontări din operațiuni în participație - pasiv"
"pcg_461","Various debtors","461","asset_receivable","True","Debitori diverși"
"pcg_4621","Various creditors - resumed in a period of up to one year","4621","liability_payable","True","Creditori diverși - reluate intr-o perioada de pana la un an"
"pcg_4622","Various creditors - resumed in a period of at least one year","4622","liability_payable","True","Creditori diverși - reluate intr-o perioada mal mare de un an"
"pcg_463","Receivables representing dividends distributed during the financial year","463","liability_payable","True","Creante reprezentand dividende repartizate in cursul exercitiului financiar"
"pcg_46611","Liabilities from trust operations - resumed in a period of up to one year","46611","liability_current","False","Datorii din operațiuni de fiducie - reluate intr-o perioada de pana la un an"
"pcg_46612","Liabilities from trust operations - resumed in a period of at least one year","46612","liability_current","False","Datorii din operațiuni de fiducie - reluate intr-o perioada mal mare de un an"
"pcg_4662","Claims from trust operations","4662","asset_current","False","Creanțe din operațiuni de fiducie"
"pcg_4711","Expenses registered in advance - resumed in a period of up to one year","4711","asset_prepayments","False","Cheltuieli inregistrate in avans - reluate intr-o perioada de pana la un an"
"pcg_4712","Expenses registered in advance - resumed in a period of at least one year","4712","asset_prepayments","False","Cheltuieli inregistrate in avans - reluate intr-o perioada mal mare de un an"
"pcg_4721","Deferred income - resumed in a period of up to one year","4721","liability_non_current","False","Venituri înregistrate în avans - reluate intr-o perioada de pana la un an"
"pcg_4722","Deferred income - resumed in a period of at least one year","4722","liability_non_current","False","Venituri înregistrate în avans - reluate intr-o perioada mal mare de un an"
"pcg_473","Settlements from operations being clarified","473","liability_current","False","Decontări din operațiuni în curs de clarificare"
"pcg_4731","Settlements from operations being clarified - resumed in a period of up to one year","4731","liability_current","False","Decontări din operațiuni în curs de clarificare - reluate intr-o perioada de pana la un an"
"pcg_4732","Settlements from operations being clarified - resumed in a period of at least one year","4732","liability_current","False","Decontări din operațiuni în curs de clarificare - reluate intr-o perioada mal mare de un an"
"pcg_47511","Government subsidies for investment - resumed in a period of up to one year","47511","income_other","False","Subvenții guvernamentale pentru investiții - reluate intr-o perioada de pana la un an"
"pcg_47512","Government subsidies for investment - resumed in a period of at least one year","47512","income_other","False","Subvenții guvernamentale pentru investiții - reluate intr-o perioada mal mare de un an"
"pcg_47521","Non-reimbursable loans in the nature of investment subsidies - resumed in a period of up to one year","47521","income_other","False","Împrumuturi nerambursabile cu caracter de subvenții pentru investiții - reluate intr-o perioada de pana la un an"
"pcg_47522","Non-reimbursable loans in the nature of investment subsidies - resumed in a period of at least one year","47522","income_other","False","Împrumuturi nerambursabile cu caracter de subvenții pentru investiții - reluate intr-o perioada mal mare de un an"
"pcg_47531","Investment donations - resumed in a period of up to one year","47531","income_other","False","Donații pentru investiții - reluate intr-o perioada de pana la un an"
"pcg_47532","Investment donations - resumed in a period of at least one year","47532","income_other","False","Donații pentru investiții - reluate intr-o perioada mal mare de un an"
"pcg_47541","Inventory pluses of the nature of fixed assets - resumed in a period of up to one year","47541","income_other","False","Plusuri de inventar de natura imobilizărilor - reluate intr-o perioada de pana la un an"
"pcg_47542","Inventory pluses of the nature of fixed assets - resumed in a period of at least one year","47542","income_other","False","Plusuri de inventar de natura imobilizărilor - reluate intr-o perioada mal mare de un an"
"pcg_47581","Other amounts received as investment subsidies - resumed in a period of up to one year","47581","income_other","False","Alte sume primite cu caracter de subvenții pentru investiții - reluate intr-o perioada de pana la un an"
"pcg_47582","Other amounts received as investment subsidies - resumed in a period of at least one year","47582","income_other","False","Alte sume primite cu caracter de subvenții pentru investiții - reluate intr-o perioada mal mare de un an"
"pcg_4781","Advance income related to assets received by transfer from customers - resumed in a period of up to one year","4781","income_other","False","Venituri în avans aferente activelor primite prin transfer de la clienți - reluate intr-o perioada de pana la un an"
"pcg_4782","Advance income related to assets received by transfer from customers - resumed in a period of at least one year","4782","income_other","False","Venituri în avans aferente activelor primite prin transfer de la clienț - reluate intr-o perioada mal mare de un an"
"pcg_481","Settlements between unit and subunits","481","liability_current","False","Decontări între unitate și subunități"
"pcg_482","Settlements between subunits","482","liability_current","False","Decontări între subunități"
"pcg_491","Adjustments for impairment of receivables - customers","491","liability_current","False","Ajustări pentru deprecierea creanțelor - clienți"
"pcg_495","Adjustments for impairment of receivables - settlements within the group and with the associated shareholders","495","liability_current","False","Ajustări pentru deprecierea creanțelor - decontări în cadrul grupului și cu acționarii/asociații"
"pcg_4951","Adjustments for impairment of receivables - settlements within the group","4951","liability_current","False","Ajustări pentru deprecierea creanțelor - decontări în cadrul grupului"
"pcg_4952","Adjustments for impairment of receivables - settlements with the associated shareholders","4952","liability_current","False","Ajustări pentru deprecierea creanțelor - decontări cu acționarii/asociații"
"pcg_496","Adjustments for impairment of receivables - other debtors","496","liability_current","False","Ajustări pentru deprecierea creanțelor - debitori diverși"
"pcg_501","Shares held in affiliated entities","501","asset_cash","False","Acțiuni deținute la entitățile afiliate"
"pcg_505","Bonds issued and redeemed","505","asset_cash","False","Obligațiuni emise și răscumpărate"
"pcg_506","Bonds","506","asset_cash","False","Obligațiuni"
"pcg_507","Green certificates received","507","asset_cash","False","Certificate verzi primite"
"pcg_5081","Other securities","5081","asset_cash","False","Alte titluri de plasament"
"pcg_5088","Interest on bonds and securities","5088","asset_cash","False","Dobânzi la obligațiuni și titluri de plasament"
"pcg_50911","Payments to be made for shares held in affiliated entities - resumed in a period of up to one year","50911","asset_cash","False","Vărsăminte de efectuat pentru acțiunile deținute la entitățile afiliate - reluate intr-o perioada de pana la un an"
"pcg_50912","Payments to be made for shares held in affiliated entities - resumed in a period of at least one year","50912","asset_cash","False","Vărsăminte de efectuat pentru acțiunile deținute la entitățile afiliate - reluate intr-o perioada mal mare de un an"
"pcg_50921","Payments to be made for other short-term investments - resumed in a period of up to one year","50921","asset_cash","False","Vărsăminte de efectuat pentru alte investiții pe termen scurt - reluate intr-o perioada de pana la un an"
"pcg_50922","Payments to be made for other short-term investments - resumed in a period of at least one year","50922","asset_cash","False","Vărsăminte de efectuat pentru alte investiții pe termen scurt - reluate intr-o perioada mal mare de un an"
"pcg_5112","Checks to be cashed","5112","asset_cash","False","Cecuri de încasat"
"pcg_5113","Notes to be collected","5113","asset_cash","False","Efecte de încasat"
"pcg_5114","Discounted notes","5114","asset_cash","False","Efecte remise spre scontare"
"pcg_5121","Bank  in lei","5121","asset_cash","False",""
"pcg_5124","Bank  in foreign currency","5124","asset_cash","False",""
"pcg_5125","Amounts being settled","5125","asset_current","False","Sume în curs de decontare"
"pcg_51861","Payable interest - resumed in a period of up to one year","51861","asset_cash","False","Dobânzi de plătit - reluate intr-o perioada de pana la un an"
"pcg_51862","Payable interest - resumed in a period of at least one year","51862","asset_cash","False","Dobânzi de plătit - reluate intr-o perioada mal mare de un an"
"pcg_5187","Interest receivable","5187","asset_cash","False","Dobânzi de încasat"
"pcg_51911","Short-term bank loans - resumed in a period of up to one year","51911","asset_cash","False","Credite bancare pe termen scurt - reluate intr-o perioada de pana la un an"
"pcg_51912","Short-term bank loans - resumed in a period of at least one year","51912","asset_cash","False","Credite bancare pe termen scurt - reluate intr-o perioada mal mare de un an"
"pcg_51921","Short-term bank loans not repaid at maturity - resumed in a period of up to one year","51921","asset_cash","False","Credite bancare pe termen scurt nerambursate la scadență - reluate intr-o perioada de pana la un an"
"pcg_51922","Short-term bank loans not repaid at maturity - resumed in a period of at least one year","51922","asset_cash","False","Credite bancare pe termen scurt nerambursate la scadență - reluate intr-o perioada mal mare de un an"
"pcg_51931","Foreign government loans - resumed in a period of up to one year","51931","asset_cash","False","Credite externe guvernamentale - reluate intr-o perioada de pana la un an"
"pcg_51932","Foreign government loans - resumed in a period of at least one year","51932","asset_cash","False","Credite externe guvernamentale - reluate intr-o perioada mal mare de un an"
"pcg_51941","Foreign loans guaranteed by the state - resumed in a period of up to one year","51941","asset_cash","False","Credite externe garantate de stat - reluate intr-o perioada de pana la un an"
"pcg_51942","Foreign loans guaranteed by the state - resumed in a period of at least one year","51942","asset_cash","False","Credite externe garantate de stat - reluate intr-o perioada mal mare de un an"
"pcg_51951","External loans guaranteed by banks - resumed in a period of up to one year","51951","asset_cash","False","Credite externe garantate de bănci - reluate intr-o perioada de pana la un an"
"pcg_51952","External loans guaranteed by banks - resumed in a period of at least one year","51952","asset_cash","False","Credite externe garantate de bănci - reluate intr-o perioada mal mare de un an"
"pcg_51961","Loans from the state treasury - resumed in a period of up to one year","51961","asset_cash","False","Credite de la trezoreria statului - reluate intr-o perioada de pana la un an"
"pcg_51962","Loans from the state treasury - resumed in a period of at least one year","51962","asset_cash","False","Credite de la trezoreria statului - reluate intr-o perioada mal mare de un an"
"pcg_51971","Domestic loans guaranteed by the state - resumed in a period of up to one year","51971","asset_cash","False","Credite interne garantate de stat - reluate intr-o perioada de pana la un an"
"pcg_51972","Domestic loans guaranteed by the state - resumed in a period of at least one year","51972","asset_cash","False","Credite interne garantate de stat - reluate intr-o perioada mal mare de un an"
"pcg_51981","Interest related to short-term bank loans - resumed in a period of up to one year","51981","asset_cash","False","Dobânzi aferente creditelor bancare pe termen scurt - reluate intr-o perioada de pana la un an"
"pcg_51982","Interest related to short-term bank loans - resumed in a period of at least one year","51982","asset_cash","False","Dobânzi aferente creditelor bancare pe termen scurt - reluate intr-o perioada mal mare de un an"
"pcg_5311","Cash in lei","5311","asset_cash","False","Casa în lei"
"pcg_5314","Cash in foreign currency","5314","asset_cash","False","Casa în valută"
"pcg_5321","Tax and postage stamps","5321","asset_cash","False","Timbre fiscale și poștale"
"pcg_5322","Treatment and rest tickets","5322","asset_cash","False","Bilete de tratament și odihnă"
"pcg_5323","Tickets and travel tickets","5323","asset_cash","False","Tichete și bilete de călătorie"
"pcg_5328","Other valuables","5328","asset_cash","False","Alte valori"
"pcg_5411","Letters of credit in lei","5411","asset_cash","False","Acreditive în lei"
"pcg_5414","Letters of credit in foreign currency","5414","asset_cash","False","Acreditive în valută"
"pcg_542","Cash advances","542","asset_cash","False","Avansuri de trezorerie"
"pcg_581","Internal transfers","581","asset_cash","False","Viramente interne"
"pcg_591","Adjustments for impairment of shares held in affiliated entities","591","asset_cash","False","Ajustări pentru pierderea de valoare a acțiunilor deținute la entitățile afiliate"
"pcg_595","Adjustments for impairment of bonds issued and redeemed","595","asset_cash","False","Ajustări pentru pierderea de valoare a obligațiunilor emise și răscumpărate"
"pcg_596","Adjustments for bond impairment","596","asset_cash","False","Ajustări pentru pierderea de valoare a obligațiunilor"
"pcg_598","Adjustments for impairment of other short-term investments and assimilated receivables","598","asset_cash","False","Ajustări pentru pierderea de valoare a altor investiții pe termen scurt și creanțe asimilate"
"pcg_601","Raw material expenses","601","expense","False","Cheltuieli cu materiile prime"
"pcg_6021","Expenses with auxiliary materials","6021","expense","False","Cheltuieli cu materiale auxiliare"
"pcg_6022","Fuel expenses","6022","expense","False","Cheltuieli privind combustibilul"
"pcg_6023","Expenses regarding packaging materials","6023","expense","False","Cheltuieli privind materialele pentru ambalat"
"pcg_6024","Spare parts costs","6024","expense","False","Cheltuieli privind piesele de schimb"
"pcg_6025","Costs of seeds and planting materials","6025","expense","False","Cheltuieli privind semințele și materialele de plantat"
"pcg_6026","Feed expenses","6026","expense","False","Cheltuieli privind furajele"
"pcg_6028","Expenses regarding other consumables","6028","expense","False","Cheltuieli privind alte materiale consumabile"
"pcg_603","Expenses regarding materials of the nature of inventory items","603","expense","False","Cheltuieli privind materialele de natura obiectelor de inventar"
"pcg_604","Expenditure on non-stocked materials","604","expense","False","Cheltuieli privind materialele nestocate"
"pcg_605","Energy and water expenses","605","expense","False","Cheltuieli privind energia și apa"
"pcg_606","Expenditures on biological assets","606","expense","False","Cheltuieli privind activele biologice de natura stocurilor"
"ro_pcg_expense","Expenditure on goods","607","expense","False","Cheltuieli privind mărfurile"
"pcg_608","Packaging charges","608","expense","False","Cheltuieli privind ambalajele"
"pcg_6091","Trade discounts received - raw materials and consumables","6091","expense","False","Reduceri comerciale primite - materiilor prime și al consumabilelor"
"pcg_6092","Trade discounts received - other expenses","6092","expense","False","Reduceri comerciale primite - alte cheltuieli"
"pcg_611","Maintenance and repair costs","611","expense","False","Cheltuieli cu întreținerea și reparațiile"
"pcg_612","Expenses for royalties, management premises and rents","612","expense","False","Cheltuieli cu redevențele, locațiile de gestiune și chiriile"
"pcg_613","Insurance premium expenses","613","expense","False","Cheltuieli cu primele de asigurare"
"pcg_614","Study and research expenses","614","expense","False","Cheltuieli cu studiile și cercetările"
"pcg_615","Staff training expenses","615","expense","False","Cheltuieli cu pregătirea personalului"
"pcg_621","Expenses regarding collaborators","621","expense","False","Cheltuieli cu colaboratorii"
"pcg_622","Expenses relating to commissions and fees","622","expense","False","Cheltuieli privind comisioanele și onorariile"
"pcg_6231","Protocol expenses","6231","expense","False","Cheltuieli de protocol"
"pcg_6232","Advertising and publicity expenses","6232","expense","False","Cheltuieli de reclamă și publicitate"
"pcg_624","Expenses for transportation of goods and personnel","624","expense","False","Cheltuieli cu transportul de bunuri și personal"
"pcg_625","Travel, secondment and transfer expenses","625","expense","False","Cheltuieli cu deplasări, detașări și transferări"
"pcg_626","Postage and telecommunications charges","626","expense","False","Cheltuieli poștale și taxe de telecomunicații"
"pcg_627","Expenses with banking and similar services","627","expense","False","Cheltuieli cu serviciile bancare și asimilate"
"pcg_628","Other expenses with services performed by third parties","628","expense","False","Alte cheltuieli cu serviciile executate de terți"
"pcg_6351","Expenses with other taxes, fees and similar payments","6351","expense","False","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate"
"pcg_6352","Expenses with other non-deductible taxes, duties and similar payments","6352","expense","False","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate nedeductibile"
"pcg_641","Staff salary expenses","641","expense","False","Cheltuieli cu salariile personalului"
"pcg_6421","Expenses with benefits in kind granted to employees","6421","expense","False","Cheltuieli cu avantajele în natură acordate salariaților"
"pcg_6422","Expenditure on vouchers granted to employees","6422","expense","False","Cheltuieli cu tichetele acordate salariaților"
"pcg_643","Remuneration expenses in equity instruments","643","expense","False","Cheltuieli cu remunerarea în instrumente de capitaluri proprii"
"pcg_644","Premium expenses representing staff profit sharing","644","expense","False","Cheltuieli cu primele reprezentând participarea personalului la profit"
"pcg_6451","Expenses regarding the unit's contribution to social security","6451","expense","False","Cheltuieli privind contribuția unității la asigurările sociale"
"pcg_6452","Expenses regarding the unit's contribution to unemployment benefit","6452","expense","False","Cheltuieli privind contribuția unității pentru ajutorul de șomaj"
"pcg_6453","Expenses regarding the employer's contribution for social health insurance","6453","expense","False","Cheltuieli privind contribuția angajatorului pentru asigurările sociale de sănătate"
"pcg_6455","Expenses relating to the unit's contribution to life insurance","6455","expense","False","Cheltuieli privind contribuția unității la asigurările de viață"
"pcg_6456","Expenses regarding the unit's contribution to voluntary pension funds","6456","expense","False","Cheltuieli privind contribuția unității la fondurile de pensii facultative"
"pcg_6457","Expenses regarding the unit's contribution to voluntary health insurance premiums","6457","expense","False","Cheltuieli privind contribuția unității la primele de asigurare voluntară de sănătate"
"pcg_6458","Other insurance and social protection expenses","6458","expense","False","Alte cheltuieli privind asigurările și protecția socială"
"pcg_646","Labor insurance contribution expenses","646","expense","False","Cheltuieli privind contribuția asiguratorie pentru muncă"
"pcg_6511","Expenses occasioned by the establishment of the trust","6511","expense","False","Cheltuieli ocazionate de constituirea fiduciei"
"pcg_6512","Expenses from running trust operations","6512","expense","False","Cheltuieli din derularea operațiunilor de fiducie"
"pcg_6513","Expenses from the liquidation of trust operations","6513","expense","False","Cheltuieli din lichidarea operațiunilor de fiducie"
"pcg_652","Environmental protection expenses","652","expense","False","Cheltuieli cu protecția mediului înconjurător"
"pcg_654","Losses from sundry receivables and debtors","654","expense","False","Pierderi din creanțe și debitori diverși"
"pcg_655","Expenses from revaluation of tangible assets","655","expense","False","Cheltuieli din reevaluarea imobilizărilor corporale"
"pcg_6581","Compensation, fines and penalties","6581","expense","False","Despăgubiri, amenzi și penalități"
"pcg_6582","Donations granted","6582","expense","False","Donații acordate"
"pcg_6583","Expenditure on divested assets and other capital operations","6583","expense","False","Cheltuieli privind activele cedate și alte operațiuni de capital"
"pcg_6584","Expenditure on amounts or goods awarded as sponsorships","6584","expense","False","Cheltuieli cu sumele sau bunurile acordate ca sponsorizări"
"pcg_6586","Expenses representing transfers and contributions due on the basis of special normative acts","6586","expense","False","Cheltuieli reprezentând transferuri și contribuții datorate în baza unor acte normative speciale"
"pcg_6587","Expenses related to calamities and other similar events","6587","expense","False","Cheltuieli privind calamitățile și alte evenimente similare"
"pcg_65881","Other operating expenses","65881","expense","False","Alte cheltuieli de exploatare"
"pcg_65882","Other non-deductible operating expenses","65882","expense","False","Alte cheltuieli de exploatare nedeductibile"
"pcg_663","Losses on receivables related to holdings","663","expense","False","Pierderi din creanțe legate de participații"
"pcg_6641","Expenses related to the financial fixed assets sold","6641","expense","False","Cheltuieli privind imobilizările financiare cedate"
"pcg_6642","Losses on short-term investments divested","6642","expense","False","Pierderi din investițiile pe termen scurt cedate"
"pcg_6651","Unfavorable exchange rate differences related to monetary items denominated in foreign currency","6651","expense","False","Diferențe nefavorabile de curs valutar legate de elementele monetare exprimate în valută"
"pcg_6652","Unfavorable exchange rate differences from the measurement of monetary items that are part of the net investment in a foreign entity","6652","expense","False","Diferențe nefavorabile de curs valutar legate de elementele monetare exprimate în valută"
"pcg_666","Interest charges","666","expense","False","Cheltuieli privind dobânzile"
"pcg_667","Expenses related to the discounts granted","667","expense","False","Cheltuieli privind sconturile acordate"
"pcg_668","Other financial expenses","668","expense","False","Alte cheltuieli financiare"
"pcg_6811","Operating expenses regarding depreciation of fixed assets","6811","expense","False","Cheltuieli de exploatare privind amortizarea imobilizărilor"
"pcg_6812","Operating expenses on provisions","6812","expense","False","Cheltuieli de exploatare privind provizioanele"
"pcg_6813","Operating expenses related to adjustments for depreciation of fixed assets","6813","expense","False","Cheltuieli de exploatare privind ajustările pentru deprecierea imobilizărilor"
"pcg_6814","Operating expenses regarding adjustments for impairment of current assets","6814","expense","False","Cheltuieli de exploatare privind ajustările pentru deprecierea activelor circulante"
"pcg_68151","Expenditures on production sold","68151","expense","False","Cheltuieli cu producția vândută"
"pcg_68152","Expenditure on the sale of goods","68152","expense","False","Cheltuieli din vânzarea mărfurilor"
"pcg_6817","Operating expenses related to goodwill impairment adjustments","6817","expense","False","Cheltuieli de exploatare privind ajustările pentru deprecierea fondului comercial"
"pcg_6861","Expenditure on update of provisions","6861","expense","False","Cheltuieli privind actualizarea provizioanelor"
"pcg_6863","Financial expenses regarding adjustments for the loss of value of financial assets","6863","expense","False","Cheltuieli financiare privind ajustările pentru pierderea de valoare a imobilizărilor financiare"
"pcg_6864","Financial expenses regarding adjustments for the loss of value of current assets","6864","expense","False","Cheltuieli financiare privind ajustările pentru pierderea de valoare a activelor circulante"
"pcg_6865","Financial expenses regarding the amortization of differences related to government securities","6865","expense","False","Cheltuieli financiare privind amortizarea diferențelor aferente titlurilor de stat"
"pcg_6868","Finance expenses related to the amortization of bond repayment premiums and other liabilities","6868","expense","False","Cheltuieli financiare privind amortizarea primelor de rambursare a obligațiunilor și a altor datorii"
"pcg_691","Income tax expenses","691","expense","False","Cheltuieli cu impozitul pe profit"
"pcg_695","Expenses with the tax specific to some activities","695","expense","False","Cheltuieli cu impozitul specific unor activitati"
"pcg_698","Expenses with income tax and other taxes that do not appear in the items above","698","expense","False","Cheltuieli cu impozitul pe venit și cu alte impozite care nu apar in elementele de mai sus"
"pcg_7015","Revenue from the sale of finished products","7015","income","False","Venituri din vânzarea produselor finite"
"pcg_7017","Income from the sale of agricultural products","7017","income","False","Venituri din vânzarea produselor agricole"
"pcg_7018","Income from the sale of biological assets in the nature of stocks","7018","income","False","Venituri din vânzarea activelor biologice de natura stocurilor"
"pcg_702","Income from the sale of semi-finished products","702","income","False","Venituri din vânzarea semifabricatelor"
"pcg_703","Income from the sale of residual products","703","income","False","Venituri din vânzarea produselor reziduale"
"pcg_704","Revenue from services provided","704","income","False","Venituri din servicii prestate"
"pcg_705","Income from studies and research","705","income","False","Venituri din studii și cercetări"
"pcg_706","Income from royalties, management premises and rents","706","income","False","Venituri din redevențe, locații de gestiune și chirii"
"ro_pcg_sale","Income from sale of goods","707","income","False","Venituri din vânzarea mărfurilor"
"pcg_708","Income from miscellaneous activities","708","income","False","Venituri din activități diverse"
"pcg_709","Commercial discounts granted","709","income","False","Reduceri comerciale acordate"
"pcg_711","Revenues related to product inventory costs","711","income","False","Venituri aferente costurilor stocurilor de produse"
"pcg_712","Revenues related to the costs of services in progress","712","income","False","Venituri aferente costurilor serviciilor în curs de execuție"
"pcg_721","Income from the production of intangible assets","721","income","False","Venituri din producția de imobilizări necorporale"
"pcg_722","Income from the production of tangible assets","722","income","False","Venituri din producția de imobilizări corporale"
"pcg_725","Income from the production of real estate investments","725","income","False","Venituri din producția de investiții imobiliare"
"pcg_7411","Income from operating subsidies related to turnover","7411","income","False","Venituri din subvenții de exploatare aferente cifrei de afaceri"
"pcg_7412","Income from operating subsidies for raw materials and materials","7412","income","False","Venituri din subvenții de exploatare pentru materii prime și materiale"
"pcg_7413","Income from operating subsidies for other external expenditure","7413","income","False","Venituri din subvenții de exploatare pentru alte cheltuieli externe"
"pcg_7414","Income from operating subsidies to pay staff","7414","income","False","Venituri din subvenții de exploatare pentru plata personalului"
"pcg_7415","Income from operating subsidies for insurance and social protection","7415","income","False","Venituri din subvenții de exploatare pentru asigurări și protecție socială"
"pcg_7416","Income from operating subsidies for other operating expenses","7416","income","False","Venituri din subvenții de exploatare pentru alte cheltuieli de exploatare"
"pcg_7417","Income from operating subsidies in case of calamities and other similar events","7417","income","False","Venituri din subvenții de exploatare în caz de calamități și alte evenimente similare"
"pcg_7418","Income from operating subsidies for interest due","7418","income","False","Venituri din subvenții de exploatare pentru dobânda datorată"
"pcg_7419","Income from operating subsidies related to other income","7419","income","False","Venituri din subvenții de exploatare aferente altor venituri"
"pcg_7511","Income occasioned by the establishment of the trust","7511","income","False","Venituri ocazionate de constituirea fiduciei"
"pcg_7512","Income from trust operations","7512","income","False","Venituri din derularea operațiunilor de fiducie"
"pcg_7513","Proceeds from liquidation of trust operations","7513","income","False","Venituri din lichidarea operațiunilor de fiducie"
"pcg_754","Income from reactivated receivables and sundry debtors","754","income","False","Venituri din creanțe reactivate și debitori diverși"
"pcg_755","Income from revaluation of tangible assets","755","income","False","Venituri din reevaluarea imobilizărilor corporale"
"pcg_7581","Income from compensation, fines and penalties","7581","income","False","Venituri din despăgubiri, amenzi și penalități"
"pcg_7582","Income from donations received","7582","income","False","Venituri din donații primite"
"pcg_7583","Proceeds from the sale of assets and other capital operations","7583","income","False","Venituri din vânzarea activelor și alte operațiuni de capital"
"pcg_7584","Income from investment subsidies","7584","income","False","Venituri din subvenții pentru investiții"
"pcg_7586","Income representing due transfers based on special normative acts","7586","income","False","Venituri reprezentând transferuri cuvenite în baza unor acte normative speciale"
"pcg_7588","Other operating revenues","7588","income","False","Alte venituri din exploatare"
"pcg_7611","Income from shares held in affiliated entities","7611","income","False","Venituri din acțiuni deținute la entitățile afiliate"
"pcg_7612","Income from shares held in associated entities","7612","income","False","Venituri din acțiuni deținute la entități asociate"
"pcg_7613","Income from shares held in jointly controlled entities","7613","income","False","Venituri din acțiuni deținute la entități controlate în comun"
"pcg_7615","Income from other financial assets","7615","income","False","Venituri din alte imobilizări financiare"
"pcg_762","Income from short-term financial investments","762","income","False","Venituri din investiții financiare pe termen scurt"
"pcg_7641","Income from financial fixed assets sold","7641","income","False","Venituri din imobilizări financiare cedate"
"pcg_7642","Gains on short-term investments ceded","7642","income","False","Câștiguri din investiții pe termen scurt cedate"
"pcg_7651","Favorable exchange rate differences related to monetary items denominated in foreign currency","7651","income","False","Diferențe favorabile de curs valutar legate de elementele monetare exprimate în valută"
"pcg_7652","Favorable exchange rate differences from the valuation of monetary items that are part of the net investment in a foreign entity","7652","income","False","Diferențe favorabile de curs valutar din evaluarea elementelor monetare care fac parte din investiția netă într-o entitate străină"
"pcg_766","Interest income","766","income","False","Venituri din dobânzi"
"pcg_767","Revenue from discounts obtained","767","income","False","Venituri din sconturi obținute"
"pcg_768","Other incomes","768","income","False","Alte venituri financiare"
"pcg_7812","Income from provisions","7812","income","False","Venituri din provizioane"
"pcg_7813","Income from adjustments for depreciation of fixed assets","7813","income","False","Venituri din ajustări pentru deprecierea imobilizărilor"
"pcg_7814","Income from adjustments for impairment of current assets","7814","income","False","Venituri din ajustări pentru deprecierea activelor circulante"
"pcg_7815","Negative goodwill income","7815","income","False","Venituri din fondul comercial negativ"
"pcg_7863","Financial income from adjustments for the loss of value of financial assets","7863","income","False","Venituri financiare din ajustări pentru pierderea de valoare a imobilizărilor financiare"
"pcg_7864","Financial income from adjustments for the loss of value of current assets","7864","income","False","Venituri financiare din ajustări pentru pierderea de valoare a activelor circulante"
"pcg_7865","Financial income from the amortization of differences related to government securities","7865","income","False","Venituri financiare din amortizarea diferențelor aferente titlurilor de stat"
"pcg_8011","Warranties and guarantees granted","8011","off_balance","False","Giruri și garanții acordate"
"pcg_8018","Other commitments granted","8018","off_balance","False","Alte angajamente acordate"
"pcg_8021","Warranties and guarantees received","8021","off_balance","False","Giruri și garanții primite"
"pcg_8028","Other commitments received","8028","off_balance","False","Alte angajamente primite"
"pcg_8031","Leased tangible assets","8031","off_balance","False","Imobilizări corporale luate cu chirie"
"pcg_8032","Material values received for processing or repair","8032","off_balance","False","Valori materiale primite spre prelucrare sau reparare"
"pcg_8033","Material values received in safekeeping or custody","8033","off_balance","False","Valori materiale primite în păstrare sau custodie"
"pcg_8034","Discharged debtors, follow up","8034","off_balance","False","Debitori scoși din activ, urmăriți în continuare"
"pcg_8035","Stocks in the nature of inventory items put into use","8035","off_balance","False","Stocuri de natura obiectelor de inventar date în folosință"
"pcg_8036","Royalties, management premises, rents and other similar liabilities","8036","off_balance","False","Redevențe, locații de gestiune, chirii și alte datorii asimilate"
"pcg_8037","Expected notes not due","8037","off_balance","False","Efecte scontate neajunse la scadență"
"pcg_8038","Assets received in administration, concession and rent","8038","off_balance","False","Bunuri primite în administrare, concesiune și cu chirie"
"pcg_8039","Other off-balance sheet values","8039","off_balance","False","Alte valori în afara bilanțului"
"pcg_804","Green certificates","804","off_balance","False","Certificate verzi"
"pcg_8051","Payable interest","8051","off_balance","False","Dobânzi de plătit"
"pcg_8052","Interest receivable","8052","off_balance","False","Dobânzi de încasat"
"pcg_806","Greenhouse gas emission certificates","806","off_balance","False","Certificate de emisii de gaze cu efect de seră"
"pcg_807","Active contingent","807","off_balance","False","Contingent activ"
"pcg_808","Contingent liabilities","808","off_balance","False","Datorii contingente"
"pcg_809","Receivables taken over by assignment","809","off_balance","False","Creanțe preluate prin cesionare"
"pcg_891","Opening balance sheet","891","off_balance","False","Bilanț de deschidere"
"pcg_892","Closing balance sheet","892","off_balance","False","Bilanț de închidere"
"pcg_901","Internal expense settlements","901","off_balance","False","Decontări interne privind cheltuielile"
"pcg_902","Internal settlements on production obtained","902","off_balance","False","Decontări interne privind producția obținută"
"pcg_903","Internal settlements of price differences","903","off_balance","False","Decontări interne privind diferențele de preț"
"pcg_921","Core business expenses","921","off_balance","False","Cheltuielile activității de bază"
"pcg_922","Expenses of auxiliary activities","922","off_balance","False","Cheltuielile activităților auxiliare"
"pcg_923","Indirect production costs","923","off_balance","False","Cheltuieli indirecte de producție"
"pcg_924","General administrative expenses","924","off_balance","False","Cheltuieli generale de administrație"
"pcg_925","Selling expenses","925","off_balance","False","Cheltuieli de desfacere"
"pcg_931","Cost of production obtained","931","off_balance","False","Costul producției obținute"
"pcg_933","Cost of production in progress","933","off_balance","False","Costul producției în curs de execuție"

```

## File: data\template\account.fiscal.position-ro.csv

```csv
"id","name","country_id","auto_apply","vat_required","sequence","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@ro"
"fiscal_position_template_1","National Regime (VAT)","base.ro","1","1","10","","","","Regimul național (TVA)"
"fiscal_position_template_11","National Regime","base.ro","1","","11","","","","Regimul National"
"fiscal_position_template_2","Reverse Tax Regime","base.ro","","1","12","","tvac_00","tvati","Regime de taxare inversă"
"","","","","","","","tvac_05","tvati",""
"","","","","","","","tvac_09","tvati",""
"","","","","","","","tvac_19","tvati",""
"","","","","","","","tvac_00_s","tvati",""
"","","","","","","","tvac_05_s","tvati",""
"","","","","","","","tvac_09_s","tvati",""
"","","","","","","","tvac_19_s","tvati",""
"","","","","","","","tvad_00","tvatip00",""
"","","","","","","","tvad_05","tvatip05",""
"","","","","","","","tvad_09","tvatip09",""
"","","","","","","","tvad_19","tvatip19",""
"","","","","","","","tvad_00_s","tvatip00",""
"","","","","","","","tvad_05_s","tvatip05",""
"","","","","","","","tvad_09_s","tvatip09",""
"","","","","","","","tvad_19_s","tvatip19",""
"fiscal_position_template_8","VAT collection system","base.ro","","1","13","","tvac_00","tvaic_00","Sistem de colectare TVA"
"","","","","","","","tvac_05","tvaic_05",""
"","","","","","","","tvac_09","tvaic_09",""
"","","","","","","","tvac_19","tvaic_19",""
"","","","","","","","tvac_00_s","tvaic_00",""
"","","","","","","","tvac_05_s","tvaic_05",""
"","","","","","","","tvac_09_s","tvaic_09",""
"","","","","","","","tvac_19_s","tvaic_19",""
"","","","","","","","tvad_00","tvaid_00",""
"","","","","","","","tvad_05","tvaid_05",""
"","","","","","","","tvad_09","tvaid_09",""
"","","","","","","","tvad_19","tvaid_19",""
"","","","","","","","tvad_00_s","tvaid_00",""
"","","","","","","","tvad_05_s","tvaid_05",""
"","","","","","","","tvad_09_s","tvaid_09",""
"","","","","","","","tvad_19_s","tvaid_19",""
"fiscal_position_template_3","Intra-Community Scheme (VAT)","","1","1","14","base.europe","tvac_00","tvati_intrab","Schema intracomunitară (TVA)"
"","","","","","","","tvac_05","tvati_intrab",""
"","","","","","","","tvac_09","tvati_intrab",""
"","","","","","","","tvac_19","tvati_intrab",""
"","","","","","","","tvac_00_s","tvati_intras",""
"","","","","","","","tvac_05_s","tvati_intras",""
"","","","","","","","tvac_09_s","tvati_intras",""
"","","","","","","","tvac_19_s","tvati_intras",""
"","","","","","","","tvad_00","tvati_intrap0b",""
"","","","","","","","tvad_05","tvati_intrap5b",""
"","","","","","","","tvad_09","tvati_intrap9b",""
"","","","","","","","tvad_19","tvati_intrap19b",""
"","","","","","","","tvad_00_s","tvati_intrap0s",""
"","","","","","","","tvad_05_s","tvati_intrap5s",""
"","","","","","","","tvad_09_s","tvati_intrap9s",""
"","","","","","","","tvad_19_s","tvati_intrap19s",""
"fiscal_position_template_4","Exempted Intra-Community Regime","","1","","15","base.europe","tvac_00","tvati_intrab","Regim intracomunitar scutit"
"","","","","","","","tvac_05","tvati_intrab",""
"","","","","","","","tvac_09","tvati_intrab",""
"","","","","","","","tvac_19","tvati_intrab",""
"","","","","","","","tvac_00_s","tvati_intras",""
"","","","","","","","tvac_05_s","tvati_intras",""
"","","","","","","","tvac_09_s","tvati_intras",""
"","","","","","","","tvac_19_s","tvati_intras",""
"","","","","","","","tvad_00","tvatiscdeda_intr",""
"","","","","","","","tvad_05","tvatiscdeda_intr",""
"","","","","","","","tvad_09","tvatiscdeda_intr",""
"","","","","","","","tvad_19","tvatiscdeda_intr",""
"","","","","","","","tvad_00_s","tvatiscdeda_intr",""
"","","","","","","","tvad_05_s","tvatiscdeda_intr",""
"","","","","","","","tvad_09_s","tvatiscdeda_intr",""
"","","","","","","","tvad_19_s","tvatiscdeda_intr",""
"fiscal_position_template_51","Exempt regime - with the right of deduction","","","","16","","tvac_00","tvatiscded","Regim de scutire - cu drept de deducere"
"","","","","","","","tvac_05","tvatiscded",""
"","","","","","","","tvac_09","tvatiscded",""
"","","","","","","","tvac_19","tvatiscded",""
"","","","","","","","tvac_00_s","tvatiscded",""
"","","","","","","","tvac_05_s","tvatiscded",""
"","","","","","","","tvac_09_s","tvatiscded",""
"","","","","","","","tvac_19_s","tvatiscded",""
"","","","","","","","tvad_00","tvatiscdeda",""
"","","","","","","","tvad_05","tvatiscdeda",""
"","","","","","","","tvad_09","tvatiscdeda",""
"","","","","","","","tvad_19","tvatiscdeda",""
"","","","","","","","tvad_00_s","tvatiscdeda",""
"","","","","","","","tvad_05_s","tvatiscdeda",""
"","","","","","","","tvad_09_s","tvatiscdeda",""
"","","","","","","","tvad_19_s","tvatiscdeda",""
"fiscal_position_template_52","Exempt regime - no right of deduction","","","","17","","tvac_00","tvatiscnoded","Regim de scutire - fără drept de deducere"
"","","","","","","","tvac_05","tvatiscnoded",""
"","","","","","","","tvac_09","tvatiscnoded",""
"","","","","","","","tvac_19","tvatiscnoded",""
"","","","","","","","tvac_00_s","tvatiscnoded",""
"","","","","","","","tvac_05_s","tvatiscnoded",""
"","","","","","","","tvac_09_s","tvatiscnoded",""
"","","","","","","","tvac_19_s","tvatiscnoded",""
"","","","","","","","tvad_00","tvatiscdeda",""
"","","","","","","","tvad_05","tvatiscdeda",""
"","","","","","","","tvad_09","tvatiscdeda",""
"","","","","","","","tvad_19","tvatiscdeda",""
"","","","","","","","tvad_00_s","tvatiscdeda",""
"","","","","","","","tvad_05_s","tvatiscdeda",""
"","","","","","","","tvad_09_s","tvatiscdeda",""
"","","","","","","","tvad_19_s","tvatiscdeda",""
"fiscal_position_template_6","Intra-Community Regime Non-taxable","","","","18","base.europe","tvac_00","tvatine","Regim intracomunitar neimpozabil"
"","","","","","","","tvac_05","tvatine",""
"","","","","","","","tvac_09","tvatine",""
"","","","","","","","tvac_19","tvatine",""
"","","","","","","","tvac_00_s","tvatine",""
"","","","","","","","tvac_05_s","tvatine",""
"","","","","","","","tvac_09_s","tvatine",""
"","","","","","","","tvac_19_s","tvatine",""
"","","","","","","","tvad_00","tvatinea",""
"","","","","","","","tvad_05","tvatinea",""
"","","","","","","","tvad_09","tvatinea",""
"","","","","","","","tvad_19","tvatinea",""
"","","","","","","","tvad_00_s","tvatinea",""
"","","","","","","","tvad_05_s","tvatinea",""
"","","","","","","","tvad_09_s","tvatinea",""
"","","","","","","","tvad_19_s","tvatinea",""
"fiscal_position_template_7","Extra-Community Regime","","1","","19","","tvac_00","tvati_extra","Regimul extracomunitar"
"","","","","","","","tvac_05","tvati_extra",""
"","","","","","","","tvac_09","tvati_extra",""
"","","","","","","","tvac_19","tvati_extra",""
"","","","","","","","tvac_00_s","tvati_extras",""
"","","","","","","","tvac_05_s","tvati_extras",""
"","","","","","","","tvac_09_s","tvati_extras",""
"","","","","","","","tvac_19_s","tvati_extras",""
"","","","","","","","tvad_00","tvati_extrap0b",""
"","","","","","","","tvad_05","tvati_extrap0b",""
"","","","","","","","tvad_09","tvati_extrap0b",""
"","","","","","","","tvad_19","tvati_extrap0b",""
"","","","","","","","tvad_00_s","tvati_extrap0s",""
"","","","","","","","tvad_05_s","tvati_extrap0s",""
"","","","","","","","tvad_09_s","tvati_extrap0s",""
"","","","","","","","tvad_19_s","tvati_extrap0s",""

```

## File: data\template\account.group-ro.csv

```csv
"id","name","code_prefix_start","code_prefix_end","name@ro"
"account_group_1","CAPITAL ACCOUNTS, PROVISIONS, LOANS AND SIMILAR LIABILITIES","1","","CONTURI DE CAPITALURI, PROVIZIOANE, ÎMPRUMUTURI ȘI DATORII ASIMILATE"
"account_group_10","Capital and reserves","10","","Capital și rezerve"
"account_group_101","Capital","101","","Capital"
"account_group_103","Other elements of equity","103","","Alte elemente de capitaluri proprii"
"account_group_104","capital premiums","104","","Prime de capital"
"account_group_106","Reserves","106","","Rezerve"
"account_group_108","Non-controlling interests","108","","Interese care nu controlează"
"account_group_109","Own shares","109","","Acțiuni proprii"
"account_group_11","Reported result","11","","Rezultatul reportat"
"account_group_117","Own shares","117","","Acțiuni proprii"
"account_group_12","Financial year result","12","","Rezultatul exercițiului financiar"
"account_group_14","Gains or losses on the issue, redemption, sale, free transfer or cancellation of equity instruments","14","","Câștiguri sau pierderi legate de emiterea, răscumpărarea, vânzarea, cedarea cu titlu gratuit sau anularea instrumentelor de capitaluri proprii"
"account_group_141","Gains on the sale or cancellation of equity instruments","141","","Câștiguri legate de vânzarea sau anularea instrumentelor de capitaluri proprii"
"account_group_149","Losses on the issue, redemption, sale, free transfer or cancellation of equity instruments","149","","Pierderi legate de emiterea, răscumpărarea, vânzarea, cedarea cu titlu gratuit sau anularea instrumentelor de capitaluri proprii"
"account_group_15","Provisions","15","","Provizioane"
"account_group_151","Provisions","151","","Provizioane"
"account_group_16","Loans and similar liabilities","16","","Împrumuturi și datorii asimilate"
"account_group_161","Loans from bond issues","161","","Împrumuturi din emisiuni de obligațiuni"
"account_group_162","Long-term bank loans","162","","Credite bancare pe termen lung"
"account_group_166","Liabilities related to financial assets","166","","Datorii care privesc imobilizările financiare"
"account_group_168","Interest on loans and similar liabilities","168","","Dobânzi aferente împrumuturilor și datoriilor asimilate"
"account_group_169","Premiums on repayment of bonds and other liabilities","169","","Prime privind rambursarea obligațiunilor și a altor datorii"
"account_group_2","FIXED ACCOUNTS","2","","CONTURI DE IMOBILIZĂRI"
"account_group_20","Intangible assets","20","","Imobilizări necorporale"
"account_group_207","Goodwill","207","","Fond comercial"
"account_group_21","Tangible assets","21","","Imobilizări corporale"
"account_group_211","Land and land development","211","","Terenuri și amenajări de terenuri"
"account_group_213","Technical installations and means of transport","213","","Instalații tehnice și mijloace de transport"
"account_group_22","Tangible assets in progress","22","","Imobilizări corporale în curs de aprovizionare"
"account_group_23","Assets in progress","23","","Imobilizări în curs"
"account_group_26","Financial assets","26","","Imobilizări financiare"
"account_group_267","Fixed receivables","267","","Creanțe imobilizate"
"account_group_269","Payments to be made for financial assets","269","","Vărsăminte de efectuat pentru imobilizări financiare"
"account_group_28","Depreciation on fixed assets","28","","Amortizări privind imobilizările"
"account_group_280","Depreciation on intangible assets","280","","Amortizări privind imobilizările necorporale"
"account_group_281","Depreciation on tangible assets","281","","Amortizări privind imobilizările corporale"
"account_group_29","Adjustments for depreciation or loss of value of fixed assets","29","","Ajustări pentru deprecierea sau pierderea de valoare a imobilizărilor"
"account_group_290","Adjustments for impairment of intangible assets","290","","Ajustări pentru deprecierea imobilizărilor necorporale"
"account_group_291","Adjustments for depreciation of tangible assets","291","","Ajustări pentru deprecierea imobilizărilor corporale"
"account_group_293","Adjustments for impairment of assets under construction","293","","Ajustări pentru deprecierea imobilizărilor în curs de execuție"
"account_group_296","Adjustments for the loss of value of financial assets","296","","Ajustări pentru pierderea de valoare a imobilizărilor financiare"
"account_group_3","STOCK ACCOUNTS AND WORK IN PROGRESS","3","","CONTURI DE STOCURI ȘI PRODUCȚIE ÎN CURS DE EXECUȚIE"
"account_group_30","Stocks of raw materials and materials","30","","Stocuri de materii prime și materiale"
"account_group_302","Consumables","302","","Materiale consumabile"
"account_group_32","Stocks in progress","32","","Stocuri în curs de aprovizionare"
"account_group_33","Production in progress","33","","Producție în curs de execuție"
"account_group_34","Products","34","","Produse"
"account_group_35","Stock held by third parties","35","","Stocuri aflate la terți"
"account_group_36","Biological assets","36","","Active biologice de natura stocurilor"
"account_group_37","Merchandise","37","","Mărfuri"
"account_group_38","Packing","38","",""
"account_group_39","Adjustments for impairment of inventories and work in progress","39","","Ajustări pentru deprecierea stocurilor și producției în curs de execuție"
"account_group_392","Adjustments for material impairment","392","","Ajustări pentru deprecierea materialelor"
"account_group_393","Adjustments for impairment of work in progress","393","","Ajustări pentru deprecierea producției în curs de execuție"
"account_group_394","Adjustments for depreciation of products","394","","Ajustări pentru deprecierea produselor"
"account_group_395","Adjustments for impairment of inventories held by third parties","395","","Ajustări pentru deprecierea stocurilor aflate la terți"
"account_group_4","THIRD PARTY ACCOUNTS","4","","CONTURI DE TERȚI"
"account_group_40","Providers and assimilated accounts","40","","Furnizori și conturi asimilate"
"account_group_409","Providers - debtors","409","","Furnizori - debitori"
"account_group_41","Customers and assimilated accounts","41","","Clienți și conturi asimilate"
"account_group_411","Customers","411","","Clienți"
"account_group_42","Personal and assimilated accounts","42","","Personal și conturi asimilate"
"account_group_428","Other liabilities and receivables related to personnel","428","","Alte datorii și creanțe în legătură cu personalul"
"account_group_43","Social insurance, social protection and related accounts","43","","Asigurări sociale, protecția socială și conturi asimilate"
"account_group_431","Social Security","431","","Asigurări sociale"
"account_group_437","Unemployment benefits","437","","Ajutor de șomaj"
"account_group_438","Other social liabilites and receivables","438","","Alte datorii și creanțe sociale"
"account_group_44","The state budget, special funds and similar accounts","44","","Bugetul statului, fonduri speciale și conturi asimilate"
"account_group_441","Income and profit taxes","441","","Impozitul pe profit/venit"
"account_group_442","Value added tax","442","","Taxa pe valoarea adăugată"
"account_group_4428","Non-chargeable VAT","4428","","TVA neexigibilă"
"account_group_445","Subsidies","445","","Subvenții"
"account_group_448","Other liabilites and receivables with the state budget","448","","Alte datorii și creanțe cu bugetul statului"
"account_group_45","Group and associated shareholders","45","","Grup și acționari/asociați"
"account_group_451","Settlements between related entities","451","","Decontări între entitățile afiliate"
"account_group_453","Settlements with associates and jointly controlled entities","453","","Decontări cu entitățile asociate și entitățile controlate în comun"
"account_group_455","Amounts owed to the shareholders of the associates","455","","Sume datorate acționarilor/asociaților"
"account_group_458","Settlements from joint ventures","458","","Decontări din operațiuni în participație"
"account_group_46","Misc. Debtors and Creditors","46","","Debitori și creditori diverși"
"account_group_466","Settlements from trust operations","466","","Decontări din operațiuni de fiducie"
"account_group_47","Subsidy, regularization and similar accounts","47","","Conturi de subvenții, regularizare și asimilate"
"account_group_475","Subsidies for investments","475","","Subvenții pentru investiții"
"account_group_48","Settlements within the unit","48","","Decontări în cadrul unității"
"account_group_49","Adjustments for impairment of receivables","49","","Ajustări pentru deprecierea creanțelor"
"account_group_5","TREASURY ACCOUNTS","5","","CONTURI DE TREZORERIE"
"account_group_50","Short-term investments","50","","Investiții pe termen scurt"
"account_group_508","Other short-term investments and similar receivables","508","","Alte investiții pe termen scurt și creanțe asimilate"
"account_group_509","Payments to be made for short-term investments","509","","Vărsăminte de efectuat pentru investițiile pe termen scurt"
"account_group_51","Bank accounts","51","","Conturi la bănci"
"account_group_511","Receivables","511","","Valori de încasat"
"account_group_512","Receivables","512","","Valori de încasat"
"account_group_518","Receivables","518","","Valori de încasat"
"account_group_519","Receivables","519","","Valori de încasat"
"account_group_53","Cash","53","","Casa"
"account_group_531","Cash","531","","Casa"
"account_group_532","Other valuables","532","","Alte valori"
"account_group_54","Letters of credit","54","","Acreditive"
"account_group_541","Letters of credit","541","","Acreditive"
"account_group_58","Internal transfers","58","","Viramente interne"
"account_group_59","Adjustments for impairment of treasury accounts","59","","Ajustări pentru pierderea de valoare a conturilor de trezorerie"
"account_group_6","EXPENSE ACCOUNTS","6","","CONTURI DE CHELTUIELI"
"account_group_60","Inventory Expenses","60","","Cheltuieli privind stocurile"
"account_group_602","Expenses with consumables","602","","Cheltuieli cu materialele consumabile"
"account_group_61","Expenses for services performed by third parties","61","","Cheltuieli cu serviciile executate de terți"
"account_group_62","Costs of other services performed by third parties","62","","Cheltuieli cu alte servicii executate de terți"
"account_group_623","Protocol expenses, advertising and publicity","623","","Cheltuieli de protocol, reclama si publicitate"
"account_group_63","Expenses with other taxes, fees and similar payments","63","","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate"
"account_group_635","Expenses with other taxes, fees and similar payments","635","","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate"
"account_group_64","Staff costs","64","","Cheltuieli cu personalul"
"account_group_642","Expenses with benefits in kind and vouchers granted to employees","642","","Cheltuieli cu avantajele în natură și tichetele acordate salariaților"
"account_group_645","Expenditure on insurance and social protection","645","","Cheltuieli privind asigurările și protecția socială"
"account_group_65","Other operating expenses","65","","Alte cheltuieli de exploatare"
"account_group_651","Expenses from trust operations","651","","Cheltuieli din operațiuni de fiducie"
"account_group_658","Other operating expenses","658","","Alte cheltuieli de exploatare"
"account_group_6588","Other operating expenses","6588","","Alte cheltuieli de exploatare"
"account_group_66","Financial expenses","66","","Cheltuieli financiare"
"account_group_664","Expenditure on divested financial investments","664","","Cheltuieli privind investițiile financiare cedate"
"account_group_665","Expenses from exchange rate differences","665","","Cheltuieli din diferențe de curs valutar"
"account_group_68","Depreciation expense, provisions and adjustments for impairment or loss of value","68","","Cheltuieli cu amortizările, provizioanele și ajustările pentru depreciere sau pierdere de valoare"
"account_group_681","Operating expenses related to depreciation, provisions and impairment adjustments","681","","Cheltuieli de exploatare privind amortizările, provizioanele și ajustările pentru depreciere"
"account_group_686","Finance charges on depreciation, provisions and impairment adjustments","686","","Cheltuieli financiare privind amortizările, provizioanele și ajustările pentru pierdere de valoare"
"account_group_69","Income tax and other tax expenses","69","","Cheltuieli cu impozitul pe profit și alte impozite"
"account_group_7","REVENUE ACCOUNTS","7","","CONTURI DE VENITURI"
"account_group_70","Net turnover","70","","Cifra de afaceri netă"
"account_group_701","Revenues from the sale of finished products, agricultural products and biological assets in the nature of stocks","701","","Venituri din vânzarea produselor finite, produselor agricole și a activelor biologice de natura stocurilor"
"account_group_71","Revenues related to the cost of production in progress","71","","Venituri aferente costului producției în curs de execuție"
"account_group_72","Income from the production of fixed assets","72","","Venituri din producția de imobilizări"
"account_group_74","Income from operating subsidies","74","","Venituri din subvenții de exploatare"
"account_group_741","Income from operating subsidies","741","","Venituri din subvenții de exploatare"
"account_group_75","Other operating revenues","75","","Alte venituri din exploatare"
"account_group_751","Income from trust operations","751","","Venituri din derularea operațiunilor de fiducie"
"account_group_758","Other operating revenues","758","","Alte venituri din exploatare"
"account_group_76","Financial income","76","","Venituri financiare"
"account_group_761","Income from financial assets","761","","Venituri din imobilizări financiare"
"account_group_764","Income from divested financial investments","764","","Venituri din investiții financiare cedate"
"account_group_765","Income from exchange rate differences","765","","Venituri din diferențe de curs valutar"
"account_group_78","Income from provisions, amortization and adjustments for depreciation or impairment","78","","Venituri din provizioane, amortizări și ajustări pentru depreciere sau pierdere de valoare"
"account_group_781","Income from provisions and impairment adjustments related to operating activity","781","","Venituri din provizioane și ajustări pentru depreciere privind activitatea de exploatare"
"account_group_786","Financial income from depreciation and impairment adjustments","786","","Venituri financiare din amortizări și ajustări pentru pierdere de valoare"

```

## File: data\template\account.tax-ro.csv

```csv
"id","sequence","description","invoice_label","name","amount","amount_type","type_tax_use","tax_group_id","tax_exigibility","cash_basis_transition_account_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@ro"
"tvac_00","4","VAT collected 0% Goods","VAT collected 0% Goods","0% G","0.0","percent","sale","tax_group_tva_0","","","base","invoice","+14 - TAX BASE","","","TVA colectat 0% Produse"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-14 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvac_05","3","VAT collected 5% Goods","VAT collected 5% Goods","5% G","5.0","percent","sale","tax_group_tva_5","","","base","invoice","+11 - TAX BASE","","","TVA colectat 5% Produse"
"","","","","","","","","","","","tax","invoice","+11 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","-11 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-11 - VAT","pcg_4427","",""
"tvac_09","2","VAT collected 9% Goods","VAT collected 9% Goods","9% G","9.0","percent","sale","tax_group_tva_9","","","base","invoice","+10 - TAX BASE","","","TVA colectat 9% Produse"
"","","","","","","","","","","","tax","invoice","+10 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","-10 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-10 - VAT","pcg_4427","",""
"tvac_19","1","VAT collected 19% Goods","VAT collected 19% Goods","19% G","19.0","percent","sale","tax_group_tva_19","","","base","invoice","+09 - TAX BASE","","","TVA colectat 19% Produse"
"","","","","","","","","","","","tax","invoice","+09 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","-09 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-09 - VAT","pcg_4427","",""
"tvac_00_s","4","VAT collected 0% Services","VAT collected 0% Services","0% S","0.0","percent","sale","tax_group_tva_0","","","base","invoice","+14 - TAX BASE","","","TVA colectat 0% Servicii"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-14 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvac_05_s","3","VAT collected 5% Services","VAT collected 5% Services","5% S","5.0","percent","sale","tax_group_tva_5","","","base","invoice","+11 - TAX BASE","","","TVA colectat 5% Servicii"
"","","","","","","","","","","","tax","invoice","+11 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","-11 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-11 - VAT","pcg_4427","",""
"tvac_09_s","2","VAT collected 9% Services","VAT collected 9% Services","9% S","9.0","percent","sale","tax_group_tva_9","","","base","invoice","+10 - TAX BASE","","","TVA colectat 9% Servicii"
"","","","","","","","","","","","tax","invoice","+10 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","-10 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-10 - VAT","pcg_4427","",""
"tvac_19_s","1","VAT collected 19% Services","VAT collected 19% Services","19% S","19.0","percent","sale","tax_group_tva_19","","","base","invoice","+09 - TAX BASE","","","TVA colectat 19% Servicii"
"","","","","","","","","","","","tax","invoice","+09 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","-09 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-09 - VAT","pcg_4427","",""
"tvad_00","43","VAT deductible 0% Goods","VAT deductible 0% Goods","0% G","0.0","percent","purchase","tax_group_tva_0","","","base","invoice","+30 - TAX BASE","","","TVA deductibil 0% Produse"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvad_05","42","VAT deductible 5% Goods","VAT deductible 5% Goods","5% G","5.0","percent","purchase","tax_group_tva_0","","","base","invoice","+26_1 - TAX BASE","","","TVA deductibil 5% Produse"
"","","","","","","","","","","","tax","invoice","+26_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-26_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-26_1 - VAT","pcg_4426","",""
"tvad_09","41","VAT deductible 9% Goods","VAT deductible 9% Goods","9% G","9.0","percent","purchase","tax_group_tva_0","","","base","invoice","+25_1 - TAX BASE","","","TVA deductibil 9% Produse"
"","","","","","","","","","","","tax","invoice","+25_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-25_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-25_1 - VAT","pcg_4426","",""
"tvad_19","40","VAT deductible 19% Goods","VAT deductible 19% Goods","19% G","19.0","percent","purchase","tax_group_tva_19","","","base","invoice","+24_1 - TAX BASE","","","TVA deductibil 19% Produse"
"","","","","","","","","","","","tax","invoice","+24_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-24_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-24_1 - VAT","pcg_4426","",""
"tvad_00_s","43","VAT deductible 0% Services","VAT deductible 0% Services","0% S","0.0","percent","purchase","tax_group_tva_0","","","base","invoice","+30 - TAX BASE","","","TVA deductibil 0% Servicii"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvad_05_s","42","VAT deductible 5% Services","VAT deductible 5% Services","5% S","5.0","percent","purchase","tax_group_tva_0","","","base","invoice","+26_1 - TAX BASE","","","TVA deductibil 5% Servicii"
"","","","","","","","","","","","tax","invoice","+26_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-26_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-26_1 - VAT","pcg_4426","",""
"tvad_09_s","41","VAT deductible 9% Services","VAT deductible 9% Services","9% S","9.0","percent","purchase","tax_group_tva_0","","","base","invoice","+25_1 - TAX BASE","","","TVA deductibil 9% Servicii"
"","","","","","","","","","","","tax","invoice","+25_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-25_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-25_1 - VAT","pcg_4426","",""
"tvad_19_s","40","VAT deductible 19% Services","VAT deductible 19% Services","19% S","19.0","percent","purchase","tax_group_tva_19","","","base","invoice","+24_1 - TAX BASE","","","TVA deductibil 19% Servicii"
"","","","","","","","","","","","tax","invoice","+24_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-24_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-24_1 - VAT","pcg_4426","",""
"tvaic_00","13","VAT on collection - 0% collected","VAT collected 0%","0%","0.0","percent","sale","tax_group_tva_0","","","base","invoice","+14 - TAX BASE","","","TVA la colectare - 0% colectat"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-14 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvaic_05","12","VAT on collection - collected 5%","VAT collected 5%","5%","5.0","percent","sale","tax_group_tva_5","on_payment","pcg_44281","base","invoice","-11 - TAX BASE","","","TVA la colectare - colectat 5%"
"","","","","","","","","","","","tax","invoice","-11 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","+11 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+11 - VAT","pcg_4427","",""
"tvaic_09","11","VAT on collection - collected 9%","VAT collected 9%","9%","9.0","percent","sale","tax_group_tva_9","on_payment","pcg_44281","base","invoice","-10 - TAX BASE","","","TVA la colectare - colectat 9%"
"","","","","","","","","","","","tax","invoice","-10 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","+10 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+10 - VAT","pcg_4427","",""
"tvaic_19","10","VAT on collection - collected 19%","VAT collected 19%","19%","19.0","percent","sale","tax_group_tva_19","on_payment","pcg_44281","base","invoice","-09 - TAX BASE","","","TVA la colectare - colectat 19%"
"","","","","","","","","","","","tax","invoice","-09 - VAT","pcg_4427","",""
"","","","","","","","","","","","base","refund","+09 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+09 - VAT","pcg_4427","",""
"tvaid_00","53","VAT on collection - 0% deductible","0% deductible VAT","0%","0.0","percent","purchase","tax_group_tva_0","","","base","invoice","+30 - TAX BASE","","","TVA la colectare - 0% deductibil"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvaid_05","52","VAT on collection - 5% deductible","5% VAT deductible","5%","5.0","percent","purchase","tax_group_tva_5","on_payment","pcg_44282","base","invoice","+26_1 - TAX BASE","","","TVA la colectare - 5% deductibil"
"","","","","","","","","","","","tax","invoice","+26_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-26_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-26_1 - VAT","pcg_4426","",""
"tvaid_09","51","VAT on collection - 9% deductible","9% VAT deductible","9%","9.0","percent","purchase","tax_group_tva_9","on_payment","pcg_44282","base","invoice","+25_1 - TAX BASE","","","TVA la colectare - 9% deductibil"
"","","","","","","","","","","","tax","invoice","+25_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-25_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-25_1 - VAT","pcg_4426","",""
"tvaid_19","50","VAT on collection - 19% deductible","19% VAT deductible","19%","19.0","percent","purchase","tax_group_tva_19","on_payment","pcg_44282","base","invoice","+24_1 - TAX BASE","","","TVA la colectare - 19% deductibil"
"","","","","","","","","","","","tax","invoice","+24_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-24_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","-24_1 - VAT","pcg_4426","",""
"tvati","20","VAT Reverse Tax","VAT Reverse Tax","0% R","0.0","percent","sale","tax_group_tva_ti","","","base","invoice","+13 - TAX BASE","","","TVA Taxare Inversa"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-13 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvatip00","63","VAT Reverse Tax 0%","VAT Reverse Tax 0%","0% R","0.0","percent","purchase","tax_group_tva_ti","","","base","invoice","+30 - TAX BASE","","","TVA Taxare Inversa 0%"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvatip05","62","VAT Reverse Tax 5%","VAT Reverse Tax 5%","5% R","5.0","percent","purchase","tax_group_tva_ti","","","base","invoice","+12_3 - TAX BASE||+27_3 - TAX BASE","","","TVA Taxare Inversa 5%"
"","","","","","","","","","","","tax","invoice","-12_3 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+27_3 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-12_3 - TAX BASE||-27_3 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+12_3 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","refund","-27_3 - VAT","pcg_4426","",""
"tvatip09","61","VAT Reverse Tax 9%","VAT Reverse Tax 9%","9% R","9.0","percent","purchase","tax_group_tva_ti","","","base","invoice","+12_2 - TAX BASE||+27_2 - TAX BASE","","","TVA Taxare Inversa 9%"
"","","","","","","","","","","","tax","invoice","-12_2 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+27_2 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-12_2 - TAX BASE||-27_2 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+12_2 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","refund","-27_2 - VAT","pcg_4426","",""
"tvatip19","60","VAT Reverse Tax 19%","VAT Reverse Tax 19%","19% R","19.0","percent","purchase","tax_group_tva_ti","","","base","invoice","+12_1 - TAX BASE||+27_1 - TAX BASE","","","TVA Taxare Inversa 19%"
"","","","","","","","","","","","tax","invoice","-12_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+27_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-12_1 - TAX BASE||-27_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+12_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","refund","-27_1 - VAT","pcg_4426","",""
"tvatiscded","31","VAT Tax exempt Deliveries with the right to deduct","Exempt-L1","0% EXEMPT D","0.0","percent","sale","tax_group_tva_scutit","","","base","invoice","+14 - TAX BASE","","","TVA Livrări exempționate cu dreptul de a scădea"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-14 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvatiscnoded","32","VAT Tax exempt Deliveries without the right to deduct","Exempt-L2","0% EXEMPT ND","0.0","percent","sale","tax_group_tva_scutit","","","base","invoice","+15 - TAX BASE","","","TVA Livrări exempționate fără drept de a scădea"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-15 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvatiscdeda","61","VAT Tax Exemption Acquisitions","Exempt-A","0% EXEMPT","0.0","percent","purchase","tax_group_tva_scutit","","","base","invoice","+30 - TAX BASE","","","TVA Cumpărări Exempționate"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvatiscdeda_intr","61","VAT Exempt Taxation Intra-Community Acquisitions","Exempt-AI","0% EU EXEMPT","0.0","percent","purchase","tax_group_tva_scutit","","","base","invoice","+30 - TAX BASE||+30_1 - TAX BASE","","","TVA Scutit Taxare Intra-comunitare Achizitii"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE||-30_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvatine","33","VAT Non-Taxable Taxes Deliveries","Non-Taxable-L","0% NT","0.0","percent","sale","tax_group_tva_neimp","","","base","invoice","+15 - TAX BASE","","","TVA Neimpozabile Taxe Livrari"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-15 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvatinea","62","VAT Tax Non-Taxable Acquisitions","Non-Taxable-A","0% NT","0.0","percent","purchase","tax_group_tva_neimp","","","base","invoice","+30 - TAX BASE","","","TVA Cumpărări Neimpozabile"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_intrab","34","Intra-Community VAT Goods Delivery","","0% EU G","0.0","percent","sale","","","","base","invoice","+01 - TAX BASE","","","Livrare intracomunitara de bunuri cu TVA"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-01 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_intras","34","Intra-Community VAT Delivery Services","","0% EU S","0.0","percent","sale","","","","base","invoice","+03 - TAX BASE","","","Servicii de livrare intracomunitare cu TVA"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-03 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_extra","35","VAT Exportation Goods","","0% EX G","0.0","percent","sale","tax_group_tva_scutit","","","base","invoice","+14 - TAX BASE","","","TVA Export Bunuri"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-14 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_extras","35","VAT Exportation Services","","0% EX S","0.0","percent","sale","tax_group_tva_scutit","","","base","invoice","+14 - TAX BASE","","","TVA Export Servicii"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-14 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_intrap0b","66","Intra-Community VAT Acquisitions 0% - Goods","","0% EU G","0.0","percent","purchase","tax_group_tva_0_eu","","","base","invoice","+30 - TAX BASE","","","Achizitii intracomunitare TVA 0% - Bunuri"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_intrap5b","65","Intra-Community VAT Acquisitions 5% - Goods","","5% EU G","5.0","percent","purchase","tax_group_tva_5_eu","","","base","invoice","+05 - TAX BASE||+05_1 - TAX BASE||+20 - TAX BASE||+20_1 - TAX BASE","","","Achizitii intracomunitare TVA 5% - Bunuri"
"","","","","","","","","","","","tax","invoice","-05 - VAT||-05_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+20 - VAT||+20_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-05 - TAX BASE||-05_1 - TAX BASE||-20 - TAX BASE||-20_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+05 - VAT||+05_1 - VAT","pcg_4426","-100",""
"","","","","","","","","","","","tax","refund","-20 - VAT||-20_1 - VAT","pcg_4427","",""
"tvati_intrap9b","64","Intra-Community VAT Acquisitions 9% - Goods","","9% EU G","9.0","percent","purchase","tax_group_tva_9_eu","","","base","invoice","+05 - TAX BASE||+05_1 - TAX BASE||+20 - TAX BASE||+20_1 - TAX BASE","","","Achizitii intracomunitare TVA 9% - Bunuri"
"","","","","","","","","","","","tax","invoice","-05 - VAT||-05_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+20 - VAT||+20_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-05 - TAX BASE||-05_1 - TAX BASE||-20 - TAX BASE||-20_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+05 - VAT||+05_1 - VAT","pcg_4426","-100",""
"","","","","","","","","","","","tax","refund","-20 - VAT||-20_1 - VAT","pcg_4427","",""
"tvati_intrap19b","63","Intra-Community VAT Acquisitions 19% - Goods","","19% EU G","19.0","percent","purchase","tax_group_tva_19_eu","","","base","invoice","+05 - TAX BASE||+05_1 - TAX BASE||+20 - TAX BASE||+20_1 - TAX BASE","","","Achizitii intracomunitare TVA 19% - Bunuri"
"","","","","","","","","","","","tax","invoice","-05 - VAT||-05_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+20 - VAT||+20_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-05 - TAX BASE||-05_1 - TAX BASE||-20 - TAX BASE||-20_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+05 - VAT||+05_1 - VAT","pcg_4426","-100",""
"","","","","","","","","","","","tax","refund","-20 - VAT||-20_1 - VAT","pcg_4427","",""
"tvati_intrap0s","70","Intra-Community VAT Acquisitions 0% - Services","","0% EU S","0.0","percent","purchase","tax_group_tva_0_eu","","","base","invoice","+30 - TAX BASE","","","Achizitii intracomunitare TVA 0% - Servicii"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-30 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_intrap5s","69","Intra-Community VAT Acquisitions 5% - Services","","5% EU S","5.0","percent","purchase","tax_group_tva_5_eu","","","base","invoice","+07 - TAX BASE||+07_1 - TAX BASE||+22 - TAX BASE||+22_1 - TAX BASE","","","Achizitii intracomunitare TVA 5% - Servicii"
"","","","","","","","","","","","tax","invoice","-07 - VAT||-07_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+22 - VAT||+22_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-07 - TAX BASE||-07_1 - TAX BASE||-22 - TAX BASE||-22_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+07 - VAT||+07_1 - VAT","pcg_4426","-100",""
"","","","","","","","","","","","tax","refund","-22 - VAT||-22_1 - VAT","pcg_4427","",""
"tvati_intrap9s","68","Intra-Community VAT Acquisitions 9% - Services","","9% EU S","9.0","percent","purchase","tax_group_tva_9_eu","","","base","invoice","+07 - TAX BASE||+07_1 - TAX BASE||+22 - TAX BASE||+22_1 - TAX BASE","","","Achizitii intracomunitare TVA 9% - Servicii"
"","","","","","","","","","","","tax","invoice","-07 - VAT||-07_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+22 - VAT||+22_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-07 - TAX BASE||-07_1 - TAX BASE||-22 - TAX BASE||-22_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+07 - VAT||+07_1 - VAT","pcg_4426","-100",""
"","","","","","","","","","","","tax","refund","-22 - VAT||-22_1 - VAT","pcg_4427","",""
"tvati_intrap19s","67","Intra-Community VAT Acquisitions 19% - Services","","19% EU S","19.0","percent","purchase","tax_group_tva_19_eu","","","base","invoice","+07 - TAX BASE||+07_1 - TAX BASE||+22 - TAX BASE||+22_1 - TAX BASE","","","Achizitii intracomunitare TVA 19% - Servicii"
"","","","","","","","","","","","tax","invoice","-07 - VAT||-07_1 - VAT","pcg_4427","-100",""
"","","","","","","","","","","","tax","invoice","+22 - VAT||+22_1 - VAT","pcg_4426","",""
"","","","","","","","","","","","base","refund","-07 - TAX BASE||-07_1 - TAX BASE||-22 - TAX BASE||-22_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+07 - VAT||+07_1 - VAT","pcg_4426","-100",""
"","","","","","","","","","","","tax","refund","-22 - VAT||-22_1 - VAT","pcg_4427","",""
"tvati_extrap0b","66","VAT Import Goods","","0% EX G","0.0","percent","purchase","tax_group_tva_scutit","","","base","invoice","+07 - TAX BASE||+07_1 - TAX BASE||+22 - TAX BASE||+22_1 - TAX BASE","","","TVA Import Produse"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-07 - TAX BASE||-07_1 - TAX BASE||-22 - TAX BASE||-22_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tvati_extrap0s","70","VAT Import Services","","0% EX S","0.0","percent","purchase","tax_group_tva_scutit","","","base","invoice","+07 - TAX BASE||+07_1 - TAX BASE||+22 - TAX BASE||+22_1 - TAX BASE","","","TVA Import Servicii"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-07 - TAX BASE||-07_1 - TAX BASE||-22 - TAX BASE||-22_1 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","","","",""
"tva_ned_5_50","72","VAT 5% Non-deductible 50%","VAT 5% Non-deductible 50%","5% ND 50%","5.0","percent","purchase","tax_group_tva_ned","","","base","invoice","+26_2 - TAX BASE","","","TVA 5% Nedeductibil 50%"
"","","","","","","","","","","","tax","invoice","-26_2 - VAT","pcg_4426","50",""
"","","","","","","","","","","","tax","invoice","","pcg_6352","50",""
"","","","","","","","","","","","base","refund","-26_2 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+26_2 - VAT","pcg_4426","50",""
"","","","","","","","","","","","tax","refund","","pcg_6352","50",""
"tva_ned_9_50","71","VAT 9% Non-deductible 50%","VAT 9% Non-deductible 50%","9% ND 50%","9.0","percent","purchase","tax_group_tva_ned","","","base","invoice","+25_2 - TAX BASE","","","TVA 9% Nedeductibil 50%"
"","","","","","","","","","","","tax","invoice","-25_2 - VAT","pcg_4426","50",""
"","","","","","","","","","","","tax","invoice","","pcg_6352","50",""
"","","","","","","","","","","","base","refund","-25_2 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+25_2 - VAT","pcg_4426","50",""
"","","","","","","","","","","","tax","refund","","pcg_6352","50",""
"tva_ned_19_50","70","VAT 19% Non-deductible 50%","VAT 19% Non-deductible 50%","19% ND 50%","19.0","percent","purchase","tax_group_tva_ned","","","base","invoice","+24_2 - TAX BASE","","","TVA 19% Nedeductibil 50%"
"","","","","","","","","","","","tax","invoice","-24_2 - VAT","pcg_4426","50",""
"","","","","","","","","","","","tax","invoice","","pcg_6352","50",""
"","","","","","","","","","","","base","refund","-24_2 - TAX BASE","","",""
"","","","","","","","","","","","tax","refund","+24_2 - VAT","pcg_4426","50",""
"","","","","","","","","","","","tax","refund","","pcg_6352","50",""

```

## File: data\template\account.tax.group-ro.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@ro"
"tax_group_tva_0","VAT 0%","base.ro","pcg_4424","pcg_44231","TVA 0%"
"tax_group_tva_5","VAT 5%","base.ro","pcg_4424","pcg_44231","TVA 5%"
"tax_group_tva_9","VAT 9%","base.ro","pcg_4424","pcg_44231","TVA 9%"
"tax_group_tva_19","VAT 19%","base.ro","pcg_4424","pcg_44231","TVA 19%"
"tax_group_tva_20","VAT 20%","base.ro","pcg_4424","pcg_44231","TVA 20%"
"tax_group_tva_24","VAT 24%","base.ro","pcg_4424","pcg_44231","TVA 24%"
"tax_group_tva_0_eu","VAT 0% EU","base.ro","pcg_4424","pcg_44231","TVA 0% EU"
"tax_group_tva_5_eu","VAT 5% EU","base.ro","pcg_4424","pcg_44231","TVA 5% EU"
"tax_group_tva_9_eu","VAT 9% EU","base.ro","pcg_4424","pcg_44231","TVA 9% EU"
"tax_group_tva_19_eu","VAT 19% EU","base.ro","pcg_4424","pcg_44231","TVA 19% EU"
"tax_group_tva_ned","Non-deductible VAT","base.ro","pcg_4424","pcg_44231","TVA Nedeductibil"
"tax_group_tva_ti","VAT Reverse Charge","base.ro","pcg_4424","pcg_44231","TVA Taxare Inversa"
"tax_group_tva_scutit","VAT Exempt","base.ro","pcg_4424","pcg_44231","TVA Scutit"
"tax_group_tva_neimp","VAT Non taxable","base.ro","pcg_4424","pcg_44231","TVA Neimpozabil"

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# @author -  Fekete Mihai <feketemihai@gmail.com>
# Copyright (C) 2020 NextERP Romania (https://www.nexterp.ro) <contact@nexterp.ro>
# Copyright (C) 2015 Forest and Biomass Services Romania (http://www.forbiom.eu).
# Copyright (C) 2011 TOTAL PC SYSTEMS (http://www.erpsystems.ro).
# Copyright (C) 2009 (<http://www.filsystem.ro>)

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = "res.partner"

    @api.model
    def _commercial_fields(self):
        return super(ResPartner, self)._commercial_fields() + ['nrc']

    nrc = fields.Char(string='NRC', help='Registration number at the Registry of Commerce')

    @api.depends('vat', 'country_id')
    def _compute_company_registry(self):
        # OVERRIDE
        # In Romania, if you have a VAT number, it's also your company registry (CUI) number
        super()._compute_company_registry()
        for partner in self.filtered(lambda p: p.country_id.code == 'RO' and p.vat):
            vat_country, vat_number = self._split_vat(partner.vat)
            if vat_country.isnumeric():
                vat_country = 'ro'
                vat_number = partner.vat
            if vat_country == 'ro' and self.simple_vat_check(vat_country, vat_number):
                partner.company_registry = vat_number

```

## File: models\template_ro.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, Command
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ro')
    def _get_ro_template_data(self):
        return {
            'property_account_receivable_id': 'ro_pcg_recv',
            'property_account_payable_id': 'pcg_4011',
            'property_account_expense_categ_id': 'ro_pcg_expense',
            'property_account_income_categ_id': 'ro_pcg_sale',
            'code_digits': '6',
            'use_storno_accounting': True,
        }

    @template('ro', 'res.company')
    def _get_ro_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ro',
                'bank_account_code_prefix': '5121',
                'cash_account_code_prefix': '5311',
                'transfer_account_code_prefix': '581',
                'account_default_pos_receivable_account_id': 'ro_pcg_recv',
                'income_currency_exchange_account_id': 'pcg_7651',
                'expense_currency_exchange_account_id': 'pcg_6651',
                'account_journal_suspense_account_id': 'pcg_5125',
                'account_journal_early_pay_discount_loss_account_id': 'pcg_6092',
                'account_journal_early_pay_discount_gain_account_id': 'pcg_709',
                'account_sale_tax_id': 'tvac_19',
                'account_purchase_tax_id': 'tvad_19',
            },
        }

    @template('ro', 'account.reconcile.model')
    def _get_ro_reconcile_model(self):
        return {
            'suppadvance_template': {
                'name': 'Avans Furnizor - Imobilizări Necorporale',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_4094',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Supplier Advance - Intangible Assets',
                    }),
                ],
            },
            'custadvance_template': {
                'name': 'Customer Advances',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_419',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Customer Advances',
                    }),
                ],
            },
            'bankcomm_template': {
                'name': 'Bank Commission',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_627',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Bank Commission',
                    }),
                ],
            },
            'interest_template': {
                'name': 'Interests',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_766',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Interests',
                    }),
                ],
            },
            'inttransfer_template': {
                'name': 'Internal transfer',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_581',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Internal transfer',
                    }),
                ],
            },
            'payroll_template': {
                'name': 'Wages',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_421',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Wages',
                    }),
                ],
            },
            'pendsettl_template': {
                'name': 'Operations being clarified',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_473',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Operations being clarified',
                    }),
                ],
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ro
from . import res_partner

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
		<record model="ir.ui.view" id="res_partner_form_ro">
			<field name="name">res.partner.form.ro</field>
			<field name="inherit_id" ref="account.view_partner_property_form"/>
			<field name="model">res.partner</field>
			<field name="arch" type="xml">
				<field name="vat" position="after">
					<field name="nrc" placeholder="e.g. J12/1234/2012" class="oe_inline" invisible="'RO' not in fiscal_country_codes"/>
                </field>
			</field>
		</record>
</odoo>

```

