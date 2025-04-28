# Odoo Module: l10n_ee

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
    'name': 'Estonia - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'version': '1.2',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ee'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Estonia in Odoo.
    """,
    'author': 'Odoo SA',
    'depends': [
        'account',
    ],
    'data': [
        'data/account_tax_report_data.xml',
        'views/account_tax_form.xml',
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
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">VAT Report (KMD)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ee"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_1" model="account.report.line">
                <field name="name">1 - Acts and transactions subject to tax at a rate of 22%</field>
                <field name="code">l10n_ee_vat_1</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1_base</field>
                    </record>
                    <record id="tax_report_line_1_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1_tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_1_1" model="account.report.line">
                <field name="name">1¹ - Acts and transactions subject to tax at a rate of 20%</field>
                <field name="code">l10n_ee_vat_1_1</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_1_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1_1_base</field>
                    </record>
                    <record id="tax_report_line_1_1_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1_1_tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_2" model="account.report.line">
                <field name="name">2 - Acts and transactions subject to tax at a rate of 9%</field>
                <field name="code">l10n_ee_vat_2</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_2_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_base</field>
                    </record>
                    <record id="tax_report_line_2_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_2_1" model="account.report.line">
                <field name="name">2¹ - Acts and transactions subject to tax at a rate of 5%</field>
                <field name="code">l10n_ee_vat_2_1</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_2_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_1_base</field>
                    </record>
                    <record id="tax_report_line_2_1_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_1_tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_2_2" model="account.report.line">
                <field name="name">2² - Acts and transactions subject to tax at a rate of 13%</field>
                <field name="code">l10n_ee_vat_2_2</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_2_2_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_2_base</field>
                    </record>
                    <record id="tax_report_line_2_2_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_2_tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_3" model="account.report.line">
                <field name="name">3 - Acts and transactions subject to tax at a rate of 0%, incl.</field>
                <field name="code">l10n_ee_vat_3</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_3_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_3.tax_tags + l10n_ee_vat_3_1.balance + l10n_ee_vat_3_2.balance</field>
                    </record>
                    <record id="tax_report_line_3_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">3</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_3_1" model="account.report.line">
                        <field name="name">3.1 - Intra-Community supply of goods and services provided to a taxable person or taxable person with limited liability of another Member State, total, incl.</field>
                        <field name="code">l10n_ee_vat_3_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_3_1_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ee_vat_3_1.tax_tags + l10n_ee_vat_3_1_1.balance</field>
                            </record>
                            <record id="tax_report_line_3_1_tag" model="account.report.expression">
                                <field name="label">tax_tags</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3_1</field>
                            </record>
                            <!-- Not visible, but used for EC Sales Report -->
                            <record id="tax_report_line_ec_services_tag" model="account.report.expression">
                                <field name="label">ec_services</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3_1_S</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_3_1_1" model="account.report.line">
                                <field name="name">3.1.1 - Intra-Community supply of goods</field>
                                <field name="code">l10n_ee_vat_3_1_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_3_1_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_1_1</field>
                                    </record>
                                    <!-- Not visible, but used for EC Sales Report -->
                                    <record id="tax_report_line_ec_goods_tag" model="account.report.expression">
                                        <field name="label">ec_goods</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_1_1_G</field>
                                    </record>
                                    <record id="tax_report_line_ec_triangular_tag" model="account.report.expression">
                                        <field name="label">ec_triangular</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_1_1_T</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_3_2" model="account.report.line">
                        <field name="name">3.2 - Exportation of goods, incl.</field>
                        <field name="code">l10n_ee_vat_3_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_3_2_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ee_vat_3_2.tax_tags + l10n_ee_vat_3_2_1.balance</field>
                            </record>
                            <record id="tax_report_line_3_2_tag" model="account.report.expression">
                                <field name="label">tax_tags</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3_2</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_3_2_1" model="account.report.line">
                                <field name="name">3.2.1 - Sale to passengers with return of value added tax</field>
                                <field name="code">l10n_ee_vat_3_2_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_3_2_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_2_1</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_4" model="account.report.line">
                <field name="name">4 - Total amount of value added tax</field>
                <field name="code">l10n_ee_vat_4</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_4_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_1.balance * 0.22 + l10n_ee_vat_1_1.balance * 0.2 + l10n_ee_vat_2.balance * 0.09 + l10n_ee_vat_2_1.balance * 0.05 + l10n_ee_vat_2_2.balance * 0.13</field>
                    </record>
                    <record id="tax_report_line_4_tag_no_rounding" model="account.report.expression">
                        <field name="label">balance_from_tags</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_1.tax + l10n_ee_vat_1_1.tax + l10n_ee_vat_2.tax + l10n_ee_vat_2_1.tax + l10n_ee_vat_2_2.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_4_1" model="account.report.line">
                <field name="name">4¹ - Value added tax payable upon the import of the goods</field>
                <field name="code">l10n_ee_vat_4_1</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_4_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">4_1_tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_5" model="account.report.line">
                <field name="name">5 - Total amount of input VAT subject to deduction pursuant to law, incl.</field>
                <field name="code">l10n_ee_vat_5</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_5_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_5.tax_tags + l10n_ee_vat_5_1.balance + l10n_ee_vat_5_2.balance + l10n_ee_vat_5_3.balance + l10n_ee_vat_5_4.balance</field>
                    </record>
                    <record id="tax_report_line_5_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">5_tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_5_1" model="account.report.line">
                        <field name="name">5.1 - VAT paid or payable on import</field>
                        <field name="code">l10n_ee_vat_5_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_1_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_5_2" model="account.report.line">
                        <field name="name">5.2 - VAT paid or payable on acquisition of fixed assets</field>
                        <field name="code">l10n_ee_vat_5_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_2_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_5_3" model="account.report.line">
                        <field name="name">5.3 - VAT paid or payable on acquisition of a car used for business purposes (100%), and on acquisition of goods and receipt of services for such car</field>
                        <field name="code">l10n_ee_vat_5_3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_3_tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_5_3_cars" model="account.report.line">
                                <field name="name">Number of cars</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_5_3_cars_value" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=0</field>
                                        <field name="figure_type">integer</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_5_4" model="account.report.line">
                        <field name="name">5.4 - VAT paid or payable on acquisition of a car used partially for business purposes, and on acquisition of goods and receipt of services for such car</field>
                        <field name="code">l10n_ee_vat_5_4</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_4_tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_5_4_cars" model="account.report.line">
                                <field name="name">Number of cars</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_5_4_cars_value" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=0</field>
                                        <field name="figure_type">integer</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_6" model="account.report.line">
                <field name="name">6 - Intra-Community acquisitions of goods and services received from a taxable person of another Member State, total, incl.</field>
                <field name="code">l10n_ee_vat_6</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_6_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_6.tax_tags + l10n_ee_vat_6_1.balance</field>
                    </record>
                    <record id="tax_report_line_6_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">6</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_6_1" model="account.report.line">
                        <field name="name">6.1 - Intra-Community acquisitions of goods</field>
                        <field name="code">l10n_ee_vat_6_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_6_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6_1</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_7" model="account.report.line">
                <field name="name">7 - Acquisition of other goods and services subject to VAT, incl.</field>
                <field name="code">l10n_ee_vat_7</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_7_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_7.tax_tags + l10n_ee_vat_7_1.balance</field>
                    </record>
                    <record id="tax_report_line_7_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">7</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_7_1" model="account.report.line">
                        <field name="name">7.1 - Acquisition of immovables, scrap metal, precious metal and metal products subject to value added tax under the special arrangements (VAT Act §41¹)</field>
                        <field name="code">l10n_ee_vat_7_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_7_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7_1</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_8" model="account.report.line">
                <field name="name">8 - Supply exempt from tax</field>
                <field name="code">l10n_ee_vat_8</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_8_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_9" model="account.report.line">
                <field name="name">9 - Supply of immovables, scrap metal, precious metal and metal products subject to value added tax under the special arrangements (VAT Act §41¹) and taxable value of goods to be installed or assembled in another Member State</field>
                <field name="code">l10n_ee_vat_9</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_9_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">9</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_10" model="account.report.line">
                <field name="name">10 - Adjustments (+)</field>
                <field name="code">l10n_ee_vat_10</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_10_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">10</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_11" model="account.report.line">
                <field name="name">11 - Adjustments (-)</field>
                <field name="code">l10n_ee_vat_11</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_11_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">11</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_12" model="account.report.line">
                <field name="name">12 - Value added tax payable</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_12_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_4.balance + l10n_ee_vat_4_1.balance - l10n_ee_vat_5.balance + l10n_ee_vat_10.balance - l10n_ee_vat_11.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_13" model="account.report.line">
                <field name="name">13 - Overpaid value added tax</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_13_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">-(l10n_ee_vat_4.balance + l10n_ee_vat_4_1.balance - l10n_ee_vat_5.balance + l10n_ee_vat_10.balance - l10n_ee_vat_11.balance)</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-ee.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@et"
"l10n_ee_1000","Cash Accounts","1000","asset_cash","","False","Rahakontod"
"l10n_ee_1001","Bank Accounts","1001","asset_cash","","False","Pangakontod"
"l10n_ee_1008","Transfer Accounts","1008","asset_current","","True","Ülekandekontod"
"l10n_ee_1009","Bank Suspense Account","1009","asset_current","","False","Panga vahekonto"
"l10n_ee_1010","Short-Term Financial Investments","1010","asset_current","","False","Lühiajalised finantsinvesteeringud"
"l10n_ee_1011","Depreciations on Short-Term Financial Investments","1011","asset_current","","False","Lühiajaliste finantsinvesteeringute amortisatsioon"
"l10n_ee_10200","Accounts Receivable","10200","asset_receivable","","True","Nõuded"
"l10n_ee_10201","Accounts Receivable (POS)","10201","asset_receivable","","True","Nõuded (POS)"
"l10n_ee_10202","Doubtful Receivables","10202","asset_receivable","","True","Ebatõenäolised nõuded"
"l10n_ee_1021","Receivables from Related Parties","1021","asset_current","","False","Nõuded seotud osapoolte vastu"
"l10n_ee_1022","Prepaid and Deferred Taxes","1022","asset_current","","False","Maksude ettemaksed ja tagasinõuded"
"l10n_ee_1023","Loan Receivables","1023","asset_current","","False","Laenunõuded"
"l10n_ee_10240","Interest Receivable","10240","asset_current","","False","Saadaolevad intressid"
"l10n_ee_10241","Dividend Receivable","10241","asset_current","","False","Saadaolevad dividendid"
"l10n_ee_10242","Accounts Receivable from Social Insurance Board","10242","asset_current","","False","Nõuded Sotsiaalkindlustusametilt"
"l10n_ee_10243","Netting Account","10243","asset_current","","False","Tasaarvelduskonto"
"l10n_ee_10244","Other Short-Term Receivables","10244","asset_current","","False","Muud lühiajalised nõuded"
"l10n_ee_1025","Prepayments","1025","asset_prepayments","","False","Ettemaksed"
"l10n_ee_1030","Raw and Other Materials","1030","asset_current","","False","Tooraine ja materjal"
"l10n_ee_1031","Work in Progress","1031","asset_current","","False","Lõpetamata toodang"
"l10n_ee_10320","Finished Goods from Agricultural Production","10320","asset_current","","False","Põllumajandustootmisest saadud valmistooted"
"l10n_ee_10321","Other Finished Goods","10321","asset_current","","False","Muud valmistooted"
"l10n_ee_1033","Goods for Resale","1033","asset_current","","False","Müügiks ostetud kaubad"
"l10n_ee_1034","Prepayments to Suppliers","1034","asset_current","","False","Ettemaksed varude eest"
"l10n_ee_1040","Biological Assets","1040","asset_current","","False","Bioloogilised varad"
"l10n_ee_1100","Shares and Participations in Subsidiaries","1100","asset_non_current","","False","Tütarettevõtjate aktsiad ja osad"
"l10n_ee_1101","Shares and Participations in Affiliates","1101","asset_non_current","","False","Sidusettevõtjate aktsiad ja osad"
"l10n_ee_1110","Other Shares, Stocks and Bonds","1110","asset_non_current","","False","Muud aktsiad, aktsiad ja võlakirjad"
"l10n_ee_1111","Other Long-Term Financial Investments","1111","asset_non_current","","False","Muud pikaajalised finantsinvesteeringud"
"l10n_ee_1120","Long-Term Trade Receivables","1120","asset_non_current","","False","Nõuded ostjate vastu"
"l10n_ee_1121","Long-Term Receivables from Related Parties","1121","asset_non_current","","False","Nõuded seotud osapoolte vastu"
"l10n_ee_1122","Long-Term Prepaid and Deferred Taxes","1122","asset_non_current","","False","Maksude ettemaksed ja tagasinõuded"
"l10n_ee_1123","Long-Term Loan Receivables","1123","asset_non_current","","False","Laenunõuded"
"l10n_ee_1124","Other Long-Term Receivables","1124","asset_non_current","","False","Muud nõuded"
"l10n_ee_11250","Constructions in Progress","11250","asset_non_current","","False","Käimasolevad ehitised"
"l10n_ee_11251","Prepayments for Non-Current Assets","11251","asset_non_current","","False","Ettemaksed põhivara eest"
"l10n_ee_11300","Investment Properties","11300","asset_fixed","","False","Investeerimiskinnisvara"
"l10n_ee_11301","Depreciations on Investment Properties","11301","asset_fixed","","False","Kinnisvarainvesteeringute amortisatsioon"
"l10n_ee_1140","Land","1140","asset_fixed","","False","Maa"
"l10n_ee_11410","Buildings and Structures","11410","asset_fixed","","False","Ehitised ja struktuurid"
"l10n_ee_11411","Depreciations on Buildings and Structures","11411","asset_fixed","","False","Hoonete ja rajatiste amortisatsioon"
"l10n_ee_11420","Vehicles","11420","asset_fixed","","False","Sõidukid"
"l10n_ee_11421","Depreciations on Vehicles","11421","asset_fixed","","False","Sõidukite amortisatsioon"
"l10n_ee_11430","Computers and Computer Systems","11430","asset_fixed","","False","Arvutid ja arvutisüsteemid"
"l10n_ee_11431","Depreciations on Computers and Computer Systems","11431","asset_fixed","","False","Arvutite ja arvutisüsteemide amortisatsioon"
"l10n_ee_11440","Other Machinery and Equipment","11440","asset_fixed","","False","Muud masinad ja seadmed"
"l10n_ee_11441","Depreciations on Other Machinery and Equipment","11441","asset_fixed","","False","Muude masinate ja seadmete amortisatsioon"
"l10n_ee_11450","Other Tangible Non-Current Assets","11450","asset_fixed","","False","Muud materiaalsed põhivarad"
"l10n_ee_11451","Depreciations on Other Tangible Non-Current Assets","11451","asset_fixed","","False","Muude materiaalsete põhivarade amortisatsioonikulud"
"l10n_ee_115","Biological Assets","115","asset_fixed","","False","Bioloogilised varad"
"l10n_ee_11600","Goodwill","11600","asset_non_current","","False","Firmaväärtus"
"l10n_ee_11601","Amortizations on Goodwill","11601","asset_non_current","","False","Firmaväärtuse amortisatsioon"
"l10n_ee_11610","Development Costs","11610","asset_non_current","","False","Arenduskulud"
"l10n_ee_11611","Amortizations on Development Costs","11611","asset_non_current","","False","Arengukulude amortisatsioon"
"l10n_ee_11620","Computer Software","11620","asset_non_current","","False","Arvutitarkvara"
"l10n_ee_11621","Amortizations on Computer Software","11621","asset_non_current","","False","Arvutitarkvara amortisatsioon"
"l10n_ee_11630","Patents, Licenses and Trademarks","11630","asset_non_current","","False","Patendid, litsentsid ja kaubamärgid"
"l10n_ee_11631","Amortizations on Patents, Licenses and Trademarks","11631","asset_non_current","","False","Patendite, litsentside ja kaubamärkide amortisatsioonid"
"l10n_ee_11640","Other Intangible Assets","11640","asset_non_current","","False","Muud immateriaalsed varad"
"l10n_ee_11641","Amortizations on Other Intangible Assets","11641","asset_non_current","","False","Muude immateriaalsete varade amortisatsioonid"
"l10n_ee_2000","Short-Term Bank Loans","2000","liability_current","","False","Lühiajalised pangalaenud"
"l10n_ee_2001","Current Bank Overdrafts","2001","liability_current","","False","Praegused panga arvelduskontod"
"l10n_ee_2002","Short-Term Loans from Owners","2002","liability_current","","False","Lühiajalised laenud omanikelt"
"l10n_ee_2003","Short-Term Loans from Other Parties","2003","liability_current","","False","Lühiajalised laenud teistelt osapooltelt"
"l10n_ee_2004","Current Portion of Long-Term Loan","2004","liability_current","","False","Pikaajalise laenu praegune osa"
"l10n_ee_2010","Accounts Payable","2010","liability_payable","","True","Võlad tarnijatele"
"l10n_ee_20110","Salaries and Wages Payable","20110","liability_current","","False","Makstavad palgad ja töötasud"
"l10n_ee_20111","Withholdings from Salary","20111","liability_current","","False","Palgast kinnipeetavad summad"
"l10n_ee_20112","Vacation Pay Reserve","20112","liability_current","","False","Puhkusetasu reserv"
"l10n_ee_20113","Other Employee Payables","20113","liability_current","","False","Muud töötajate võlad"
"l10n_ee_201200","VAT Current Account","201200","liability_current","","False","Käibemaksu arvelduskonto"
"l10n_ee_201201","VAT Receivable (Input VAT)","201201","asset_current","","False","Saadaolev käibemaks (sisendkäibemaks)"
"l10n_ee_201202","VAT on the Acquisition of Non-Current Assets","201202","liability_current","","False","Käibemaksu põhivara soetamisel"
"l10n_ee_201203","VAT on the Import at Customs","201203","liability_current","","False","Impordi käibemaksu tasumine tollis"
"l10n_ee_201204","VAT Payable (Output VAT)","201204","liability_current","","False","Maksmisele kuuluv käibemaks (väljundkäibemaks)"
"l10n_ee_20121","Income Tax Payable (Company)","20121","liability_current","","False","Maksmisele kuuluv tulumaks (ettevõte)"
"l10n_ee_20122","Income Tax Payable (Personal)","20122","liability_current","","False","Maksmisele kuuluv tulumaks (füüsiline)"
"l10n_ee_20123","Income Tax Payable (Fringe Benefits)","20123","liability_current","","False","Maksmisele kuuluv tulumaks (lisahüvitised)"
"l10n_ee_20124","Social Tax Payable","20124","liability_current","","False","Maksmisele kuuluv sotsiaalmaks"
"l10n_ee_20125","Unemployment Insurance Premium Payable","20125","liability_current","","False","Tasumisele kuuluv töötuskindlustusmakse"
"l10n_ee_20126","Pension Insurance Payable","20126","liability_current","","False","Maksmisele kuuluv pensionikindlustus"
"l10n_ee_20127","Land Tax Payable","20127","liability_current","","False","Maksmisele kuuluv maamaks"
"l10n_ee_20128","Excise Tax Payable","20128","liability_current","","False","Maksmisele kuuluv aktsiisimaks"
"l10n_ee_20129","Other Taxes Payable","20129","liability_current","","False","Muud tasumisele kuuluvad maksud"
"l10n_ee_20130","Payables to Related Parties","20130","liability_current","","False","Võlad seotud osapooltele"
"l10n_ee_20131","Dividends Payable","20131","liability_current","","False","Maksmisele kuuluvad dividendid"
"l10n_ee_20132","Interests Payable","20132","liability_current","","False","Tasumisele kuuluvad intressid"
"l10n_ee_20133","Other Short-Term Payables","20133","liability_current","","False","Muud lühiajalised võlgnevused"
"l10n_ee_20140","Prepayments from Customers","20140","liability_current","","False","Klientide ettemaksed"
"l10n_ee_2015","Other Received Prepayments","2015","liability_current","","False","Muud saadud ettemaksed"
"l10n_ee_2020","Warranty Provisions","2020","liability_current","","False","Garantiieraldis"
"l10n_ee_2021","Tax Provisions","2021","liability_current","","False","Maksueraldis"
"l10n_ee_2022","Other Provisions","2022","liability_current","","False","Muud eraldised"
"l10n_ee_203","Government Grants","203","liability_current","","False","Sihtfinantseerimine"
"l10n_ee_2100","Long-Term Bank Loans","2100","liability_non_current","","False","Pikaajalised pangalaenud"
"l10n_ee_2101","Long-Term Loans from Owners","2101","liability_non_current","","False","Omanike pikaajalised laenud"
"l10n_ee_2102","Long-Term Portion of Financial Lease","2102","liability_non_current","","False","Kapitalirendi pikaajaline osa"
"l10n_ee_2110","Long-Term Accounts Payable","2110","liability_non_current","","False","Võlad tarnijatele"
"l10n_ee_2111","Long-Term Employee Payables","2111","liability_non_current","","False","Võlad töövõtjatele"
"l10n_ee_2112","Long-Term Taxes Payable","2112","liability_non_current","","False","Maksuvõlad"
"l10n_ee_2113","Other Long-Term Payables","2113","liability_non_current","","False","Muud võlad"
"l10n_ee_2114","Long-Term Deferred Revenue","2114","liability_non_current","","False","Tulevaste perioodide tulud"
"l10n_ee_2115","Other Received Long-Term Prepayments","2115","liability_non_current","","False","Muud saadud ettemaksed"
"l10n_ee_2120","Long-Term Warranty Provisions","2120","liability_non_current","","False","Garantiieraldis"
"l10n_ee_2121","Long-Term Tax Provisions","2121","liability_non_current","","False","Maksueraldis"
"l10n_ee_2122","Other Long-Term Provisions","2122","liability_non_current","","False","Muud eraldised"
"l10n_ee_213","Long-Term Government Grants","213","liability_non_current","","False","Sihtfinantseerimine"
"l10n_ee_300","Share Capital (Nominal Value)","300","equity","","False","Aktsiakapital või osakapital nimiväärtuses"
"l10n_ee_301","Unregistered Share Capital or Equity","301","equity","","False","Registreerimata aktsiakapital või osakapital"
"l10n_ee_302","Unpaid Share Capital","302","equity","","False","Sissemaksmata osakapital"
"l10n_ee_303","Share Premium","303","equity","","False","Ülekurss"
"l10n_ee_304","Own Shares","304","equity","","False","Oma aktsiad või osad"
"l10n_ee_310","Statutory Reserve Capital","310","equity","","False","Kohustuslik reservkapital"
"l10n_ee_311","Other Reserves","311","equity","","False","Muud reservid"
"l10n_ee_32","Other Equity","32","equity","","False","Muu omakapital"
"l10n_ee_330","Retained Profit/Loss From Previous Periods","330","equity","","False","Eelmiste perioodide jaotamata kasum (kahjum)"
"l10n_ee_331","Profit/Loss for the Financial Year","331","equity","","False","Aruandeaasta kasum (kahjum)"
"l10n_ee_40000","Sales of Goods in Estonia","40000","income","","False","Kaupade müük Eestis"
"l10n_ee_40001","Sales of Goods from Biological Assets in Estonia","40001","income","","False","Kaupade müük bioloogilistest varadest Eestis"
"l10n_ee_4001","Sales of Services in Estonia","4001","income","","False","Teenuste müük Eestis"
"l10n_ee_40100","Sales of Goods in the EU","40100","income","","False","Kaupade müük ELis"
"l10n_ee_40101","Sales of Goods from Biological Assets in the EU","40101","income","","False","Kaupade müük bioloogilistest varadest ELis"
"l10n_ee_4011","Sales of Services in the EU","4011","income","","False","Teenuste müük ELis"
"l10n_ee_40200","Export of Goods","40200","income","","False","Kaupade eksport"
"l10n_ee_40201","Export of Goods from Biological Assets","40201","income","","False","Kaupade eksport bioloogilistest varadest"
"l10n_ee_4021","Export of Services","4021","income","","False","Teenuste eksport"
"l10n_ee_41","Sales of Assets","41","income","","False","Varade müük"
"l10n_ee_420","Cash Rounding Gains","420","income_other","","False","Sularaha ümardamise kasumid"
"l10n_ee_421","Payment Difference Gains","421","income_other","","False","Maksete erinevus kasum"
"l10n_ee_422","Currency Exchange Rate Gains","422","income_other","","False","Valuutakursi kasumid"
"l10n_ee_423","Interest Received","423","income_other","","False","Saadud intressid"
"l10n_ee_424","Financial Income from Shares in Subsidiaries","424","income_other","","False","Finantstulu tütarettevõtete aktsiatest"
"l10n_ee_425","Financial Income from Shares in Associates","425","income_other","","False","Finantstulu sidusettevõtete aktsiatest"
"l10n_ee_426","Financial Income from Other Financial Investments","426","income_other","","False","Finantstulu muudest finantsinvesteeringutest"
"l10n_ee_427","Other Financial Income","427","income_other","","False","Muud finantstulud"
"l10n_ee_430","Cash Discount Gains","430","income_other","","False","Sularaha diskontokasumid"
"l10n_ee_431","Other Income","431","income_other","","False","Muud tulud"
"l10n_ee_50","Purchase of Goods for Resale","50","expense_direct_cost","","False","Kaupade ostmine edasimüügiks"
"l10n_ee_51","Purchase of Raw and Other Materials","51","expense_direct_cost","","False","Tooraine ja muude materjalide ostmine"
"l10n_ee_52","Purchase of Services / Subcontracting","52","expense_direct_cost","","False","Teenuste ostmine / alltöövõtt"
"l10n_ee_53","Transportation Costs for Goods, Raw Materials and Services","53","expense_direct_cost","","False","Kaupade, tooraine ja teenuste transpordikulud"
"l10n_ee_54","Customs Fees for Goods, Raw Materials and Services","54","expense_direct_cost","","False","Kaupade, tooraine ja teenuste tollimaksud"
"l10n_ee_600","Buildings Rental","600","expense","","False","Hoonete rentimine"
"l10n_ee_6010","Office Rental","6010","expense","","False","Kontori rentimine"
"l10n_ee_6011","Office Utilities","6011","expense","","False","Kontori kommunaalteenused"
"l10n_ee_6012","Office Security Costs","6012","expense","","False","Kontori turvakulud"
"l10n_ee_6013","Office Maintenance and Repairs","6013","expense","","False","Kontori hooldus ja remont"
"l10n_ee_6020","Workshop Rental","6020","expense","","False","Töökoja rentimine"
"l10n_ee_6021","Workshop Utilities","6021","expense","","False","Töökoja kommunaalteenused"
"l10n_ee_6022","Workshop Security Costs","6022","expense","","False","Töökoja turvalisuse kulud"
"l10n_ee_6023","Workshop Maintance and Repairs","6023","expense","","False","Töökoja hooldus ja remont"
"l10n_ee_603","Property Insurance","603","expense","","False","Varakindlustus"
"l10n_ee_604","Electricity / Gas","604","expense","","False","Elekter / gaas"
"l10n_ee_605","Water","605","expense","","False","Vesi"
"l10n_ee_606","Internet","606","expense","","False","Internet"
"l10n_ee_607","Phone Costs","607","expense","","False","Telefonikulud"
"l10n_ee_610","Equipment Rental","610","expense","","False","Seadmete rentimine"
"l10n_ee_611","Machinery and Equipment Maintenance and Repairs","611","expense","","False","Masinate ja seadmete hooldus ja remont"
"l10n_ee_612","Fuels, Oils and Lubricants","612","expense","","False","Kütused, õlid ja määrdeained"
"l10n_ee_620","Office Supplies and Postal Expenses","620","expense","","False","Kontoritarbed ja postikulud"
"l10n_ee_621","Informational and Educational Materials","621","expense","","False","Informatsiooni- ja õppematerjalid"
"l10n_ee_622","Small Tools","622","expense","","False","Väikesed tööriistad"
"l10n_ee_630","IT Services","630","expense","","False","IT-teenused"
"l10n_ee_631","Software","631","expense","","False","Tarkvara"
"l10n_ee_632","Consultations and Trainings","632","expense","","False","Konsultatsioonid ja koolitused"
"l10n_ee_633","Accounting Services","633","expense","","False","Raamatupidamisteenused"
"l10n_ee_634","Legal Services","634","expense","","False","Õigusteenused"
"l10n_ee_635","Auditing Services","635","expense","","False","Auditeerimisteenused"
"l10n_ee_636","Costs of Entertaining Guests","636","expense","","False","Külaliste majutamisega seotud kulud"
"l10n_ee_637","Marketing Expenses","637","expense","","False","Turunduskulud"
"l10n_ee_640","Car Rental","640","expense","","False","Autorent"
"l10n_ee_641","Car Insurance","641","expense","","False","Autokindlustus"
"l10n_ee_642","Car Fuel","642","expense","","False","Autokütus"
"l10n_ee_643","Car Maintenance and Repairs","643","expense","","False","Autode hooldus ja remont"
"l10n_ee_644","Other Car Expenses","644","expense","","False","Muud autokulud"
"l10n_ee_650","Compensation for the Use of a Personal Car","650","expense","","False","Isikliku auto kasutamise eest makstav hüvitis"
"l10n_ee_651","Fringe Benefits to Employees","651","expense","","False","Töötajatele makstavad lisahüvitised"
"l10n_ee_652","Salaries and Wages","652","expense","","False","Palgad ja töötasu"
"l10n_ee_653","Social Security Costs","653","expense","","False","Sotsiaalkindlustuskulud"
"l10n_ee_654","Unemployment Insurance Premium","654","expense","","False","Töötuskindlustusmakse"
"l10n_ee_655","Pension Expenses","655","expense","","False","Pensionikulud"
"l10n_ee_656","Vacation Pay Reserve","656","expense","","False","Puhkusetasu reserv"
"l10n_ee_6570","Income Tax on Fringe Benefits","6570","expense","","False","Lisahüvitiste tulumaks"
"l10n_ee_6571","Social Tax on Fringe Benefits","6571","expense","","False","Sotsiaalmaks lisahüvitistele"
"l10n_ee_6572","Social Tax","6572","expense","","False","Sotsiaalmaks"
"l10n_ee_660","State Fees","660","expense","","False","Riigilõivud"
"l10n_ee_661","Land Tax","661","expense","","False","Maamaks"
"l10n_ee_662","Income Tax","662","expense","","False","Tulumaks"
"l10n_ee_663","Fines and Fines for Delay","663","expense","","False","Trahvid ja trahvid hilinemise eest"
"l10n_ee_670","Bank Fees","670","expense","","False","Pangatasud"
"l10n_ee_671","Cash Rounding Losses","671","expense","","False","Sularaha ümardamise kahjum"
"l10n_ee_672","Payment Difference Losses","672","expense","","False","Maksete erinevus kahjud"
"l10n_ee_673","Currency Exchange Losses","673","expense","","False","Valuutakursikahjum"
"l10n_ee_674","Interest Expenses","674","expense","","False","Intressikulud"
"l10n_ee_675","Financial Expenses from Shares in Subsidiaries","675","expense","","False","Tütarettevõtete aktsiatest tulenevad finantskulud"
"l10n_ee_676","Financial Expenses from Shares in Associates","676","expense","","False","Finantskulud sidusettevõtete aktsiatest"
"l10n_ee_677","Financial Expenses from Other Financial Investments","677","expense","","False","Finantskulud muudest finantsinvesteeringutest"
"l10n_ee_678","Other Financial Expenses","678","expense","","False","Muud finantskulud"
"l10n_ee_680","Capitalized Expenses in the Manufacturing of Fixed Assets for Own Use","680","expense","","False","Kapitaliseeritud väljaminekud oma tarbeks põhivarade valmistamisel"
"l10n_ee_6810","Changes in Inventories of Finished Goods and Work in Progress","6810","expense","","False","Valmis ja lõpetamata toodangu varude jääkide muutus"
"l10n_ee_6811","Changes in Inventories of Agricultural Production","6811","expense","","False","Põllumajandusliku toodangu varude jääkide muutus"
"l10n_ee_682","Irrecoverable Receivables","682","expense","","False","Sissenõudmata nõuded"
"l10n_ee_6830","Depreciations on Non-Current Assets","6830","expense_depreciation","","False","Põhivara amortisatsioon"
"l10n_ee_6831","Amortizations on Intangible Assets","6831","expense_depreciation","","False","Immateriaalsete varade amortisatsioonid"
"l10n_ee_6832","Loss on Sales of Biological Assets","6832","expense","","False","Kahju bioloogiliste varade müügist"
"l10n_ee_6833","Loss on Sales of Non-Current Assets","6833","expense","","False","Kahju põhivara müügist"
"l10n_ee_6834","Significant Impairment of Current Assets","6834","expense","","False","Olulised käibevara allahindlused"
"l10n_ee_684","Gifts and Donations","684","expense","","False","Kingitused ja annetused"
"l10n_ee_6850","Cash Discount Losses","6850","expense","","False","Sularaha diskontokahjumid"
"l10n_ee_6851","Other Operating Expenses","6851","expense","","False","Muud ärikulud"
"l10n_ee_70","Clearing Account","70","off_balance","","False","Arvelduskonto"

```

## File: data\template\account.fiscal.position-ee.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@et"
"afpt_national","1","National","1","1","base.ee","","","","","","Riiklik"
"afpt_eu_private","2","EU Private","1","","","base.europe","","","","","ELi eraisik"
"afpt_eu_ic","3","EU Intra-Community","1","1","","base.europe","l10n_ee_vat_out_20_g","l10n_ee_vat_out_0_eu_g","","","ELi ühendusesisene"
"","","","","","","","l10n_ee_vat_out_22_g","l10n_ee_vat_out_0_eu_g","","",""
"","","","","","","","l10n_ee_vat_out_24_g","l10n_ee_vat_out_0_eu_g","","",""
"","","","","","","","l10n_ee_vat_out_20_s","l10n_ee_vat_out_0_eu_s","","",""
"","","","","","","","l10n_ee_vat_out_22_s","l10n_ee_vat_out_0_eu_s","","",""
"","","","","","","","l10n_ee_vat_out_24_s","l10n_ee_vat_out_0_eu_s","","",""
"","","","","","","","l10n_ee_vat_out_13_s","l10n_ee_vat_out_0_eu_s","","",""
"","","","","","","","l10n_ee_vat_out_9_g","l10n_ee_vat_out_0_eu_g","","",""
"","","","","","","","l10n_ee_vat_out_9_s","l10n_ee_vat_out_0_eu_s","","",""
"","","","","","","","l10n_ee_vat_out_5_g","l10n_ee_vat_out_0_eu_g","","",""
"","","","","","","","l10n_ee_vat_out_5_s","l10n_ee_vat_out_0_eu_s","","",""
"","","","","","","","l10n_ee_vat_out_0_g","l10n_ee_vat_out_0_eu_g","","",""
"","","","","","","","l10n_ee_vat_out_0_s","l10n_ee_vat_out_0_eu_s","","",""
"","","","","","","","l10n_ee_vat_in_20_g","l10n_ee_vat_in_0_eu_g_22","","",""
"","","","","","","","l10n_ee_vat_in_20_s","l10n_ee_vat_in_0_eu_s_22","","",""
"","","","","","","","l10n_ee_vat_in_22_s","l10n_ee_vat_in_0_eu_s_22","","",""
"","","","","","","","l10n_ee_vat_in_22_g","l10n_ee_vat_in_0_eu_g_22","","",""
"","","","","","","","l10n_ee_vat_in_24_g","l10n_ee_vat_in_0_eu_g_24","","",""
"","","","","","","","l10n_ee_vat_in_24_s","l10n_ee_vat_in_0_eu_s_24","","",""
"","","","","","","","l10n_ee_vat_in_13_s","l10n_ee_vat_in_0_eu_s_22","","",""
"","","","","","","","l10n_ee_vat_in_9_g","l10n_ee_vat_in_0_eu_g_22","","",""
"","","","","","","","l10n_ee_vat_in_9_s","l10n_ee_vat_in_0_eu_s_22","","",""
"","","","","","","","l10n_ee_vat_in_5_g","l10n_ee_vat_in_0_eu_g_22","","",""
"","","","","","","","l10n_ee_vat_in_5_s","l10n_ee_vat_in_0_eu_s_22","","",""
"","","","","","","","l10n_ee_vat_in_0_g","l10n_ee_vat_in_0_eu_g_22","","",""
"","","","","","","","l10n_ee_vat_in_0_s","l10n_ee_vat_in_0_eu_s_22","","",""
"","","","","","","","","","l10n_ee_40000","l10n_ee_40100",""
"","","","","","","","","","l10n_ee_40001","l10n_ee_40101",""
"","","","","","","","","","l10n_ee_4001","l10n_ee_4011",""
"afpt_imp_exp","4","Import/Export","1","","","","l10n_ee_vat_out_20_g","l10n_ee_vat_out_0_exp_g","","","Import/Eksport"
"","","","","","","","l10n_ee_vat_out_22_g","l10n_ee_vat_out_0_exp_g","","",""
"","","","","","","","l10n_ee_vat_out_24_g","l10n_ee_vat_out_0_exp_g","","",""
"","","","","","","","l10n_ee_vat_out_20_s","l10n_ee_vat_out_0_exp_s","","",""
"","","","","","","","l10n_ee_vat_out_22_s","l10n_ee_vat_out_0_exp_s","","",""
"","","","","","","","l10n_ee_vat_out_24_s","l10n_ee_vat_out_0_exp_s","","",""
"","","","","","","","l10n_ee_vat_out_13_s","l10n_ee_vat_out_0_exp_s","","",""
"","","","","","","","l10n_ee_vat_out_9_g","l10n_ee_vat_out_0_exp_g","","",""
"","","","","","","","l10n_ee_vat_out_9_s","l10n_ee_vat_out_0_exp_s","","",""
"","","","","","","","l10n_ee_vat_out_5_g","l10n_ee_vat_out_0_exp_g","","",""
"","","","","","","","l10n_ee_vat_out_5_s","l10n_ee_vat_out_0_exp_s","","",""
"","","","","","","","l10n_ee_vat_out_0_g","l10n_ee_vat_out_0_exp_g","","",""
"","","","","","","","l10n_ee_vat_out_0_s","l10n_ee_vat_out_0_exp_s","","",""
"","","","","","","","l10n_ee_vat_in_20_g","l10n_ee_vat_in_20_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_22_g","l10n_ee_vat_in_22_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_24_g","l10n_ee_vat_in_24_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_20_s","l10n_ee_vat_in_20_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_22_s","l10n_ee_vat_in_22_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_24_s","l10n_ee_vat_in_24_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_13_s","l10n_ee_vat_in_13_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_9_g","l10n_ee_vat_in_9_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_9_s","l10n_ee_vat_in_9_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_5_g","l10n_ee_vat_in_5_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_5_s","l10n_ee_vat_in_5_imp_kms_38","","",""
"","","","","","","","l10n_ee_vat_in_0_g","l10n_ee_vat_in_0_imp","","",""
"","","","","","","","l10n_ee_vat_in_0_s","l10n_ee_vat_in_0_imp","","",""
"","","","","","","","","","l10n_ee_40000","l10n_ee_40200",""
"","","","","","","","","","l10n_ee_40001","l10n_ee_40201",""
"","","","","","","","","","l10n_ee_4001","l10n_ee_4021",""

```

## File: data\template\account.group-ee.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@et"
"ee_group_1","1","","Assets","Varad"
"ee_group_10","10","","Current Assets","Käibevarad"
"ee_group_100","100","","Cash","Raha"
"ee_group_101","101","","Financial Investments","Finantsinvesteeringud"
"ee_group_102","102","","Receivables and Prepayments","Nõuded ja ettemaksed"
"ee_group_1020","1020","","Trade Receivables","Nõuded ostjate vastu"
"ee_group_1021","1021","","Receivables from Related Parties","Nõuded seotud osapoolte vastu"
"ee_group_1022","1022","","Prepaid and Deferred Taxes","Maksude ettemaksed ja tagasinõuded"
"ee_group_1023","1023","","Loan Receivables","Laenunõuded"
"ee_group_1024","1024","","Other Receivables","Muud nõuded"
"ee_group_1025","1025","","Prepayments","Ettemaksed"
"ee_group_103","103","","Inventory","Varud"
"ee_group_1030","1030","","Raw and Other Materials","Tooraine ja materjal"
"ee_group_1031","1031","","Work in Progress","Lõpetamata toodang"
"ee_group_1032","1032","","Finished Goods","Valmistoodang"
"ee_group_1033","1033","","Goods for Resale","Müügiks ostetud kaubad"
"ee_group_1034","1034","","Prepayments to Suppliers","Ettemaksed varude eest"
"ee_group_104","104","","Biological Assets","Bioloogilised varad"
"ee_group_11","11","","Non-Current Assets","Põhivarad"
"ee_group_110","110","","Investments in Subsidiaries and Affiliates","Investeeringud tütar- ja sidusettevõtjatesse"
"ee_group_1100","1100","","Shares and Participations in Subsidiaries","Tütarettevõtjate aktsiad ja osad"
"ee_group_1101","1101","","Shares and Participations in Affiliates","Sidusettevõtjate aktsiad ja osad"
"ee_group_111","111","","Financial Investments","Finantsinvesteeringud"
"ee_group_112","112","","Receivables and Prepayments","Nõuded ja ettemaksed"
"ee_group_1120","1120","","Long-Term Trade Receivables","Nõuded ostjate vastu"
"ee_group_1121","1121","","Long-Term Receivables from Related Parties","Nõuded seotud osapoolte vastu"
"ee_group_1122","1122","","Long-Term Prepaid and Deferred Taxes","Maksude ettemaksed ja tagasinõuded"
"ee_group_1123","1123","","Long-Term Loan Receivables","Laenunõuded"
"ee_group_1124","1124","","Other Long-Term Receivables","Muud nõuded"
"ee_group_1125","1125","","Long-Term Prepayments","Pikaajalised ettemaksed"
"ee_group_113","113","","Real Estate Investments","Kinnisvarainvesteeringud"
"ee_group_114","114","","Tangible Non-Current Assets","Materiaalsed põhivarad"
"ee_group_115","115","","Biological Assets","Bioloogilised varad"
"ee_group_116","116","","Intangible Non-Current Assets","Immateriaalsed põhivarad"
"ee_group_2","2","","Liabilities","Kohustised"
"ee_group_20","20","","Current Liabilities","Lühiajalised kohustised"
"ee_group_200","200","","Loan Liabilities","Laenukohustised"
"ee_group_201","201","","Payables and Prepayments","Võlad ja ettemaksed"
"ee_group_2010","2010","","Accounts Payable","Võlad tarnijatele"
"ee_group_2011","2011","","Employee Payables","Võlad töövõtjatele"
"ee_group_2012","2012","","Taxes Payable","Maksuvõlad"
"ee_group_2013","2013","","Other Payables","Muud võlad"
"ee_group_2014","2014","","Deferred Revenue","Tulevaste perioodide tulud"
"ee_group_2015","2015","","Other Received Prepayments","Muud saadud ettemaksed"
"ee_group_202","202","","Provisions","Eraldised"
"ee_group_2020","2020","","Warranty Provisions","Garantiieraldis"
"ee_group_2021","2021","","Tax Provisions","Maksueraldis"
"ee_group_2022","2022","","Other Provisions","Muud eraldised"
"ee_group_203","203","","Government Grants","Sihtfinantseerimine"
"ee_group_21","21","","Non-Current Liabilities","Pikaajalised kohustised"
"ee_group_210","210","","Long-Term Loan Liabilities","Laenukohustised"
"ee_group_211","211","","Long-Term Payables and Prepayments","Võlad ja ettemaksed"
"ee_group_2110","2110","","Long-Term Accounts Payable","Võlad tarnijatele"
"ee_group_2111","2111","","Long-Term Employee Payables","Võlad töövõtjatele"
"ee_group_2112","2112","","Long-Term Taxes Payable","Maksuvõlad"
"ee_group_2113","2113","","Other Long-Term Payables","Muud võlad"
"ee_group_2114","2114","","Long-Term Deferred Revenue","Tulevaste perioodide tulud"
"ee_group_2115","2115","","Other Received Long-Term Prepayments","Muud saadud ettemaksed"
"ee_group_212","212","","Long-Term Provisions","Eraldised"
"ee_group_2120","2120","","Long-Term Warranty Provisions","Garantiieraldis"
"ee_group_2121","2121","","Long-Term Tax Provisions","Maksueraldis"
"ee_group_2122","2122","","Other Long-Term Provisions","Muud eraldised"
"ee_group_213","213","","Long-Term Government Grants","Sihtfinantseerimine"
"ee_group_3","3","","Equity","Omakapital"
"ee_group_30","30","","Share Capital (Nominal Value)","Aktsiakapital või osakapital nimiväärtuses"
"ee_group_31","31","","Unregistered Share Capital or Equity","Registreerimata aktsiakapital või osakapital"
"ee_group_32","32","","Unpaid Share Capital","Sissemaksmata osakapital"
"ee_group_33","33","","Share Premium","Ülekurss"
"ee_group_34","34","","Own Shares","Oma aktsiad või osad"
"ee_group_35","35","","Statutory Reserve Capital","Kohustuslik reservkapital"
"ee_group_36","36","","Other Reserves","Muud reservid"
"ee_group_37","37","","Other Equity","Muu omakapital"
"ee_group_38","38","","Retained Profit/Loss From Previous Periods","Eelmiste perioodide jaotamata kasum (kahjum)"
"ee_group_39","39","","Profit/Loss for the Financial Year","Aruandeaasta kasum (kahjum)"
"ee_group_4","4","","Income","Tulud"
"ee_group_40","40","","Sales of Goods and Services","Kaupade ja teenuste müük"
"ee_group_400","400","","Sales of Goods and Services in Estonia","Kaupade ja teenuste müük Eestis"
"ee_group_401","401","","Sales of Goods and Services in the EU","Kaupade ja teenuste müük ELis"
"ee_group_402","402","","Export of Goods and Services","Kaupade ja teenuste eksport"
"ee_group_41","41","","Sales of Assets","Varade müük"
"ee_group_42","42","","Financial Income","Finantstulud"
"ee_group_43","43","","Other Income","Muud tulud"
"ee_group_5","5","","Cost of Goods Sold","Müüdud kaupade maksumus"
"ee_group_6","6","","Other Expenses","Muud kulud"
"ee_group_60","60","","Property Expenses","Kinnisvarakulud"
"ee_group_600","600","","Buildings Rental","Hoonete rentimine"
"ee_group_601","601","","Office Expenses","Kontorikulud"
"ee_group_602","602","","Workshop Expenses","Töötoa kulud"
"ee_group_603","603","","Property Insurance","Varakindlustus"
"ee_group_604","604","","Electricity / Gas","Elekter / gaas"
"ee_group_605","605","","Water","Vesi"
"ee_group_606","606","","Internet","Internet"
"ee_group_607","607","","Phone Costs","Telefonikulud"
"ee_group_61","61","","Equipment Expenses","Seadmete kulud"
"ee_group_62","62","","Product Expenses","Toote kulud"
"ee_group_63","63","","Service Expenses","Teenuse kulud"
"ee_group_64","64","","Car Expenses","Autokulud"
"ee_group_65","65","","Employee Expenses","Töötajate kulud"
"ee_group_657","657","","Employee Expenses - Taxes","Töötajate kulud - Maksud"
"ee_group_66","66","","Fees and Taxes","Tasud ja maksud"
"ee_group_67","67","","Financial Expenses","Financial Expenses"
"ee_group_68","68","","Other Operating Expenses","Muud ärikulud"
"ee_group_70","70","","Other Accounts","Muud arved"

```

## File: data\template\account.tax-ee.csv

```csv
"id","sequence","name","type_tax_use","amount","amount_type","description","tax_group_id","active","l10n_ee_kmd_inf_code","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@et"
"l10n_ee_vat_out_20_g","10","20% G","sale","20.0","percent","20%","tax_group_vat_20","False","","base","invoice","+1_1_base","","","20% K"
"","","","","","","","","","","tax","invoice","+1_1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-1_1_base","","",""
"","","","","","","","","","","tax","refund","-1_1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_22_g","11","22% G","sale","22.0","percent","22%","tax_group_vat_22","","","base","invoice","+1_base","","","22% K"
"","","","","","","","","","","tax","invoice","+1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-1_base","","",""
"","","","","","","","","","","tax","refund","-1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_24_g","11","24% G","sale","24.0","percent","24%","tax_group_vat_24","","","base","invoice","+1_base","","","24% K"
"","","","","","","","","","","tax","invoice","+1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-1_base","","",""
"","","","","","","","","","","tax","refund","-1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_20_s","20","20% S","sale","20.0","percent","20%","tax_group_vat_20","False","","base","invoice","+1_1_base","","","20% T"
"","","","","","","","","","","tax","invoice","+1_1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-1_1_base","","",""
"","","","","","","","","","","tax","refund","-1_1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_22_s","21","22% S","sale","22.0","percent","22%","tax_group_vat_22","","","base","invoice","+1_base","","","22% T"
"","","","","","","","","","","tax","invoice","+1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-1_base","","",""
"","","","","","","","","","","tax","refund","-1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_24_s","21","24% S","sale","24.0","percent","24%","tax_group_vat_24","","","base","invoice","+1_base","","","24% T"
"","","","","","","","","","","tax","invoice","+1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-1_base","","",""
"","","","","","","","","","","tax","refund","-1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_13_s","25","13% S","sale","13.0","percent","13%","tax_group_vat_13","","","base","invoice","+2_2_base","","","13% T"
"","","","","","","","","","","tax","invoice","+2_2_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-2_2_base","","",""
"","","","","","","","","","","tax","refund","-2_2_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_9_g","30","9% G","sale","9.0","percent","9%","tax_group_vat_9","","","base","invoice","+2_base","","","9% K"
"","","","","","","","","","","tax","invoice","+2_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-2_base","","",""
"","","","","","","","","","","tax","refund","-2_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_9_s","40","9% S","sale","9.0","percent","9%","tax_group_vat_9","False","","base","invoice","+2_base","","","9% T"
"","","","","","","","","","","tax","invoice","+2_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-2_base","","",""
"","","","","","","","","","","tax","refund","-2_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_5_g","50","5% G","sale","5.0","percent","5%","tax_group_vat_5","","","base","invoice","+2_1_base","","","5% K"
"","","","","","","","","","","tax","invoice","+2_1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-2_1_base","","",""
"","","","","","","","","","","tax","refund","-2_1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_5_s","60","5% S","sale","5.0","percent","5%","tax_group_vat_5","False","","base","invoice","+2_1_base","","","5% T"
"","","","","","","","","","","tax","invoice","+2_1_tax","l10n_ee_201204","",""
"","","","","","","","","","","base","refund","-2_1_base","","",""
"","","","","","","","","","","tax","refund","-2_1_tax","l10n_ee_201204","",""
"l10n_ee_vat_out_0_g","70","0% G","sale","0.0","percent","0%","tax_group_vat_0","","","base","invoice","+3","","","0% K"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_s","80","0% S","sale","0.0","percent","0%","tax_group_vat_0","False","","base","invoice","+3","","","0% T"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_eu_g","90","0% EU G","sale","0.0","percent","0% EU","tax_group_vat_0","","","base","invoice","+3_1_1||+3_1_1_G","","","0% EL K"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3_1_1||-3_1_1_G","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_eu_g_t","100","0% EU G Trian","sale","0.0","percent","0% EU T","tax_group_vat_0","False","","base","invoice","+3_1_1_T","","","0% EL K Kolmn."
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3_1_1_T","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_eu_s","110","0% EU S","sale","0.0","percent","0% EU","tax_group_vat_0","","","base","invoice","+3_1||+3_1_S","","","0% EL T"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3_1||-3_1_S","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_exp_g","120","0% EX G","sale","0.0","percent","0%","tax_group_vat_0","False","","base","invoice","+3_2","","","0% EX K"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3_2","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_pas","130","0% Passengers","sale","0.0","percent","0%","tax_group_vat_0","False","","base","invoice","+3_2_1","","","0% Reisijad"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3_2_1","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_exp_s","140","0% EX S","sale","0.0","percent","0%","tax_group_vat_0","False","","base","invoice","+3","","","0% EX T"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-3","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_exempt","150","0% Exempt","sale","0.0","percent","Exempt","tax_group_vat_0","False","","base","invoice","+8","","","0% Maksuvaba"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-8","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_out_0_kms_41_1","160","20% KMS §41¹","sale","20.0","percent","20% Special","tax_group_vat_20","False","2","base","invoice","+9","","","20% KMS §41¹"
"","","","","","","","","","","tax","invoice","","l10n_ee_201204","100",""
"","","","","","","","","","","tax","invoice","","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-9","","",""
"","","","","","","","","","","tax","refund","","l10n_ee_201204","100",""
"","","","","","","","","","","tax","refund","","l10n_ee_201204","-100",""
"l10n_ee_vat_out_0_kms_41_2","161","22% KMS §41¹","sale","22.0","percent","22% Special","tax_group_vat_22","False","2","base","invoice","+9","","","22% KMS §41¹"
"","","","","","","","","","","tax","invoice","","l10n_ee_201204","100",""
"","","","","","","","","","","tax","invoice","","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-9","","",""
"","","","","","","","","","","tax","refund","","l10n_ee_201204","100",""
"","","","","","","","","","","tax","refund","","l10n_ee_201204","-100",""
"l10n_ee_vat_out_0_kms_41_3","161","24% KMS §41¹","sale","24.0","percent","24% Special","tax_group_vat_24","False","2","base","invoice","+9","","","24% KMS §41¹"
"","","","","","","","","","","tax","invoice","","l10n_ee_201204","100",""
"","","","","","","","","","","tax","invoice","","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-9","","",""
"","","","","","","","","","","tax","refund","","l10n_ee_201204","100",""
"","","","","","","","","","","tax","refund","","l10n_ee_201204","-100",""
"l10n_ee_vat_in_20_g","170","20% G","purchase","20.0","percent","20%","tax_group_vat_20","False","","base","invoice","","","","20% K"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_22_g","171","22% G","purchase","22.0","percent","22%","tax_group_vat_22","","","base","invoice","","","","22% K"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_24_g","171","24% G","purchase","24.0","percent","24%","tax_group_vat_24","","","base","invoice","","","","24% K"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_20_s","180","20% S","purchase","20.0","percent","20%","tax_group_vat_20","False","","base","invoice","","","","20% T"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_22_s","181","22% S","purchase","22.0","percent","22%","tax_group_vat_22","","","base","invoice","","","","22% T"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_24_s","181","24% S","purchase","24.0","percent","24%","tax_group_vat_24","","","base","invoice","","","","24% T"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_13_s","185","13% S","purchase","13.0","percent","13%","tax_group_vat_13","","","base","invoice","","","","13% T"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_9_g","190","9% G","purchase","9.0","percent","9%","tax_group_vat_9","","","base","invoice","","","","9% K"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_9_s","200","9% S","purchase","9.0","percent","9%","tax_group_vat_9","False","","base","invoice","","","","9% T"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_5_g","210","5% G","purchase","5.0","percent","5%","tax_group_vat_5","","","base","invoice","","","","5% K"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_5_s","220","5% S","purchase","5.0","percent","5%","tax_group_vat_5","False","","base","invoice","","","","5% T"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_0_g","230","0% G","purchase","0.0","percent","0%","tax_group_vat_0","","","base","invoice","","","","0% K"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_in_0_s","240","0% S","purchase","0.0","percent","0%","tax_group_vat_0","False","","base","invoice","","","","0% T"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_in_0_eu_g","250","0% EU G 20%","purchase","20.0","percent","0% EU","tax_group_vat_0","False","","base","invoice","+1_1_base||+6_1","","","0% EL K 20%"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_1_base||-6_1","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_eu_g_22","251","0% EU G 22%","purchase","22.0","percent","0% EU","tax_group_vat_0","","","base","invoice","+1_base||+6_1","","","0% EL K 22%"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_base||-6_1","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_eu_g_24","251","0% EU G 24%","purchase","24.0","percent","0% EU","tax_group_vat_0","","","base","invoice","+1_base||+6_1","","","0% EL K 24%"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_base||-6_1","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_eu_s","260","0% EU S 20%","purchase","20.0","percent","0% EU","tax_group_vat_0","False","","base","invoice","+1_1_base||+6","","","0% EL T 20%"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_1_base||-6","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_eu_s_22","261","0% EU S 22%","purchase","22.0","percent","0% EU","tax_group_vat_0","","","base","invoice","+1_base||+6","","","0% EL T 22%"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_base||-6","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_eu_s_24","261","0% EU S 24%","purchase","24.0","percent","0% EU","tax_group_vat_0","","","base","invoice","+1_base||+6","","","0% EL T 24%"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_base||-6","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_20_car","270","20% Car","purchase","20.0","percent","20%","tax_group_vat_20","False","","base","invoice","","","","20% Auto"
"","","","","","","","","","","tax","invoice","+5_3_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_3_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_20_car_part","280","20% Car 50%","purchase","20.0","percent","20%","tax_group_vat_20","False","11","base","invoice","","","","20% Auto 50%"
"","","","","","","","","","","tax","invoice","+5_4_tax","l10n_ee_201201","50",""
"","","","","","","","","","","tax","invoice","","","50",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_4_tax","l10n_ee_201201","50",""
"","","","","","","","","","","tax","refund","","","50",""
"l10n_ee_vat_in_22_car","271","22% Car","purchase","22.0","percent","22%","tax_group_vat_22","","","base","invoice","","","","22% Auto"
"","","","","","","","","","","tax","invoice","+5_3_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_3_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_24_car","271","24% Car","purchase","24.0","percent","24%","tax_group_vat_24","","","base","invoice","","","","24% Auto"
"","","","","","","","","","","tax","invoice","+5_3_tax","l10n_ee_201201","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_3_tax","l10n_ee_201201","",""
"l10n_ee_vat_in_22_car_part","281","22% Car 50%","purchase","22.0","percent","22%","tax_group_vat_22","False","11","base","invoice","","","","22% Auto 50%"
"","","","","","","","","","","tax","invoice","+5_4_tax","l10n_ee_201201","50",""
"","","","","","","","","","","tax","invoice","","","50",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_4_tax","l10n_ee_201201","50",""
"","","","","","","","","","","tax","refund","","","50",""
"l10n_ee_vat_in_24_car_part","281","24% Car 50%","purchase","24.0","percent","24%","tax_group_vat_24","False","11","base","invoice","","","","24% Auto 50%"
"","","","","","","","","","","tax","invoice","+5_4_tax","l10n_ee_201201","50",""
"","","","","","","","","","","tax","invoice","","","50",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_4_tax","l10n_ee_201201","50",""
"","","","","","","","","","","tax","refund","","","50",""
"l10n_ee_vat_in_20_assets","290","20% Fixed Assets","purchase","20.0","percent","20%","tax_group_vat_20","False","","base","invoice","","","","20% Põhivara"
"","","","","","","","","","","tax","invoice","+5_2_tax","l10n_ee_201202","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_2_tax","l10n_ee_201202","",""
"l10n_ee_vat_in_22_assets","291","22% Fixed Assets","purchase","22.0","percent","22%","tax_group_vat_22","False","","base","invoice","","","","22% Põhivara"
"","","","","","","","","","","tax","invoice","+5_2_tax","l10n_ee_201202","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_2_tax","l10n_ee_201202","",""
"l10n_ee_vat_in_24_assets","291","24% Fixed Assets","purchase","24.0","percent","24%","tax_group_vat_24","False","","base","invoice","","","","24% Põhivara"
"","","","","","","","","","","tax","invoice","+5_2_tax","l10n_ee_201202","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_2_tax","l10n_ee_201202","",""
"l10n_ee_vat_in_imp_cus","300","EX VAT Customs","purchase","0.0","percent","Import VAT","tax_group_vat_20","","","base","invoice","+5_1_tax","","","EX KM Toll"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-5_1_tax","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_in_20_imp_kms_38","310","20% EX KMS §38","purchase","20.0","percent","20%","tax_group_vat_20","False","","base","invoice","","","","20% EX KMS §38"
"","","","","","","","","","","tax","invoice","+5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-4_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+4_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_22_imp_kms_38","311","22% EX KMS §38","purchase","22.0","percent","22%","tax_group_vat_22","False","","base","invoice","","","","22% EX KMS §38"
"","","","","","","","","","","tax","invoice","+5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-4_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+4_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_24_imp_kms_38","311","24% EX KMS §38","purchase","24.0","percent","24%","tax_group_vat_24","False","","base","invoice","","","","24% EX KMS §38"
"","","","","","","","","","","tax","invoice","+5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-4_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+4_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_13_imp_kms_38","311","13% EX KMS §38","purchase","13.0","percent","13%","tax_group_vat_13","False","","base","invoice","","","","13% EX KMS §38"
"","","","","","","","","","","tax","invoice","+5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-4_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+4_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_9_imp_kms_38","330","9% EX KMS §38","purchase","9.0","percent","9%","tax_group_vat_9","False","","base","invoice","","","","9% EX KMS §38"
"","","","","","","","","","","tax","invoice","+5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-4_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+4_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_5_imp_kms_38","350","5% EX KMS §38","purchase","5.0","percent","5%","tax_group_vat_5","False","","base","invoice","","","","5% EX KMS §38"
"","","","","","","","","","","tax","invoice","+5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-4_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","-5_1_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+4_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_imp","360","0% EX","purchase","0.0","percent","0%","tax_group_vat_0","False","","base","invoice","","","","0% Eksport"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_ee_vat_in_0_kms_41_1","370","20% KMS §41¹","purchase","20.0","percent","20%","tax_group_vat_20","False","12","base","invoice","+1_1_base||+7_1","","","20% KMS §41¹"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_1_base||-7_1","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_kms_41_2","370","22% KMS §41¹","purchase","22.0","percent","22%","tax_group_vat_22","False","12","base","invoice","+1_base||+7_1","","","22% KMS §41¹"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_base||-7_1","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_tax","l10n_ee_201204","-100",""
"l10n_ee_vat_in_0_kms_41_3","370","24% KMS §41¹","purchase","24.0","percent","24%","tax_group_vat_24","False","12","base","invoice","+1_base||+7_1","","","24% KMS §41¹"
"","","","","","","","","","","tax","invoice","+5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","invoice","-1_tax","l10n_ee_201204","-100",""
"","","","","","","","","","","base","refund","-1_base||-7_1","","",""
"","","","","","","","","","","tax","refund","-5_tax","l10n_ee_201201","100",""
"","","","","","","","","","","tax","refund","+1_tax","l10n_ee_201204","-100",""

```

## File: data\template\account.tax.group-ee.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@et"
"tax_group_vat_20","VAT 20%","base.ee","l10n_ee_201200","l10n_ee_201200","KM 20%"
"tax_group_vat_22","VAT 22%","base.ee","l10n_ee_201200","l10n_ee_201200","KM 22%"
"tax_group_vat_24","VAT 24%","base.ee","l10n_ee_201200","l10n_ee_201200","KM 24%"
"tax_group_vat_13","VAT 13%","base.ee","l10n_ee_201200","l10n_ee_201200","KM 13%"
"tax_group_vat_9","VAT 9%","base.ee","l10n_ee_201200","l10n_ee_201200","KM 9%"
"tax_group_vat_5","VAT 5%","base.ee","l10n_ee_201200","l10n_ee_201200","KM 5%"
"tax_group_vat_0","VAT 0%","base.ee","l10n_ee_201200","l10n_ee_201200","KM 0%"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID

def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'ee')], order="parent_path"):
        env['account.chart.template'].try_loading('ee', company)

```

## File: migrations\1.2\end-migrate_update_taxes.py

```python
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'ee')], order="parent_path"):
        env['account.chart.template'].try_loading('ee', company)

```

## File: migrations\1.2\pre-migrate.py

```python
def migrate(cr, version):

    cr.execute("""
        UPDATE ir_model_data
           SET name = 'tax_report'
         WHERE module='l10n_ee' AND name='tax_report_vat'
    """)

```

## File: models\account_tax.py

```python
from odoo import fields, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_ee_kmd_inf_code = fields.Selection(
        selection=[
            ('1', 'Sale KMS §41/42'),
            ('2', 'Sale KMS §41^1'),
            ('11', 'Purchase KMS §29(4)/30/32'),
            ('12', 'Purchase KMS §41^1'),
        ],
        string='KMD INF Code',
        default=False,
        help='This field is used for the comments/special code column in the KMD INF report.'
    )

```

## File: models\template_ee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ee')
    def _get_ee_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_ee_10200',
            'property_account_payable_id': 'l10n_ee_2010',
            'property_account_income_categ_id': 'l10n_ee_40000',
            'property_account_expense_categ_id': 'l10n_ee_50',
            'code_digits': '6',
        }

    @template('ee', 'res.company')
    def _get_ee_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ee',
                'bank_account_code_prefix': '1001',
                'cash_account_code_prefix': '1000',
                'transfer_account_code_prefix': '1008',
                'account_default_pos_receivable_account_id': 'l10n_ee_10201',
                'income_currency_exchange_account_id': 'l10n_ee_422',
                'expense_currency_exchange_account_id': 'l10n_ee_673',
                'account_journal_suspense_account_id': 'l10n_ee_1009',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_ee_6850',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_ee_430',
                'default_cash_difference_income_account_id': 'l10n_ee_420',
                'default_cash_difference_expense_account_id': 'l10n_ee_671',
                'account_sale_tax_id': 'l10n_ee_vat_out_22_g',
                'account_purchase_tax_id': 'l10n_ee_vat_in_22_g',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ee
from . import account_tax

```

## File: views\account_tax_form.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_tax_form_inherit_l10n_ee" model="ir.ui.view">
            <field name="name">account.tax.form</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_form"/>
            <field name="arch" type="xml">
                <field name="country_id" position="after">
                    <field name="l10n_ee_kmd_inf_code" invisible="country_code != 'EE'"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

