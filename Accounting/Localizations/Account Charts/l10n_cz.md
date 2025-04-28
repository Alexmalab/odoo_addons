# Odoo Module: l10n_cz

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
    'name': 'Czech - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['cz'],
    'version': '1.1',
    'author': '26HOUSE (http://www.26house.com)',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Czech accounting chart and localization.  With Chart of Accounts with taxes and basic fiscal positions.

Tento modul definuje:

- Českou účetní osnovu za rok 2020

- Základní sazby pro DPH z prodeje a nákupu

- Základní fiskální pozice pro českou legislativu
    """,
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
        'data/tax_report.xml',
        'views/report_invoice.xml',
        'views/res_partner_views.xml',
        'views/res_company_views.xml',
        'views/account_move_views.xml',
        'views/report_template.xml',
    ],
    'demo': [
        'data/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\demo_company.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="partner_demo_company_cz" model="res.partner">
        <field name="name">CZ Company</field>
        <field name="vat">CZ12345679</field>
        <field name="street">Pařížská Street 25/31</field>
        <field name="city">Praha</field>
        <field name="country_id" ref="base.cz"/>
        <field name="state_id" ref="base.state_L"/>
        <field name="zip"/>
        <field name="phone">+420 5 12 34 56 78</field>
        <field name="email">info@company.czexample.com</field>
        <field name="website">www.czexample.com</field>
    </record>

    <record id="demo_company_cz" model="res.company">
        <field name="name">CZ Company</field>
        <field name="partner_id" ref="partner_demo_company_cz"/>
    </record>

    <record id="demo_bank_cz" model="res.partner.bank">
        <field name="acc_number">CZ6050517873128883346693</field>
        <field name="partner_id" ref="partner_demo_company_cz"/>
        <field name="company_id" ref="demo_company_cz"/>
    </record>

    <function model="res.company" name="_onchange_country_id">
        <value eval="[ref('demo_company_cz')]"/>
    </function>

    <function model="res.users" name="write">
        <value eval="[ref('base.user_root'), ref('base.user_admin'), ref('base.user_demo')]"/>
        <value eval="{'company_ids': [(4, ref('l10n_cz.demo_company_cz'))]}"/>
    </function>

    <function model="account.chart.template" name="try_loading">
        <value eval="[]"/>
        <value>cz</value>
        <value model="res.company" eval="obj().env.ref('l10n_cz.demo_company_cz')"/>
    </function>
</odoo>

```

## File: data\tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_cz_vat_declaration" model="account.report">
        <field name="name">VAT tax return</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="availability_condition">country</field>
        <field name="country_id" ref="base.cz"/>
        <field name="filter_hierarchy">optional</field>
        <field name="filter_multi_company">disabled</field>
        <field name="default_opening_date_filter">last_month</field>
        <field name="filter_date_range" eval="True"/>
        <field name="filter_period_comparison" eval="True"/>
        <field name="filter_show_draft" eval="True"/>
        <field name="column_ids">
            <record id="l10n_cz_vat_declaration_tax_base" model="account.report.column">
                <field name="name">Tax base</field>
                <field name="expression_label">tax_base</field>
                <field name="figure_type">monetary</field>
            </record>
            <record id="l10n_cz_vat_declaration_output_tax" model="account.report.column">
                <field name="name">Output Tax</field>
                <field name="expression_label">output_tax</field>
                <field name="figure_type">monetary</field>
            </record>
            <record id="l10n_cz_vat_declaration_value" model="account.report.column">
                <field name="name">Value</field>
                <field name="expression_label">value</field>
                <field name="figure_type">monetary</field>
            </record>
            <record id="l10n_cz_vat_declaration_in_full_amount" model="account.report.column">
                <field name="name">In full</field>
                <field name="expression_label">in_full_amount</field>
                <field name="figure_type">monetary</field>
            </record>
            <record id="l10n_cz_vat_declaration_reduced_claim" model="account.report.column">
                <field name="name">Short deduction</field>
                <field name="expression_label">reduced_claim</field>
                <field name="figure_type">monetary</field>
            </record>
            <record id="l10n_cz_vat_declaration_coefficient" model="account.report.column">
                <field name="name">Coefficient</field>
                <field name="expression_label">coefficient</field>
                <field name="figure_type">percentage</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_cz_vat_declaration_line_1" model="account.report.line">
                <field name="name">I. Taxable transactions</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_cz_vat_declaration_line_2" model="account.report.line">
                        <field name="name">Supply of goods or services with a place of performance in the domestic territory</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_3" model="account.report.line">
                                <field name="name">Basic - line 1</field>
                                <field name="code">R1</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_3_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 1 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_3_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 1 Tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_4" model="account.report.line">
                                <field name="name">Reduced - line 2</field>
                                <field name="code">R2</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_4_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 2 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_4_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 2 Tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_5" model="account.report.line">
                        <field name="name">Acquisition of goods from another Member State (16;17(6)(e);19(3)/19(6))</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_6" model="account.report.line">
                                <field name="name">Basic - line 3</field>
                                <field name="code">R3</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_6_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 3 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_6_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 3 Tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_7" model="account.report.line">
                                <field name="name">Reduced - line 4</field>
                                <field name="code">R4</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_7_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 4 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_7_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 4 Tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_8" model="account.report.line">
                        <field name="name">Receipt of services with a place of supply referred to in Article 9(1) from a person registered for tax in another Member State</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_9" model="account.report.line">
                                <field name="name">Basic - line 5</field>
                                <field name="code">R5</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_9_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 5 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_9_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 5 Tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_10" model="account.report.line">
                                <field name="name">Reduced - line 6</field>
                                <field name="code">R6</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_10_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 6 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_10_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 6 Tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_11" model="account.report.line">
                        <field name="name">Import of goods (23)</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_12" model="account.report.line">
                                <field name="name">Basic - line 7</field>
                                <field name="code">R7</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_12_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 7 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_12_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 7 Tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_13" model="account.report.line">
                                <field name="name">Reduced - line 8</field>
                                <field name="code">R8</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_13_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 8 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_13_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 8 Tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_14" model="account.report.line">
                        <field name="name">Acquisition of a new means of transport (19(4)/19(6)) - line 9</field>
                        <field name="code">R9</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_14_tax_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 9 Base</field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_14_output_tax" model="account.report.expression">
                                <field name="label">output_tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 9 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_15" model="account.report.line">
                        <field name="name">Tax reverse charge scheme (92a) - buyer of goods or recipient of services</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_16" model="account.report.line">
                                <field name="name">Basic - line 10</field>
                                <field name="code">R10</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_16_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 10 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_16_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 10 Tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_17" model="account.report.line">
                                <field name="name">Reduced - line 11</field>
                                <field name="code">R11</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_17_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 11 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_17_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 11 Tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_18" model="account.report.line">
                        <field name="name">Other taxable supplies which are subject to the obligation to declare tax on receipt (108)</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_19" model="account.report.line">
                                <field name="name">Basic - line 12</field>
                                <field name="code">R12</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_19_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 12 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_19_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 12 Tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_20" model="account.report.line">
                                <field name="name">Reduced - line 13</field>
                                <field name="code">R13</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_20_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 13 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_20_output_tax" model="account.report.expression">
                                        <field name="label">output_tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 13 Tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_cz_vat_declaration_line_21" model="account.report.line">
                <field name="name">II. Other supplies and supplies with a place of supply outside the domestic territory with a right to tax deduction</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_cz_vat_declaration_line_22" model="account.report.line">
                        <field name="name">Delivery of goods to another Member State (64) - line 20</field>
                        <field name="code">R20</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_22_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 20</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_23" model="account.report.line">
                        <field name="name">Supply of services with a place of supply in another Member State as defined in Article 102(1)(d) and (2) - line 21</field>
                        <field name="code">R21</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_23_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 21</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_24" model="account.report.line">
                        <field name="name">Export of goods (66) - line 22</field>
                        <field name="code">R22</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_24_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 22</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_25" model="account.report.line">
                        <field name="name">Supply of a new means of transport to a person not registered for tax in another Member State (Article 19(4)) - line 23</field>
                        <field name="code">R23</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_25_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 23</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_26" model="account.report.line">
                        <field name="name">Selected transactions (110b(2)) - line 24</field>
                        <field name="code">R24</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_26_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 24</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_27" model="account.report.line">
                        <field name="name">Tax reverse charge scheme (92a) - supplier of goods or services - line 25</field>
                        <field name="code">R25</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_27_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 25</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_28" model="account.report.line">
                        <field name="name">Other transactions with the right to tax deduction (e.g., 24a, 67, 68, 69, 70, 71h, 89, 90, 92) - line 26</field>
                        <field name="code">R26</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_28_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 26</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_cz_vat_declaration_line_29" model="account.report.line">
                <field name="name">III. Additional data</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_cz_vat_declaration_line_30" model="account.report.line">
                        <field name="name">Simplified procedure for the supply of goods in the form of a triangular transaction (17) by a middle person</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_31" model="account.report.line">
                                <field name="name">Acquisition of goods - line 30</field>
                                <field name="code">R30</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_31_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 30</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_32" model="account.report.line">
                                <field name="name">Delivery of goods - line 31</field>
                                <field name="code">R31</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_32_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 31</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_33" model="account.report.line">
                        <field name="name">Import of goods exempted according to 71g - line 32</field>
                        <field name="code">R32</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_33_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 32</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_34" model="account.report.line">
                        <field name="name">Adjustment of tax in case of bad debt (46 et seq. or 74a)</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_35" model="account.report.line">
                                <field name="name">Creditor - line 33</field>
                                <field name="code">R33</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_35_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 33</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_36" model="account.report.line">
                                <field name="name">Debtor - line 34</field>
                                <field name="code">R34</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_36_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 34</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_cz_vat_declaration_line_37" model="account.report.line">
                <field name="name">IV. Entitlement to tax deduction</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_cz_vat_declaration_line_38" model="account.report.line">
                        <field name="name">Taxable supplies received from payers</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_39" model="account.report.line">
                                <field name="name">Basic - line 40</field>
                                <field name="code">R40</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_39_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 40 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_39_in_full_amount" model="account.report.expression">
                                        <field name="label">in_full_amount</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 40 Total</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_39_reduced_claim" model="account.report.expression">
                                        <field name="label">reduced_claim</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_40" model="account.report.line">
                                <field name="name">Reduced - line 41</field>
                                <field name="code">R41</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_40_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 41 Base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_40_in_full_amount" model="account.report.expression">
                                        <field name="label">in_full_amount</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 41 Total</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_40_reduced_claim" model="account.report.expression">
                                        <field name="label">reduced_claim</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_41" model="account.report.line">
                        <field name="name">When importing goods where the customs office is the tax administrator - line 42</field>
                        <field name="code">R42</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_41_tax_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 42 Base</field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_41_in_full_amount" model="account.report.expression">
                                <field name="label">in_full_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 42 Total</field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_41_reduced_claim" model="account.report.expression">
                                <field name="label">reduced_claim</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_42" model="account.report.line">
                        <field name="name">Of the taxable supplies shown on lines 3 to 13</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_43" model="account.report.line">
                                <field name="name">Basic - line 43</field>
                                <field name="code">R43</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_43_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">R3.tax_base + R5.tax_base + R7.tax_base + R9.tax_base + R10.tax_base + R12.tax_base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_43_in_full_amount" model="account.report.expression">
                                        <field name="label">in_full_amount</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">R3.output_tax + R5.output_tax + R7.output_tax + R9.output_tax + R10.output_tax + R12.output_tax</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_43_reduced_claim" model="account.report.expression">
                                        <field name="label">reduced_claim</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_44" model="account.report.line">
                                <field name="name">Reduced - line 44</field>
                                <field name="code">R44</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_44_tax_base" model="account.report.expression">
                                        <field name="label">tax_base</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">R4.tax_base + R6.tax_base + R8.tax_base + R11.tax_base + R13.tax_base</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_44_in_full_amount" model="account.report.expression">
                                        <field name="label">in_full_amount</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">R4.output_tax + R6.output_tax + R8.output_tax + R11.output_tax + R13.output_tax</field>
                                    </record>
                                    <record id="l10n_cz_vat_declaration_line_44_reduced_claim" model="account.report.expression">
                                        <field name="label">reduced_claim</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_45" model="account.report.line">
                        <field name="name">Adjustment of tax deductions according to 75, 77, 79 to 79e - line 45</field>
                        <field name="code">R45</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_45_in_full_amount" model="account.report.expression">
                                <field name="label">in_full_amount</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_45_reduced_claim" model="account.report.expression">
                                <field name="label">reduced_claim</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_46" model="account.report.line">
                        <field name="name">Total tax deduction (40 + 41 + 42 + 43 + 44 + 45) - line 46</field>
                        <field name="code">R46</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_46_in_full_amount" model="account.report.expression">
                                <field name="label">in_full_amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">R40.in_full_amount + R41.in_full_amount + R42.in_full_amount + R43.in_full_amount + R44.in_full_amount + R45.in_full_amount</field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_46_reduced_claim" model="account.report.expression">
                                <field name="label">reduced_claim</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">R40.reduced_claim + R41.reduced_claim + R42.reduced_claim + R43.reduced_claim + R44.reduced_claim + R45.reduced_claim</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_47" model="account.report.line">
                        <field name="name">Value of acquired assets as defined in Section 4(4)(d) and (e) - line 47</field>
                        <field name="code">R47</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_47_tax_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 47 Base</field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_47_in_full_amount" model="account.report.expression">
                                <field name="label">in_full_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 47 Total</field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_47_reduced_claim" model="account.report.expression">
                                <field name="label">reduced_claim</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_cz_vat_declaration_line_48" model="account.report.line">
                <field name="name">V. Reduction of tax deduction entitlement</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_cz_vat_declaration_line_49" model="account.report.line">
                        <field name="name">Transactions exempt from tax without the right to tax deduction - line 50</field>
                        <field name="code">R50</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_49_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 50</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_50" model="account.report.line">
                        <field name="name">Value of transactions not included in the calculation of the coefficient (76(4))</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_51" model="account.report.line">
                                <field name="name">Eligible for deduction - line 51a</field>
                                <field name="code">R51A</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_51_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 51 with deduction</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_52" model="account.report.line">
                                <field name="name">No deduction - line 51b</field>
                                <field name="code">R51B</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_52_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT 51 without deduction</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_53" model="account.report.line">
                        <field name="name">Part of the tax deduction in a reduced amount</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_54" model="account.report.line">
                                <field name="name">Coefficient (%) - line 52a</field>
                                <field name="code">R52A</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_54_coefficient" model="account.report.expression">
                                        <field name="label">coefficient</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_55" model="account.report.line">
                                <field name="name">Deduction - line 52b</field>
                                <field name="code">R52B</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_55_value" model="account.report.expression">
                                        <field name="label">value</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_56" model="account.report.line">
                        <field name="name">Settlement of tax deduction (76(7) to (10))</field>
                        <field name="children_ids">
                            <record id="l10n_cz_vat_declaration_line_57" model="account.report.line">
                                <field name="name">Settlement coefficient (%) - line 53a</field>
                                <field name="code">R53A</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_57_coefficient" model="account.report.expression">
                                        <field name="label">coefficient</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_cz_vat_declaration_line_58" model="account.report.line">
                                <field name="name">Change of deduction - line 53b</field>
                                <field name="code">R53B</field>
                                <field name="expression_ids">
                                    <record id="l10n_cz_vat_declaration_line_58_value" model="account.report.expression">
                                        <field name="label">value</field>
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
            <record id="l10n_cz_vat_declaration_line_59" model="account.report.line">
                <field name="name">VI. Calculation of the tax</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_cz_vat_declaration_line_60" model="account.report.line">
                        <field name="name">Adjustment of tax deduction (78 et seq.) - line 60</field>
                        <field name="code">R60</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_60_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_61" model="account.report.line">
                        <field name="name">Refund of tax (84) - line 61</field>
                        <field name="code">R61</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_61_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT 61</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_62" model="account.report.line">
                        <field name="name">Output tax (sum of 1 to 13 - 61 + tax according to 108 not mentioned elsewhere) - line 62</field>
                        <field name="code">R62</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_62_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">R1.output_tax + R2.output_tax + R3.output_tax + R4.output_tax + R5.output_tax + R6.output_tax + R7.output_tax + R8.output_tax + R9.output_tax +
                                    R10.output_tax + R11.output_tax + R12.output_tax + R13.output_tax - R61.value</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_63" model="account.report.line">
                        <field name="name">Tax deduction (46 In full + 52 Deduction + 53 Change in deduction + 60) - line 63</field>
                        <field name="code">R63</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_63_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">R46.in_full_amount + R52B.value + R53B.value + R60.value</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_64" model="account.report.line">
                        <field name="name">Own tax (62 - 63) - line 64</field>
                        <field name="code">R64</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_64_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">R62.value - R63.value</field>
                                <field name="subformula">if_above(CUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_65" model="account.report.line">
                        <field name="name">Excess deduction (63 - 62) - line 65</field>
                        <field name="code">R65</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_65_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">R63.value - R62.value</field>
                                <field name="subformula">if_above(CUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_cz_vat_declaration_line_66" model="account.report.line">
                        <field name="name">Difference from the last known tax when filing the additional tax return (62 - 63) - line 66</field>
                        <field name="code">R66</field>
                        <field name="expression_ids">
                            <record id="l10n_cz_vat_declaration_line_66_value" model="account.report.expression">
                                <field name="label">value</field>
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
</odoo>

```

## File: data\template\account.account-cz.csv

```csv
"id","name","code","account_type","reconcile","name@cs"
"chart_cz_012000","Intangible results of research and development","012000","asset_non_current","False","Nehmotné výsledky výzkumu a vývoje"
"chart_cz_013000","Software","013000","asset_non_current","False","Software"
"chart_cz_014000","Other valuable rights","014000","asset_non_current","False","Ostatní ocenitelná práva"
"chart_cz_015000","Goodwill","015000","asset_non_current","False","Dobrá vůle"
"chart_cz_016000","Emission allowances","016000","asset_non_current","False","Povolenky na emise"
"chart_cz_017000","Preferential limits","017000","asset_non_current","False","Preferenční limity"
"chart_cz_019000","Other intangible fixed assets","019000","asset_non_current","False","Ostatní dlouhodobý nehmotný majetek"
"chart_cz_021000","Buildings","021000","asset_non_current","False","Stavby"
"chart_cz_022000","Tangible movable assets and groups thereof","022000","asset_non_current","False","Hmotné movité věci a jejich soubory"
"chart_cz_025000","Growing units of permanent crops","025000","asset_non_current","False","Pěstitelské celky trvalých porostů"
"chart_cz_026000","Adult animals and groups thereof","026000","asset_non_current","False","Dospělá zvířata a jejich skupiny"
"chart_cz_027000","Valuation difference on acquired assets","027000","asset_non_current","False","Oceňovací rozdíl k nabytému majetku"
"chart_cz_029000","Other tangible fixed assets","029000","asset_non_current","False","Jiný dlouhodobý hmotný majetek"
"chart_cz_031000","Land","031000","asset_non_current","False","Pozemky"
"chart_cz_032000","Works of art and collections","032000","asset_non_current","False","Umělecká díla a sbírky"
"chart_cz_041000","Acquisition of intangible fixed assets","041000","asset_non_current","False","Pořízení dlouhodobého nehmotného majetku"
"chart_cz_042000","Acquisition of tangible fixed assets","042000","asset_non_current","False","Pořízení dlouhodobého hmotného majetku"
"chart_cz_043000","Acquisition of financial fixed assets","043000","asset_non_current","False","Pořízení dlouhodobého finančního majetku"
"chart_cz_051000","Advances made for intangible fixed assets","051000","asset_non_current","False","Poskytnuté zálohy na dlouhodobý nehmotný majetek"
"chart_cz_052000","Advances made for tangible fixed assets","052000","asset_non_current","False","Poskytnuté zálohy na dlouhodobý hmotný majetek"
"chart_cz_053000","Advances made for fixed financial assets","053000","asset_non_current","False","Poskytnuté zálohy na dlouhodobý finanční majetek"
"chart_cz_061000","Shares - controlled and controlling person","061000","asset_non_current","False","Podíly - ovládaná a ovládajicí osoba"
"chart_cz_062000","Interests in entities under significant influence","062000","asset_non_current","False","Podíly v účetních jednotkách pod podstatným vplyvem"
"chart_cz_063000","Other long-term securities and shares","063000","asset_non_current","False","Ostatní dlouhodobé cenné papíry a podíly"
"chart_cz_065000","Debt securities held to maturity","065000","asset_non_current","False","Dluhové cenné papíry držené do splatnosti"
"chart_cz_066000","Borrowings and loans - controlled and non-controlling interests","066000","asset_non_current","False","Zápůjčky a úvěry - ovládaná neno ovládajicí osoba"
"chart_cz_067000","Other borrowings and loans","067000","asset_non_current","False","Ostatní zápůjčky a úvěry"
"chart_cz_068000","Borrowings and loans - significant influence","068000","asset_non_current","False","Zápůjčky a úvěry - podstatný vliv"
"chart_cz_069000","Other non-current financial assets","069000","asset_non_current","False","Jiný dlouhodobý finanční majetek"
"chart_cz_072000","Rights to intangible research and development results","072000","asset_non_current","False","Oprávky k nehmotným výsledkům výzkumu a vývoje"
"chart_cz_073000","Software rights","073000","asset_non_current","False","Oprávky k softwaru"
"chart_cz_074000","Rights to measurable rights","074000","asset_non_current","False","Oprávky k ocenitelným právům"
"chart_cz_075000","Goodwill rights","075000","asset_non_current","False","Oprávky ke goodwillu"
"chart_cz_079000","Rights to other intangible fixed assets","079000","asset_non_current","False","Oprávky k ostatnímu dlouhodobému nehmotnému majetku"
"chart_cz_081000","Rights to buildings","081000","asset_non_current","False","Oprávky k stavbám"
"chart_cz_082000","Rights to tangible movable assets and tangible assemblies","082000","asset_non_current","False","Oprávky k hmotným movitým věcem a jejich souborům"
"chart_cz_085000","Rights to perennial crops","085000","asset_non_current","False","Oprávky k pěstitelským celkům trvalých porostů"
"chart_cz_086000","Rights to adult animals and groups thereof","086000","asset_non_current","False","Oprávky k dospělým zvířatům a jejich skupinám"
"chart_cz_089000","Rights to other tangible fixed assets","089000","asset_non_current","False","Oprávky k jinému dlouhodobému hmotnému majetku"
"chart_cz_091000","Allowance for intangible fixed assets","091000","asset_non_current","False","Opravná položka k dlouhodobému nehmotnému majetku"
"chart_cz_092000","Allowance for tangible fixed assets","092000","asset_non_current","False","Opravná položka k dlouhodobému hmotnému majetku"
"chart_cz_093000","Allowance for intangible fixed assets in progress","093000","asset_non_current","False","Opravná položka k dlouhodobému nedokončenému nehmotnému majetku"
"chart_cz_094000","Allowance for intangible fixed assets","094000","asset_non_current","False","Opravná položka k dlouhodobému nedokončenému hmotnému majetku"
"chart_cz_095000","Allowance for advances on fixed assets","095000","asset_non_current","False","Opravná položka k poskytnutým zálohám na dlouhodobý majetek"
"chart_cz_096000","Allowance for fixed financial assets","096000","asset_non_current","False","Opravná položka k dlouhodobému finančnímu majetku"
"chart_cz_097000","Valuation difference on acquired assets","097000","asset_non_current","False","Oceňovací rozdíl k nabytému majetku"
"chart_cz_098000","Allowance for valuation difference on acquired assets","098000","asset_non_current","False","Oprávky k oceňovacímu rozdílu k nabytému majetku"
"chart_cz_111000","Acquisition of materials","111000","asset_current","False","Pořízení materiálu"
"chart_cz_112000","Material in stock","112000","asset_current","False","Materiál na skladě"
"chart_cz_119000","Material in transit","119000","asset_current","False","Materiál na cestě"
"chart_cz_121000","Work in progress","121000","asset_current","False","Nedokončená výroba"
"chart_cz_122000","Semi-finished goods","122000","asset_current","False","Polotovary"
"chart_cz_123000","Products","123000","asset_current","False","Výrobky"
"chart_cz_124000","Young and other animals and groups thereof","124000","asset_current","False","Mladá a ostatní zvířata ajejich skupiny"
"chart_cz_131000","Acquisition of goods","131000","asset_current","False","Pořízení zboží"
"chart_cz_132000","Goods in stock and in stores","132000","asset_current","False","Zboží na skladě a v prodejnách"
"chart_cz_139000","Goods on the way","139000","asset_current","False","Zboží na cestě"
"chart_cz_151000","Advances made for materials","151000","asset_current","False","Poskytnuté zálohy na materiál"
"chart_cz_152000","Advances made for animals","152000","asset_current","False","Poskytnuté zálohy na zvířata"
"chart_cz_153000","Advances made for goods","153000","asset_current","False","Poskytnuté zálohy na zboží"
"chart_cz_191000","Allowance for materials","191000","asset_current","False","Opravná položka k materiálu"
"chart_cz_192000","Allowance for work in progress","192000","asset_current","False","Opravná položka k nedokončené výrobě"
"chart_cz_193000","Allowance for semi-finished goods","193000","asset_current","False","Opravná položka k polotovarům"
"chart_cz_194000","Allowance for products","194000","asset_current","False","Opravná položka k výrobkům"
"chart_cz_195000","Allowance for young and other animals and groups thereof","195000","asset_current","False","Opravná položka k mladým a ostatním zvířatům a jejich skupinám"
"chart_cz_196000","Allowance for goods","196000","asset_current","False","Opravná položka ke zboží"
"chart_cz_197000","Allowance for advances on materials","197000","asset_current","False","Opravná položka k zálohám na materiál"
"chart_cz_198000","Allowance for advances on goods","198000","asset_current","False","Opravná položka k zálohám na zboží"
"chart_cz_199000","Allowance for advances on young animals","199000","asset_current","False","Opravná položka k zálohám na mladá zvířata"
"chart_cz_211000","Treasury","211000","asset_cash","False","Pokladna"
"chart_cz_213000","Valuables","213000","asset_cash","False","Ceniny"
"chart_cz_221000","Bank accounts","221000","asset_cash","False","Bankovní účty"
"chart_cz_231000","Short-term loans","231000","asset_current","False","Krátkodobé úvěry"
"chart_cz_232000","Overdraft facilities","232000","asset_current","False","Eskontní úvěry"
"chart_cz_241000","Short-term bonds issued","241000","asset_current","False","Emitované krátkodobé dluhopisy"
"chart_cz_249000","Other short-term financial assistance","249000","asset_current","False","Ostatní krátkodobé finanční výpomoci"
"chart_cz_251000","Equity securities held for trading","251000","asset_current","False","Majetkové cenné papíry k obchodování"
"chart_cz_252000","Treasury shares and treasury shares","252000","asset_current","False","Vlastní akcie a vlastní obchodní podíly"
"chart_cz_253000","Debt securities held for trading","253000","asset_current","False","Dluhové cenné papíry k obchodování"
"chart_cz_254000","Bills of exchange for collection","254000","asset_current","False","Směnky k inkasu"
"chart_cz_255000","Own bonds","255000","liability_current","False","Vlastní dluhopisy"
"chart_cz_256000","Debt securities with maturity up to one year held to maturity","256000","asset_current","False","Dluhové cenné papíry se splatností do jednoho roku držené do splatnosti"
"chart_cz_257000","Other securities","257000","asset_current","False","Ostatní cenné papíry"
"chart_cz_258000","Shares - controlled or controlling person","258000","asset_current","False","Podíly - ovládaná nebo ovládající osoba"
"chart_cz_259000","Acquisition of short-term financial assets","259000","asset_current","False","Pořízování krátkodobého finančního majetku"
"chart_cz_261000","Cash in transit","261000","asset_cash","False","Peníze na cestě"
"chart_cz_291000","Allowance for short-term financial assets","291000","asset_current","False","Opravná položka ke krátkodobému finančnímu majetku"
"chart_cz_311000","Customers","311000","asset_receivable","True","Odběratelé"
"chart_cz_311001","Customers (POS)","311001","asset_receivable","True","Odběratelé (POS)"
"chart_cz_312000","Bills of exchange for collection","312000","asset_receivable","True","Směnky k inkasu"
"chart_cz_313000","Receivables for discounted securities","313000","asset_receivable","True","Pohledávky za eskontované cenné papíry"
"chart_cz_314000","Operating advances provided","314000","asset_receivable","True","Poskytnuté provozní zálohy"
"chart_cz_315000","Other receivables","315000","asset_receivable","True","Ostatní pohledávky"
"chart_cz_321000","Suppliers","321000","liability_payable","True","Dodavatelé"
"chart_cz_322000","Notes receivable","322000","liability_current","False","Směnky k úhradě"
"chart_cz_324000","Operating advances received","324000","liability_current","False","Přijaté provozní zálohy"
"chart_cz_325000","Other payables","325000","liability_current","False","Ostatní závazky"
"chart_cz_331000","Employees","331000","liability_current","False","Zaměstnanci"
"chart_cz_333000","Other payables to employees","333000","liability_current","False","Ostatní závazky vůči zaměstnancům"
"chart_cz_335000","Due from employees","335000","asset_current","False","Pohledávky za zaměstnanci"
"chart_cz_336000","Settlements with social security and health insurance institutions","336000","liability_current","False","Zúčtování s institucemi sociálního zabezpečení a zdravotního pojištění"
"chart_cz_341000","Income tax","341000","liability_current","False","Daň z příjmů"
"chart_cz_342000","Other direct taxes","342000","liability_current","False","Ostatní přímé daně"
"chart_cz_343000","Value added tax","343000","liability_current","False","Daň z přidané hodnoty"
"chart_cz_343001","Tax receivable","343001","asset_current","False","Daňové pohledávky"
"chart_cz_343002","Tax Payable","343002","liability_current","False","Splatná daň"
"chart_cz_343112","VAT reduced rate input","343112","asset_current","False","DPH snížená sazba vstup"
"chart_cz_343115","VAT reduced input rate (15%)","343115","asset_current","False","DPH snížená sazba vstup (15 %)"
"chart_cz_343121","VAT basic rate input","343121","asset_current","False","DPH základní sazba vstup"
"chart_cz_343212","VAT reduced rate output","343212","liability_current","False","DPH snížená sazba výstup"
"chart_cz_343215","VAT reduced rate output (15%)","343215","liability_current","False","DPH snížená sazba výstup (15 %)"
"chart_cz_343221","VAT basic rate output","343221","liability_current","False","DPH základní sazba výstup"
"chart_cz_345000","Other taxes and charges","345000","liability_current","False","Ostatní daně a poplatky"
"chart_cz_346000","Subsidies from the state budget","346000","liability_current","False","Dotace ze státního rozpočtu"
"chart_cz_347000","Other deliveries","347000","liability_current","False","Ostatní dodace"
"chart_cz_349000","VAT link account","349000","liability_current","False","Spojovací účet k DPH"
"chart_cz_351000","Receivables - controlled or controlling person","351000","asset_current","False","Pohledávky – ovládaná nebo ovládající osoba"
"chart_cz_352000","Receivables - significant influence","352000","asset_current","False","Pohledávky – podstatný vliv"
"chart_cz_353000","Receivables for subscribed share capital","353000","asset_current","False","Pohledávky za upsaný základní kapitál"
"chart_cz_354000","Receivables for shareholders in settlement of losses","354000","asset_current","False","Pohledávky za společníky při úhradě ztráty"
"chart_cz_355000","Other receivables from shareholders of the corporation","355000","asset_current","False","Ostatní pohledávky za společníky obchodní korporace"
"chart_cz_358000","Receivables from shareholders associated in the company","358000","asset_current","False","Pohledávky za společníky združenými ve společnosti"
"chart_cz_361000","Accounts payable - controlled or controlling person","361000","liability_current","False","Závazky – ovládaná nebo ovládající osoba"
"chart_cz_362000","Liabilities - significant influence","362000","liability_current","False","Závazky - podstatný vliv"
"chart_cz_364000","Liabilities to shareholders on distribution of profits","364000","liability_current","False","Závazky ke společníkům při rozdělování zisku"
"chart_cz_365000","Other liabilities to shareholders of the corporation","365000","liability_current","False","Ostatní závazky ke společníkům obchodní korporace"
"chart_cz_366000","Liabilities to partners and members of the cooperative from dependent activities","366000","liability_current","False","Závazky ke společníkům a členům družstva ze závislé činnosti"
"chart_cz_367000","Liabilities from subscribed outstanding securities and deposits","367000","liability_current","False","Závazky z upsaných nesplacených cenných papírů a vkladů"
"chart_cz_368000","Liabilities to associates of the company","368000","liability_current","False","Závazky ke společníkům sdruženým ve společnosti"
"chart_cz_371000","Receivables from the sale of the plant","371000","asset_current","False","Pohledávky z prodeje závodu"
"chart_cz_372000","Liabilities from purchase of plant","372000","liability_current","False","Závazky z koupě závodu"
"chart_cz_373000","Receivables and payables from fixed term operations","373000","asset_current","False","Pohledávky a závazky z pevných termínových operací"
"chart_cz_374000","Receivables from lease and rent","374000","asset_current","False","Pohledávky z nájmu a pachtu"
"chart_cz_375000","Receivables from bonds issued","375000","asset_current","False","Pohledávky z emitovaných dluhopisů"
"chart_cz_376000","Purchased options","376000","asset_current","False","Nakoupené opce"
"chart_cz_377000","Options sold","377000","liability_current","False","Prodané opce"
"chart_cz_378000","Other receivables","378000","asset_current","False","Jiné pohledávky"
"chart_cz_379000","Other payables","379000","liability_current","False","Jiné závazky"
"chart_cz_381000","Accrued expenses","381000","asset_current","False","Náklady příštích období"
"chart_cz_382000","Comprehensive accrued expenses","382000","asset_current","False","Komplexní náklady příštích období"
"chart_cz_383000","Accrued expenses","383000","liability_current","False","Výdaje příštích období"
"chart_cz_384000","Deferred income","384000","liability_current","False","Výnosy příštích období"
"chart_cz_385000","Deferred income","385000","asset_current","False","Příjmy příštích období"
"chart_cz_388000","Accounts receivable active","388000","asset_current","False","Dohadné účty aktivní"
"chart_cz_389000","Accounts receivable and payable","389000","liability_current","False","Dohadné účty pasivní"
"chart_cz_391000","Allowance for receivables","391000","asset_current","False","Opravná položka k pohledávkám"
"chart_cz_395000","Internal clearing","395000","asset_current","False","Vnitřní zúčtování"
"chart_cz_398000","Company (association) liaison account","398000","liability_current","False","Spojovací účet při společnosti (sdružení)"
"chart_cz_411000","Share capital","411000","equity","False","Základní kapitál"
"chart_cz_412000","Premium","412000","equity","False","Ážio"
"chart_cz_413000","Other capital funds","413000","equity","False","Ostatní kapitálové fondy"
"chart_cz_414000","Valuation differences on revaluation of assets and liabilities","414000","equity","False","Oceňovací rozdíly z přecenění majetku a závazků"
"chart_cz_416000","Revaluation differences on corporate reorganisations","416000","equity","False","Rozdíly z přecenění při přemenách obchodních korporací"
"chart_cz_417000","Differences on conversions of business corporations","417000","equity","False","Rozdíly z přeměn obchodních korporací"
"chart_cz_418000","Valuation differences on revaluations on conversions of business corporations","418000","equity","False","Oceňovací rozdíly z přecenění při přeměnách obchodních korporací"
"chart_cz_419000","Changes in share capital","419000","equity","False","Změny základního kapitálu"
"chart_cz_421000","Reserve fund","421000","equity","False","Rezervní fond"
"chart_cz_422000","Undivided fund","422000","equity","False","Nedělitelný fond"
"chart_cz_423000","Statutory funds","423000","equity","False","Statutární fondy"
"chart_cz_424000","Other funds from profit","424000","equity","False","Ostatní fondy ze zisku"
"chart_cz_426000","Other result of previous years","426000","equity","False","Jiný výsledek hospodáření minulých let"
"chart_cz_427000","Other funds","427000","equity","False","Ostatní fondy"
"chart_cz_428000","Retained earnings of previous years","428000","equity","False","Nerozdělený zisk minulých let"
"chart_cz_429000","Unrelieved loss of previous years","429000","equity","False","Neuhrazená ztráta minulých let"
"chart_cz_431000","Outturn under approval","431000","equity_unaffected","False","Výsledek hospodaření v schvalovacím řízení"
"chart_cz_432000","Advances on profit-sharing","432000","equity","False","Zálohy na podíly na zisku"
"chart_cz_451000","Reserves under special legislation","451000","equity","False","Rezervy podle zvláštních právních předpisů"
"chart_cz_452000","Provision for pensions and similar liabilities","452000","equity","False","Rezerva na důchody a podobné závazky"
"chart_cz_453000","Provision for income tax","453000","equity","False","Rezerva na daň z příjmů"
"chart_cz_459000","Other reserves","459000","equity","False","Ostatní rezervy"
"chart_cz_461000","Long-term loans","461000","equity","False","Dlouhodobé úvěry"
"chart_cz_471000","Long-term liabilities - controlled or controlling person","471000","liability_non_current","False","Dlouhodobé závazky – ovládaná nebo ovládající osoba"
"chart_cz_472000","Long-term liabilities - significant influence","472000","liability_non_current","False","Dlouhodobé závazky – podstatný vliv"
"chart_cz_473000","Bonds issued","473000","liability_non_current","False","Emitované dluhopisy"
"chart_cz_473100","Long-term bonds issued","473100","liability_non_current","False","Vydané dlouhodobé dluhopisy"
"chart_cz_473200","Short-term bonds issued","473200","liability_non_current","False","Vydané krátkodobé dluhopisy"
"chart_cz_474000","Liabilities under leases and tenancies","474000","liability_non_current","False","Závazky z nájmu a pachtu"
"chart_cz_475000","Long-term advances received","475000","liability_non_current","False","Dlouhodobé přijaté zálohy"
"chart_cz_478000","Long-term notes receivable","478000","liability_non_current","False","Dlouhodobé směnky k úhradě"
"chart_cz_479000","Other long-term liabilities","479000","liability_non_current","False","Ostatní dlouhodobé závazky"
"chart_cz_481000","Deferred tax liability and receivable","481000","liability_non_current","False","Odložený daňový závazek a pohledávka"
"chart_cz_491000","Sole proprietorship account","491000","equity","False","Účet individuálního podnikatele"
"chart_cz_501000","Consumption of ma","501000","expense","False","Spotřeba materiálu"
"chart_cz_502000","Energy consumption","502000","expense","False","Spotřeba energie"
"chart_cz_503000","Consumption of other non-stackable supplies","503000","expense","False","Spotřeba ostatních neskladovatelných dodávek"
"chart_cz_504000","Goods sold","504000","expense","False","Prodané zboží"
"chart_cz_511000","Repairs and maintenance","511000","expense","False","Opravy a udržování"
"chart_cz_512000","Travel","512000","expense","False","Cestovné"
"chart_cz_513000","Representation costs","513000","expense","False","Náklady na reprezentaci"
"chart_cz_518000","Other services","518000","expense","False","Ostatní služby"
"chart_cz_521000","Payroll costs","521000","expense","False","Mzdové náklady"
"chart_cz_522000","Income of partners of a corporation from dependent activities","522000","expense","False","Příjmy společníků obchodní korporace ze závislé činnosti"
"chart_cz_523000","Remuneration to members of the bodies of the corporation","523000","expense","False","Odměny členům orgánů obchodní korporace"
"chart_cz_524000","Statutory social and health insurance","524000","expense","False","Zákonné sociální a zdravotní pojištění"
"chart_cz_525000","Other social insurance","525000","expense","False","Ostatní sociální pojištění"
"chart_cz_526000","Health and social insurance of individual entrepreneur","526000","expense","False","Zdravotní a sociální pojištění individuálního podnikatele"
"chart_cz_527000","Statutory social costs","527000","expense","False","Zákonné sociální náklady"
"chart_cz_528000","Other social costs","528000","expense","False","Ostatní sociální náklady"
"chart_cz_531000","Road tax","531000","expense","False","Daň silniční"
"chart_cz_532000","Real estate tax","532000","expense","False","Daň z nemovitých věcí"
"chart_cz_538000","Other taxes and charges","538000","expense","False","Ostatní daně a poplatky"
"chart_cz_541000","Residual value of intangible and tangible fixed assets sold","541000","expense","False","Zůstatková cena prodaného dlouhodobého nehmotného a hmotného majetku"
"chart_cz_542000","Materials sold","542000","expense","False","Prodaný materiál"
"chart_cz_543000","Donations","543000","expense","False","Dary"
"chart_cz_544000","Contractual penalties and default interest","544000","expense","False","Smluvní pokuty a úroky z prodlení"
"chart_cz_545000","Other fines and penalties","545000","expense","False","Ostatní pokuty a penále"
"chart_cz_546000","Write-off of receivables","546000","expense","False","Odpis pohledávky"
"chart_cz_547000","Extraordinary operating expenses","547000","expense","False","Mimořádné provozní náklady"
"chart_cz_548000","Other operating expenses","548000","expense","False","Ostatní provozní náklady"
"chart_cz_549000","Deficits and losses from operating activities","549000","expense","False","Manka a škody z provozní činnosti"
"chart_cz_551000","Amortisation of intangible and tangible fixed assets","551000","expense_depreciation","False","Odpisy dlouhodobého nehmotného a hmotného majetku"
"chart_cz_552000","Creation and settlement of legal reserves under special legislation","552000","expense","False","Tvorba a zúčtování zákonných rezerv podle zvláštních právních předpisů"
"chart_cz_554000","Creation and settlement of other provisions","554000","expense","False","Tvorba a zúčtování ostatních rezerv"
"chart_cz_555000","Formation and settlement of comprehensive accrued expenses","555000","expense","False","Tvorba a zúčtování komplexních nákladů příštích období"
"chart_cz_557000","Recognition of valuation allowance on acquired assets","557000","expense","False","Zúčtování oprávky k oceňovacímu rozdílu k nabytému majetku"
"chart_cz_558000","Generation and recognition of legal provisions in operating activities","558000","expense","False","Tvorba a zúčtování zákonných opravných položek v provozní činnosti"
"chart_cz_559000","Generation and recognition of valuation allowances in operating activities","559000","expense","False","Tvorba a zúčtování opravných položek v provozní činnosti"
"chart_cz_561000","Securities and shares sold","561000","expense","False","Prodané cenné papíry a podíly"
"chart_cz_562000","Interest","562000","expense","False","Úroky"
"chart_cz_563000","Exchange losses","563000","expense","False","Kursové ztráty"
"chart_cz_564000","Securities revaluation charges","564000","expense","False","Náklady na přecenění cenných papírů"
"chart_cz_565000","Donations made in the financial area","565000","expense","False","Poskytnuté dary ve finanční oblasti"
"chart_cz_566000","Cost of financial assets","566000","expense","False","Náklady z finančního majetku"
"chart_cz_567000","Cost of derivative transactions","567000","expense","False","Náklady z derivátových operací"
"chart_cz_568000","Other financial expenses","568000","expense","False","Ostatní finanční náklady"
"chart_cz_569000","Deficits and losses on financial assets","569000","expense","False","Manka a škody na finančním majetku"
"chart_cz_574000","Creation and settlement of financial provisions","574000","expense","False","Tvorba a zúčtování finančních rezerv"
"chart_cz_579000","Provisioning and settlement of provisions in financial activities","579000","expense","False","Tvorba a zúčtování opravných položek ve finanční činnosti"
"chart_cz_581000","Change in work in progress","581000","expense","False","Změna stavu nedokončené výroby"
"chart_cz_582000","Change in semi-finished goods","582000","expense","False","Změna stavu polotovarů"
"chart_cz_583000","Change in products","583000","expense","False","Změna stavu výrobků"
"chart_cz_584000","Change in young and other animals","584000","expense","False","Změna stavu mladých a ostatních zvířat"
"chart_cz_585000","Activation of materials and goods","585000","expense","False","Aktivace materiálu a zboží"
"chart_cz_586000","Activation of in-house services","586000","expense","False","Aktivace vnitropodnikových služeb"
"chart_cz_587000","Activation of intangible fixed assets","587000","expense","False","Aktivace dlouhodobého nehmotného majetku"
"chart_cz_588000","Activation of tangible fixed assets","588000","expense","False","Aktivace dlouhodobého hmotného majetku"
"chart_cz_591000","Income tax payable","591000","expense","False","Daň z příjmů splatná"
"chart_cz_592000","Deferred income tax","592000","expense","False","Daň z příjmů odložená"
"chart_cz_593000","Income tax on extraordinary activities - payable","593000","expense","False","Daň z příjmů z mimořádné činnosti - splatná"
"chart_cz_595000","Additional income tax payments","595000","expense","False","Dodatečné odvody daně z příjmů"
"chart_cz_596000","Transfer of share of profit or loss to partners of v.o.s. and general partners of k.s.","596000","expense","False","Převod podílu na výsledku hospodaření společníkům v.o.s. a komplementářům k.s."
"chart_cz_597000","Transfer of operating expenses","597000","expense","False","Převod provozních nákladů"
"chart_cz_598000","Transfer of finance costs","598000","expense","False","Převod finančních nákladů"
"chart_cz_599000","Change in income tax provision","599000","expense","False","Změna stavu rezervy na daň z příjmů"
"chart_cz_601000","Revenue from own products","601000","income","False","Tržby za vlastní výrobky"
"chart_cz_602000","Revenue from sale of services","602000","income","False","Tržby z prodeje služeb"
"chart_cz_604000","Sales of goods","604000","income","False","Tržby za zboží"
"chart_cz_641000","Revenue from the sale of intangible and tangible fixed assets","641000","income","False","Tržby z prodeje dlouhodobého nehmotného a hmotného majetku"
"chart_cz_642000","Revenue from the sale of materials","642000","income","False","Tržby z prodeje materiálu"
"chart_cz_643000","Donations received in operating activities","643000","income","False","Přijaté dary v provozní oblasti"
"chart_cz_644000","Contractual penalties and interest on late payments","644000","income","False","Smluvní pokuty a úroky z prodlení"
"chart_cz_645000","Proceeds from assigned receivables","645000","income","False","Výnosy z postoupených pohledávek"
"chart_cz_646000","Revenue from written-off receivables","646000","income","False","Výnosy z odepsaných pohledávek"
"chart_cz_647000","Extraordinary operating income","647000","income","False","Mimořádné provozní výnosy"
"chart_cz_648000","Other operating income","648000","income","False","Ostatní provozní výnosy"
"chart_cz_649000","Amortisation of negative goodwill and recognition of valuation allowance on acquired assets","649000","income","False","Odpis záporného goodwillu a zúčtování oprávky k oceň. rozdílu k nabytému majetku"
"chart_cz_661000","Proceeds from sale of securities and shares","661000","income","False","Tržby z prodeje cenných papírů a podílů"
"chart_cz_662000","Interest","662000","income","False","Úroky"
"chart_cz_663000","Foreign exchange gains","663000","income","False","Kurzové zisky"
"chart_cz_664000","Gains on revaluation of securities","664000","income","False","Výnosy z přecenění cenných papírů"
"chart_cz_665000","Proceeds from non-current financial assets","665000","income","False","Výnosy z dlouhodobého finančního majetku"
"chart_cz_666000","Proceeds from short-term financial assets","666000","income","False","Výnosy z krátkodobého finančního majetku"
"chart_cz_667000","Income from derivative transactions","667000","income","False","Výnosy z derivátových operací"
"chart_cz_668000","Other financial income","668000","income","False","Ostatní finanční výnosy"
"chart_cz_669000","Donations received in the financial area","669000","income","False","Přijaté dary ve finanční oblasti"
"chart_cz_697000","Transfer of operating income","697000","income","False","Převod provozních výnosů"
"chart_cz_698000","Transfer of financial income","698000","income","False","Převod finančních výnosů"
"chart_cz_699000","Revenue from economic centres","699000","income","False","Výnosy hospodářských středisek"
"chart_cz_701000","Initial balance sheet account","701000","off_balance","False","Počáteční účet rozvažný"
"chart_cz_702000","Closing balance sheet account","702000","off_balance","False","Konečný účet rozvažný"
"chart_cz_710000","Profit and loss account","710000","off_balance","False","Účet zisků a ztrát"

```

## File: data\template\account.fiscal.position-cz.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@cs"
"fiscal_position_domestic_cz","1","Domestic partners","1","1","base.cz","","","","Domácí partneři"
"fiscal_position_eu_private_cz","2","Private trade with the EU","1","","","base.europe","","","Soukromý obchod s EU"
"fiscal_position_eu_cz","3","Trade with the EU","1","1","","base.europe","l10n_cz_supply_goods_eu","l10n_cz_not_included_vat","Obchod s EU"
"","","","","","","","l10n_cz_supply_service_eu","l10n_cz_not_included_vat",""
"","","","","","","","l10n_cz_12_purchase_goods_eu","l10n_cz_21_other_supplies_chargeable",""
"","","","","","","","l10n_cz_21_acquisition_goods_eu","l10n_cz_21_other_supplies_chargeable",""
"","","","","","","","l10n_cz_12_receipt_service_person_eu","l10n_cz_21_other_supplies_chargeable",""
"","","","","","","","l10n_cz_21_receipt_service_person_eu","l10n_cz_21_other_supplies_chargeable",""

```

## File: data\template\account.group-cz.csv

```csv
"id","code_prefix_start","name"
"chart_cz_0","0","Dlouhodobý majetek"
"chart_cz_01","01","Dlouhodobý nehmotný majetek"
"chart_cz_012","012","Nehmotné výsledky výzkumu a vývoje"
"chart_cz_013","013","Software"
"chart_cz_014","014","Ostatní ocenitelná práva"
"chart_cz_015","015","Goodwill"
"chart_cz_016","016","Povolenky na emise"
"chart_cz_017","017","Preferenční limity"
"chart_cz_019","019","Ostatní dlouhodobý nehmotný majetek"
"chart_cz_02","02","Dlouhodobý hmotný majetek - odpisovaný"
"chart_cz_021","021","Stavby"
"chart_cz_022","022","Hmotné movité věci a jejich soubory"
"chart_cz_025","025","Pěstitelské celky trvalých porostů"
"chart_cz_026","026","Dospělá zvířata a jejich skupiny"
"chart_cz_027","027","Oceňovací rozdíl k nabytému majetku"
"chart_cz_029","029","Jiný dlouhodobý hmotný majetek"
"chart_cz_03","03","Dlouhodobý hmotný majetek - neodpisovaný"
"chart_cz_031","031","Pozemky"
"chart_cz_032","032","Umělecká díla a sbírky"
"chart_cz_04","04","Nedokončený dlouhodobý nehmotný a hmotný majetek"
"chart_cz_041","041","Pořízení dlouhodobého nehmotného majetku"
"chart_cz_042","042","Pořízení dlouhodobého hmotného majetku"
"chart_cz_043","043","Pořízení dlouhodobého finančního majetku"
"chart_cz_05","05","Poskytnuté zálohy na dlouhodobý majetek"
"chart_cz_051","051","Poskytnuté zálohy na dlouhodobý nehmotný majetek"
"chart_cz_052","052","Poskytnuté zálohy na dlouhodobý hmotný majetek"
"chart_cz_055","055","Poskytnuté zálohy na dlouhodobý finanční majetek"
"chart_cz_06","06","Dlouhodobý finanční majetek"
"chart_cz_061","061","Podíly - ovládaná a ovládajicí osoba"
"chart_cz_062","062","Podíly v účetních jednotkách pod podstatným vplyvem"
"chart_cz_063","063","Ostatní dlouhodobé cenné papíry a podíly"
"chart_cz_065","065","Dluhové cenné papíry držené do splatnosti"
"chart_cz_066","066","Zápůjčky a úvéry - ovládaná neno ovládajicí osoba"
"chart_cz_067","067","Zápůjčky a úvéry - podstatní vliv"
"chart_cz_068","068","Ostatní zápůjčky a úvéry"
"chart_cz_069","069","Jiný dlouhodobý finanční majetek"
"chart_cz_07","07","Oprávky k dlouhodobému nehmotnému majetku"
"chart_cz_072","072","Oprávky k nehmotným výsledkům výzkumu a vývoje"
"chart_cz_073","073","Oprávky k softwaru"
"chart_cz_074","074","Oprávky k ocenitelným právům"
"chart_cz_075","075","Oprávky ke goodwillu"
"chart_cz_079","079","Oprávky k ostatnímu dlouhodobému nehmotnému majetku"
"chart_cz_08","08","Oprávky k dlouhodobému hmotnému majetku"
"chart_cz_081","081","Oprávky k stavbám"
"chart_cz_082","082","Oprávky k hmotným movitým věcem a jejich souborům"
"chart_cz_085","085","Oprávky k pěstitelským celkům trvalých porostů"
"chart_cz_086","086","Oprávky k dospělým zvířatům a jejich skupinám"
"chart_cz_089","089","Oprávky k jinému dlouhodobému hmotnému majetku"
"chart_cz_09","09","Opravné položky k dlouhodobému majetku"
"chart_cz_091","091","Opravná položka k dlouhodobému nehmotnému majetku"
"chart_cz_092","092","Opravná položka k dlouhodobému hmotnému majetku"
"chart_cz_093","093","Opravná položka k dlouhodobému nedokončenému nehmotnému majetku"
"chart_cz_094","094","Opravná položka k dlouhodobému nedokončenému hmotnému majetku"
"chart_cz_095","095","Opravná položka k poskytnutým zálohám na dlouhodobý majetek"
"chart_cz_096","096","Opravná položka k dlouhodobému finančnímu majetku"
"chart_cz_097","097","Oceňovací rozdíl k nabytému majetku"
"chart_cz_098","098","Oprávky k oceňovacímu rozdílu k nabytému majetku"
"chart_cz_1","1","Zásoby"
"chart_cz_11","11","Materiál"
"chart_cz_111","111","Pořízení materiálu"
"chart_cz_112","112","Materiál na skladě"
"chart_cz_119","119","Materiál na cestě"
"chart_cz_12","12","Zásoby vlastní činnosti"
"chart_cz_121","121","Nedokončená výroba"
"chart_cz_122","122","Polotovary"
"chart_cz_123","123","Výrobky"
"chart_cz_124","124","Mladá a ostatní zvířata ajejich skupiny"
"chart_cz_13","13","Zboží"
"chart_cz_131","131","Pořízení zboží"
"chart_cz_132","132","Zboží na skladě a v prodejnách"
"chart_cz_139","139","Zboží na cestě"
"chart_cz_15","15","Poskytnuté zálohy na zásoby"
"chart_cz_151","151","Poskytnuté zálohy na materiál"
"chart_cz_152","152","Poskytnuté zálohy na zvířata"
"chart_cz_153","153","Poskytnuté zálohy na zboží"
"chart_cz_19","19","Opravné položky k zásobám"
"chart_cz_191","191","Opravná položka k materiálu"
"chart_cz_192","192","Opravná položka k nedokončené výrobě"
"chart_cz_193","193","Opravná položka k polotovarům"
"chart_cz_194","194","Opravná položka k výrobkům"
"chart_cz_195","195","Opravná položka k mladým a ostatním zvířatům a jejich skupinám"
"chart_cz_196","196","Opravná položka ke zboží"
"chart_cz_197","197","Opravná položka k zálohám na materiál"
"chart_cz_198","198","Opravná položka k zálohám na zvířata"
"chart_cz_199","199","Opravná položka k zálohám na zboží"
"chart_cz_2","2","Krátkodobý finanční majetek a peněžní prostředky"
"chart_cz_21","21","Peněžní prostředky v pokladně"
"chart_cz_211","211","Pokladna"
"chart_cz_213","213","Ceniny"
"chart_cz_22","22","Peněžní prostředky na účtech"
"chart_cz_221","221","Bankovní účty"
"chart_cz_23","23","Krátkodobé úvěry"
"chart_cz_231","231","Krátkodobé úvěry"
"chart_cz_232","232","Eskontní úvěry"
"chart_cz_24","24","Krátkodobé finanční výpomoci"
"chart_cz_241","241","Emitované krátkodobé dluhopisy"
"chart_cz_249","249","Ostatní krátkodobé finanční výpomoci"
"chart_cz_25","25","Krátkodobý finanční majetek"
"chart_cz_251","251","Majetkové cenné papíry k obchodování"
"chart_cz_252","252","Vlastní akcie a vlastní obchodní podíly"
"chart_cz_253","253","Dluhové cenné papíry k obchodování"
"chart_cz_255","255","Vlastní dluhopisy"
"chart_cz_256","256","Dluhové cenné papíry se splatností do jednoho roku držené do splatnosti"
"chart_cz_257","257","Ostatní cenné papíry"
"chart_cz_259","259","Pořízování krátkodobého finančního majetku"
"chart_cz_26","26","Převody mezi finančními účty"
"chart_cz_261","261","Peníze na cestě"
"chart_cz_29","29","Opravné položky ke krátkodobému finančnímu majetku"
"chart_cz_291","291","Opravná položka ke krátkodobému finančnímu majetku"
"chart_cz_3","3","Zúčtovací vztahy"
"chart_cz_31","31","Pohledávky"
"chart_cz_311","311","Odběratelé"
"chart_cz_313","313","Pohledávky za eskontované cenné papíry"
"chart_cz_314","314","Poskytnuté provozní zálohy"
"chart_cz_315","315","Ostatní pohledávky"
"chart_cz_32","32","Závazky (krátkodobé)"
"chart_cz_321","321","Dodavatelé"
"chart_cz_322","322","Směnky k úhradě"
"chart_cz_324","324","Přijaté provozní zálohy"
"chart_cz_325","325","Ostatní závazky"
"chart_cz_33","33","Zúčtování se zaměstnanci a institucemi"
"chart_cz_331","331","Zaměstnanci"
"chart_cz_333","333","Ostatní závazky vůči zaměstnancům"
"chart_cz_335","335","Pohledávky za zaměstnanci"
"chart_cz_336","336","Zúčtování s institucemi sociálního zabezpečení a zdravotního pojištění"
"chart_cz_34","34","Zúčtování daní a dotací"
"chart_cz_341","341","Daň z příjmů"
"chart_cz_342","342","Ostatní přímé daně"
"chart_cz_343","343","Daň z přidané hodnoty"
"chart_cz_345","345","Ostatní daně a poplatky"
"chart_cz_346","346","Dotace ze státního rozpočtu"
"chart_cz_347","347","Ostatní dodace"
"chart_cz_349","349","Spojovací účet k DPH"
"chart_cz_35","35","Pohledávky za společníky"
"chart_cz_351","351","Pohledávky – ovládaná nebo ovládající osoba"
"chart_cz_352","352","Pohledávky – podstatný vliv"
"chart_cz_353","353","Pohledávky za upsaný základní kapitál"
"chart_cz_354","354","Pohledávky za společníky při úhradě ztráty"
"chart_cz_355","355","Ostatní pohledávky za společníky obchodní korporace"
"chart_cz_358","358","Pohledávky za společníky združenými ve společnosti"
"chart_cz_36","36","Závazky ke společníkům"
"chart_cz_361","361","Závazky – ovládaná nebo ovládající osoba"
"chart_cz_362","362","Závazky - podstatný vliv"
"chart_cz_364","364","Závazky ke společníkům při rozdělování zisku"
"chart_cz_365","365","Ostatní závazky ke společníkům obchodní korporace"
"chart_cz_366","366","Závazky ke společníkům a členům družstva ze závislé činnosti"
"chart_cz_367","367","Závazky z upsaných nesplacených cenných papírů a vkladů"
"chart_cz_368","368","Závazky ke společníkům sdruženým ve společnosti"
"chart_cz_37","37","Jiné pohledávky a závazky"
"chart_cz_371","371","Pohledávky z prodeje závodu"
"chart_cz_372","372","Závazky z koupě závodu"
"chart_cz_373","373","Pohledávky a závazky z pevných termínových operací"
"chart_cz_374","374","Pohledávky z nájmu a pachtu"
"chart_cz_375","375","Pohledávky z emitovaných dluhopisů"
"chart_cz_376","376","Nakoupené opce"
"chart_cz_377","377","Prodané opce"
"chart_cz_378","378","Jiné pohledávky"
"chart_cz_379","379","Jiné závazky"
"chart_cz_38","38","Přechodné účty aktiv a pasiv"
"chart_cz_381","381","Náklady příštích období"
"chart_cz_382","382","Komplexní náklady příštích období"
"chart_cz_383","383","Výdaje příštích období"
"chart_cz_384","384","Výnosy příštích období"
"chart_cz_385","385","Příjmy příštích období"
"chart_cz_388","388","Dohadné účty aktivní"
"chart_cz_389","389","Dohadné účty pasivní"
"chart_cz_39","39","Opravná položka k zúčtovacím vztahům a vnitřní zúčtování"
"chart_cz_391","391","Opravná položka k pohledávkám"
"chart_cz_395","395","Vnitřní zúčtování"
"chart_cz_398","398","Spojovací účet při společnosti (sdružení)"
"chart_cz_4","4","Kapitálové účty a dlouhodobé závazky"
"chart_cz_41","41","Základní kapitál a kapitálové fondy"
"chart_cz_411","411","Základní kapitál"
"chart_cz_412","412","Ážio"
"chart_cz_413","413","Ostatní kapitálové fondy"
"chart_cz_414","414","Oceňovací rozdíly z přecenění majetku a závazků"
"chart_cz_416","416","Rozdíly z přecenění při přemenách obchodních korporací"
"chart_cz_417","417","Rozdíly z přeměn obchodních korporací"
"chart_cz_418","418","Oceňovací rozdíly z přecenění při přeměnách obchodních korporací"
"chart_cz_419","419","Změny základního kapitálu"
"chart_cz_42","42","Fondy ze zisku a převedené výsledky hospodaření"
"chart_cz_421","421","Rezervní fond"
"chart_cz_422","422","Nedělitelný fond"
"chart_cz_423","423","Statutární fondy"
"chart_cz_424","424","Ostatní fondy ze zisku"
"chart_cz_426","426","Jiný výsledek hospodáření minulých let"
"chart_cz_428","428","Nerozdělený zisk minulých let"
"chart_cz_429","429","Neuhrazená ztráta minulých let"
"chart_cz_43","43","Výsledek hospodaření"
"chart_cz_431","431","Výsledek hospodaření v schvalovacím řízení"
"chart_cz_432","432","Zálohy na podíly na zisku"
"chart_cz_45","45","Rezervy"
"chart_cz_451","451","Rezervy podle zvláštních právních předpisů"
"chart_cz_453","453","Rezerva na daň z příjmů"
"chart_cz_459","459","Ostatní rezervy"
"chart_cz_46","46","Dlouhodobé závazky k úvěrovým institucím"
"chart_cz_461","461","Dlouhodobé úvěry"
"chart_cz_47","47","dlouhodobé závazky"
"chart_cz_471","471","Dlouhodobé závazky – ovládaná nebo ovládající osoba"
"chart_cz_472","472","Dlouhodobé závazky – podstatný vliv"
"chart_cz_473","473","Emitované dluhopisy"
"chart_cz_474","474","Závazky z nájmu a pachtu"
"chart_cz_475","475","Dlouhodobé přijaté zálohy"
"chart_cz_478","478","Dlouhodobé směnky k úhradě"
"chart_cz_479","479","Ostatní dlouhodobé závazky"
"chart_cz_48","48","Odložený daňový závazek a pohledávka"
"chart_cz_481","481","Odložený daňový závazek a pohledávka"
"chart_cz_49","49","Individuální podnikatel"
"chart_cz_491","491","Účet individuálního podnikatele"
"chart_cz_5","5","Náklady"
"chart_cz_50","50","Spotřebované nákupy"
"chart_cz_501","501","Spotřeba materiálu"
"chart_cz_502","502","Spotřeba energie"
"chart_cz_503","503","Spotřeba ostatních neskladovatelných dodávek"
"chart_cz_504","504","Prodané zboží"
"chart_cz_51","51","Služby"
"chart_cz_511","511","Opravy a udržování"
"chart_cz_512","512","Cestovné"
"chart_cz_513","513","Náklady na reprezentaci"
"chart_cz_518","518","Ostatní služby"
"chart_cz_52","52","Osobní náklady"
"chart_cz_521","521","Mzdové náklady"
"chart_cz_522","522","Příjmy společníků obchodní korporace ze závislé činnosti"
"chart_cz_523","523","Odměny členům orgánů obchodní korporace"
"chart_cz_524","524","Zákonné sociální a zdravotní pojištění"
"chart_cz_525","525","Ostatní sociální pojištění"
"chart_cz_526","526","Zdravotní a sociální pojištění individuálního podnikatele"
"chart_cz_527","527","Zákonné sociální náklady"
"chart_cz_528","528","Ostatní sociální náklady"
"chart_cz_53","53","Daně a poplatky"
"chart_cz_531","531","Daň silniční"
"chart_cz_532","532","Daň z nemovitých věcí"
"chart_cz_538","538","Ostatní daně a poplatky"
"chart_cz_54","54","Jiné provozní náklady"
"chart_cz_541","541","Zůstatková cena prodaného dlouhodobého nehmotného a hmotného majetku"
"chart_cz_542","542","Prodaný materiál"
"chart_cz_543","543","Dary"
"chart_cz_544","544","Smluvní pokuty a úroky z prodlení"
"chart_cz_545","545","Ostatní pokuty a penále"
"chart_cz_546","546","Odpis pohledávky"
"chart_cz_547","547","Mimořádné provozní náklady"
"chart_cz_548","548","Ostatní provozní náklady"
"chart_cz_549","549","Manka a škody z provozní činnosti"
"chart_cz_55","55","Odpisy, rezervy, komplexní náklady příštích období a opravné položky v provozní oblasti"
"chart_cz_551","551","Odpisy dlouhodobého nehmotného a hmotného majetku"
"chart_cz_552","552","Tvorba a zúčtování zákonných rezerv podle zvláštních právních předpisů"
"chart_cz_554","554","Tvorba a zúčtování ostatních rezerv"
"chart_cz_555","555","Tvorba a zúčtování komplexních nákladů příštích období"
"chart_cz_557","557","Zúčtování oprávky k oceňovacímu rozdílu k nabytému majetku"
"chart_cz_558","558","Tvorba a zúčtování zákonných opravných položek v provozní činnosti"
"chart_cz_559","559","Tvorba a zúčtování opravných položek v provozní činnosti"
"chart_cz_56","56","Finanční náklady"
"chart_cz_561","561","Prodané cenné papíry a podíly"
"chart_cz_562","562","Úroky"
"chart_cz_563","563","Kursové ztráty"
"chart_cz_564","564","Náklady na přecenění cenných papírů"
"chart_cz_565","565","Náklady z finančního majetku"
"chart_cz_566","566","Náklady z derivátových operací"
"chart_cz_567","567","Mimořádné finanční náklady"
"chart_cz_568","568","Ostatní finanční náklady"
"chart_cz_569","569","Manka a škody na finančním majetku"
"chart_cz_57","57","Rezervy a opravné položky ve finanční oblasti"
"chart_cz_574","574","Tvorba a zúčtování finančních rezerv"
"chart_cz_579","579","Tvorba a zúčtování opravných položek ve finanční činnosti"
"chart_cz_58","58","Změny stavu zásob vlastní činnosti a aktivace"
"chart_cz_581","581","Změna stavu nedokončené výroby"
"chart_cz_582","582","Změna stavu polotovarů"
"chart_cz_583","583","Změna stavu výrobků"
"chart_cz_584","584","Změna stavu mladých a ostatních zvířat"
"chart_cz_585","585","Aktivace materiálu a zboží"
"chart_cz_586","586","Aktivace vnitropodnikových služeb"
"chart_cz_587","587","Aktivace dlouhodobého nehmotného majetku"
"chart_cz_588","588","Aktivace dlouhodobého hmotného majetku"
"chart_cz_59","59","Daně z příjmů, převodové účty a rezerva na daň z příjmů"
"chart_cz_591","591","Daň z příjmů splatná"
"chart_cz_592","592","Daň z příjmů odložená"
"chart_cz_593","593","Tvorba a zúčtování rezervy na daň z příjmů"
"chart_cz_595","595","Dodatečné odvody daně z příjmů"
"chart_cz_596","596","Převod podílu na výsledku hospodaření společníkům v.o.s. a komplementářům k.s."
"chart_cz_597","597","Převod provozních nákladů"
"chart_cz_598","598","Převod finančních nákladů"
"chart_cz_599","599","Náklady hospodářských středisek"
"chart_cz_6","6","Výnosy"
"chart_cz_60","60","Tržby za vlastní výkony a zboží"
"chart_cz_601","601","Tržby za vlastní výrobky"
"chart_cz_602","602","Tržby z prodeje služeb"
"chart_cz_604","604","Tržby za zboží"
"chart_cz_64","64","Jiné provozní výnosy"
"chart_cz_641","641","Tržby z prodeje dlouhodobého nehmotného a hmotného majetku"
"chart_cz_642","642","Tržby z prodeje materiálu"
"chart_cz_644","644","Smluvní pokuty a úroky z prodlení"
"chart_cz_646","646","Výnosy z odepsaných pohledávek"
"chart_cz_647","647","Mimořádné provozní výnosy"
"chart_cz_648","648","Ostatní provizní výnosy"
"chart_cz_66","66","Finanční výnosy"
"chart_cz_661","661","Tržby z prodeje cenných papírů a podílů"
"chart_cz_662","662","Úroky"
"chart_cz_663","663","Kurzové zisky"
"chart_cz_664","664","Výnosy z přecenění cenných papírů"
"chart_cz_665","665","Výnosy z finančního majetku"
"chart_cz_666","666","Výnosy z derivátových operací"
"chart_cz_667","667","Mimořádné finanční výnosy"
"chart_cz_668","668","Ostatní finanční výnosy"
"chart_cz_69","69","Převodové účty"
"chart_cz_697","697","Převod provozních výnosů"
"chart_cz_698","698","Převod finančních výnosů"
"chart_cz_699","699","Výnosy hospodářských středisek"
"chart_cz_7","7","Závěrkové účty a podrozvahové účty"
"chart_cz_70","70","Rozvahové závěrkové účty"
"chart_cz_701","701","Počáteční účet rozvažný"
"chart_cz_702","702","Konečný účet rozvažný"
"chart_cz_71","71","Výsledovkový zisků a účet"
"chart_cz_710","710","Účet zisků a ztrát"

```

## File: data\template\account.tax-cz.csv

```csv
"id","name","description","invoice_label","tax_scope","type_tax_use","amount","invoice_repartition_line_ids/factor_percent","invoice_repartition_line_ids/repartition_type","invoice_repartition_line_ids/document_type","invoice_repartition_line_ids/factor","invoice_repartition_line_ids/account_id","invoice_repartition_line_ids/tag_ids","refund_repartition_line_ids/factor_percent","refund_repartition_line_ids/repartition_type","refund_repartition_line_ids/document_type","refund_repartition_line_ids/factor","refund_repartition_line_ids/account_id","refund_repartition_line_ids/tag_ids","price_include","amount_type","tax_group_id","name@cs","description@cs","invoice_label@cs"
"l10n_cz_other_transaction_deduction","0% Exempt","Other Taxable Fulfillment With Applicable Tax-exemption","Tax exemption according to VAT Law no. 235/2004","","sale","0","100","base","invoice","1","","+VAT 26","100","base","refund","1","","-VAT 26","False","percent","tax_group_vat_0","0 % Osvobozeno","Ostatní uskutečněná plnění s nárokem na odpočet daně","Osvobozeno od daně podle zákona o DPH č. 235/2004"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_12_purchase_goods_eu","12% EU G","12% - EU Purchase of Goods","12% VAT","consu","purchase","12","100","base","invoice","1","","+VAT 4 Base","100","base","refund","1","","-VAT 4 Base","False","percent","tax_group_vat_12","12% EU G","12% - Pořízení zboží z jiného státu EU","12% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343112","","100","tax","refund","1","chart_cz_343112","-VAT 4 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343212","-VAT 4 Tax","-100","tax","refund","-1","chart_cz_343212","","","","","","",""
"l10n_cz_supply_goods_eu","0% EU G","EU Sale of Goods","The buyer is obligated to fill in the VAT amounts and pay the tax.","consu","sale","0","100","base","invoice","1","","+VAT 20","100","base","refund","1","","-VAT 20","False","percent","tax_group_vat_0","0% EU G","Dodání zboží do jiného státu EU","Daň odvede zákazník"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_21_acquisition_goods_eu","21% EU G","21% - EU Purchase of Goods","21% VAT","consu","purchase","21","100","base","invoice","1","","+VAT 3 Base","100","base","refund","1","","-VAT 3 Base","False","percent","tax_group_vat_21","21% EU G","21% - Pořízení zboží z jiného státu EU","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","","100","tax","refund","1","chart_cz_343121","-VAT 3 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343221","-VAT 3 Tax","-100","tax","refund","-1","chart_cz_343221","","","","","","",""
"l10n_cz_12_receipt_service_person_eu","12% EU S","12% - EU Purchase of Services","12% VAT","service","purchase","12","100","base","invoice","1","","+VAT 6 Base","100","base","refund","1","","-VAT 6 Base","False","percent","tax_group_vat_12","12% EU S","12% - Přijetí služby od osoby reg. k dani v jiném EU státě","12% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343112","","100","tax","refund","1","chart_cz_343112","-VAT 6 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343212","-VAT 6 Tax","-100","tax","refund","-1","chart_cz_343212","","","","","","",""
"l10n_cz_21_receipt_service_person_eu","21% EU S","21% - EU Purchase of Services","21% VAT","service","purchase","21","100","base","invoice","1","","+VAT 5 Base","100","base","refund","1","","-VAT 5 Base","False","percent","tax_group_vat_21","21% EU S","21% - Přijetí služby od osoby reg. k dani v jiném EU státě","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","","100","tax","refund","1","chart_cz_343121","-VAT 5 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343221","-VAT 5 Tax","-100","tax","refund","-1","chart_cz_343221","","","","","","",""
"l10n_cz_supply_service_eu","0% EU S","EU Sale of Services","The buyer is obligated to fill in the VAT amounts and pay the tax.","service","sale","0","100","base","invoice","1","","+VAT 21","100","base","refund","1","","-VAT 21","False","percent","tax_group_vat_0","0% EU S","Poskytnutí služeb v jiném EU státě","Daň odvede zákazník"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","chart_cz_343221","","","","","","",""
"l10n_cz_21_import_goods","21% EX G","21% - Purchase of Goods Outside EU","21% VAT","consu","purchase","21","100","base","invoice","1","","+VAT 7 Base","100","base","refund","1","","-VAT 7 Base","False","percent","tax_group_vat_21","21% EX G","21% - Dovoz zboží","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","+VAT 7 Tax","100","tax","refund","1","chart_cz_343121","-VAT 7 Tax","","","","","",""
"l10n_cz_exempt_importation_goods","0% EX G","Import of Tax-exempt Goods","0% VAT","consu","purchase","0","100","base","invoice","1","","+VAT 32","100","base","refund","1","","-VAT 32","False","percent","tax_group_vat_0","0% EX G","Osvobozený dovoz zboží","0% DPH"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","-VAT 32","","","","","",""
"l10n_cz_investment_gold","0% Gold","Investment Gold","0% VAT","consu","sale","0","100","base","invoice","1","","+VAT 26","100","base","refund","1","","-VAT 26","False","percent","tax_group_vat_0","0% Zlato","Investiční zlato","0% DPH"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_12_import_goods","12% EX G","12% - Purchase of Goods Outside EU","12% VAT","consu","purchase","12","100","base","invoice","1","","+VAT 8 Base","100","base","refund","1","","-VAT 8 Base","False","percent","tax_group_vat_12","12% EX G","12% - Dovoz zboží","12% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343112","+VAT 8 Tax","100","tax","refund","1","chart_cz_343112","-VAT 8 Tax","","","","","",""
"l10n_cz_21_receipt_service_person_non_eu","21% EX S","21% - Purchase of Services Outside EU","21% VAT","service","purchase","21","100","base","invoice","1","","+VAT 12 Base","100","base","refund","1","","-VAT 12 Base","False","percent","tax_group_vat_21","21% EX S","21% - Přijetí služby od osoby mimo EU","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","","100","tax","refund","1","chart_cz_343121","-VAT 12 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343221","-VAT 12 Tax","-100","tax","refund","-1","chart_cz_343221","","","","","","",""
"l10n_cz_12_receipt_service_person_non_eu","12% EX S","12% - Purchase of Services Outside EU","12% VAT","service","purchase","12","100","base","invoice","1","","+VAT 13 Base","100","base","refund","1","","-VAT 13 Base","False","percent","tax_group_vat_12","12% EX S","12% - Přijetí služby od osoby mimo EU","12% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343112","+VAT 13 Tax","100","tax","refund","1","chart_cz_343112","-VAT 13 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343212","","-100","tax","refund","-1","chart_cz_343212","","","","","","",""
"l10n_cz_acquisition_transport","21% Vehicle","Acquisition of a New Vehicle","21% VAT","consu","purchase","21","100","base","invoice","1","","+VAT 9 Base","100","base","refund","1","","-VAT 9 Base","False","percent","tax_group_vat_21","21% Vozidlo","Pořízení nového dopravního prostředku","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","+VAT 9 Tax","100","tax","refund","1","chart_cz_343121","-VAT 9 Tax","","","","","",""
"l10n_cz_export_goods","0% EX G","Sale of Goods Outside EU","Tax exemption according to § 66 of VAT Law no. 235/2004","consu","sale","0","100","base","invoice","1","","+VAT 22","100","base","refund","1","","-VAT 22","False","percent","tax_group_vat_0","0% EX G","Vývoz zboží","Osvobozeno od daně podle § 66 zákona o DPH č. 235/2004"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_12_domestic_supplies","12%","12% - Local Sales","12% VAT","","sale","12","100","base","invoice","1","","+VAT 2 Base","100","base","refund","1","","-VAT 2 Base","False","percent","tax_group_vat_12","12%","12% - Plnění v tuzemsku","12% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343212","+VAT 2 Tax","100","tax","refund","1","chart_cz_343212","-VAT 2 Tax","","","","","",""
"l10n_cz_12_receipt_domestic_supplies","12%","12% - Local Purchase","12% VAT","","purchase","12","100","base","invoice","1","","+VAT 41 Base","100","base","refund","1","","-VAT 41 Base","False","percent","tax_group_vat_12","12%","12% - Přijetí plnění v tuzemsku","12% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343112","+VAT 41 Total","100","tax","refund","1","chart_cz_343112","-VAT 41 Total","","","","","",""
"l10n_cz_tax_reverse_charge_mode","0% RC","Reverse Charge","The buyer is obligated to fill in the VAT amounts and pay the tax.","","sale","0","100","base","invoice","1","","+VAT 25","100","base","refund","1","","-VAT 25","False","percent","tax_group_vat_21","0% RC","Režim přenesení daňové povinnosti","Daň odvede zákazník"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_21_tax_reverse_charge_scheme","21% RC","21% - Reverse Charge","The buyer is obligated to fill in the VAT amounts and pay the tax.","","purchase","21","100","base","invoice","1","","+VAT 10 Base","100","base","refund","1","","-VAT 10 Base","False","percent","tax_group_vat_21","21% RC","21% - Režim přenesení daňové povinnosti","Vystaveno zákazníkem"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","","100","tax","refund","1","chart_cz_343121","-VAT 10 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343221","-VAT 10 Tax","-100","tax","refund","-1","chart_cz_343221","","","","","","",""
"l10n_cz_21_other_supplies_chargeable","21% ND","21% - Other taxable fulfillment, where the tax is nondeductible","21% VAT","","purchase","21","100","base","invoice","1","","+VAT 12 Base","100","base","refund","1","","-VAT 12 Base","False","percent","tax_group_vat_21","21% ND","21% - Ostatní zdanitelná plnění, u kterých je povinnost přiznat daň","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","","100","tax","refund","1","chart_cz_343121","-VAT 12 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343221","-VAT 12 Tax","-100","tax","refund","-1","chart_cz_343221","","","","","","",""
"l10n_cz_12_other_supplies_obligation","12% ND","12% - Other taxable fulfillment, where the tax is nondeductible","12% VAT","","purchase","12","100","base","invoice","1","","+VAT 13 Base","100","base","refund","1","","-VAT 13 Base","False","percent","tax_group_vat_12","12% ND","12% - Ostatní zdanitelná plnění, u kterých je povinnost přiznat daň","12% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343112","","100","tax","refund","1","chart_cz_343112","-VAT 13 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343212","-VAT 13 Tax","-100","tax","refund","-1","chart_cz_343212","","","","","","",""
"l10n_cz_12_tax_reverse_charge_scheme","12% RC","12% - Reverse Charge","The buyer is obligated to fill in the VAT amounts and pay the tax.","","purchase","12","100","base","invoice","1","","+VAT 11 Base","100","base","refund","1","","-VAT 11 Base","False","percent","tax_group_vat_12","12% RC","12% - Režim přenesení daňové povinnosti","Vystaveno zákazníkem"
"","","","","","","","100","tax","invoice","1","chart_cz_343112","","100","tax","refund","1","chart_cz_343112","-VAT 11 Tax","","","","","",""
"","","","","","","","-100","tax","invoice","-1","chart_cz_343212","-VAT 11 Tax","-100","tax","refund","-1","chart_cz_343212","","","","","","",""
"l10n_cz_21_domestic_supplies","21% ","21% - Local Sales","21% VAT","","sale","21","100","base","invoice","1","","+VAT 1 Base","100","base","refund","1","","-VAT 1 Base","False","percent","tax_group_vat_21","21% ","21% - Plnění v tuzemsku","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343221","+VAT 1 Tax","100","tax","refund","1","chart_cz_343221","-VAT 1 Tax","","","","","",""
"l10n_cz_21_receipt_domestic_supplies","21% ","21% - Local Purchase","21% VAT","","purchase","21","100","base","invoice","1","","+VAT 40 Base","100","base","refund","1","","-VAT 40 Base","False","percent","tax_group_vat_21","21% ","21% - Přijetí plnění v tuzemsku","21% DPH"
"","","","","","","","100","tax","invoice","1","chart_cz_343121","+VAT 40 Total","100","tax","refund","1","chart_cz_343121","-VAT 40 Total","","","","","",""
"l10n_cz_special_mode","0% Regime","Special Tax Regime","Special Tax Regime - Tax exemption according to VAT Law no. 235/2004","","sale","0","100","base","invoice","1","","+VAT 26","100","base","refund","1","","-VAT 26","False","percent","tax_group_vat_0","0% Režim","Zvláštní režim","Zvláštní režim - Osvobozeno od daně podle zákona o DPH č. 235/2004"
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_not_included_vat","0% NA","VAT not applicable","","","sale","0","100","base","invoice","1","","","100","base","refund","1","","","False","percent","","0% NA","Nevstupuje do DPH",""
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_not_subject_vat","0% NA","VAT not applicable","","","purchase","0","100","base","invoice","1","","","100","base","refund","1","","","False","percent","","0% NA","Nevstupuje do DPH",""
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""
"l10n_cz_import_goods_tax_authority","0% EX G.Custom","Import of goods, where the tax administrator is the custom office","","","purchase","0","100","base","invoice","1","","+VAT 42 Base || +VAT 42 Total","100","base","refund","1","","-VAT 42 Base || -VAT 42 Total","False","percent","tax_group_vat_0","0% EX G.Zvyk","Dovoz zboží, kdy je správcem daně celní úřad",""
"","","","","","","","100","tax","invoice","1","","","100","tax","refund","1","","","","","","","",""

```

## File: data\template\account.tax.group-cz.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@cs"
"tax_group_vat_21","VAT 21%","base.cz","chart_cz_343001","chart_cz_343002","DPH 21%"
"tax_group_vat_15","VAT 15%","base.cz","chart_cz_343001","chart_cz_343002","DPH 15%"
"tax_group_vat_12","VAT 12%","base.cz","chart_cz_343001","chart_cz_343002","DPH 12%"
"tax_group_vat_0","VAT 0%","base.cz","chart_cz_343001","chart_cz_343002","DPH 0%"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'cz')], order="parent_path"):
        env['account.chart.template'].try_loading('cz', company)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class AccountMove(models.Model):
    _inherit = 'account.move'

    taxable_supply_date = fields.Date(default=fields.Date.today())

    @api.depends('taxable_supply_date')
    def _compute_date(self):
        super()._compute_date()
        for move in self:
            if move.country_code == 'CZ' and move.taxable_supply_date and move.state == 'draft':
                move.date = move.taxable_supply_date

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def _get_rate_date(self):
        # EXTENDS 'account'
        self.ensure_one()
        if self.move_id.country_code == 'CZ':
            return self.move_id.taxable_supply_date or self.move_id.date or fields.Date.context_today(self)
        return super()._get_rate_date()

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    trade_registry = fields.Char()

class BaseDocumentLayout(models.TransientModel):
    _inherit = 'base.document.layout'

    account_fiscal_country_id = fields.Many2one(related="company_id.account_fiscal_country_id")
    company_registry = fields.Char(related='company_id.company_registry')

```

## File: models\template_cz.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cz')
    def _get_cz_template_data(self):
        return {
            'code_digits': '6',
            'use_storno_accounting': True,
            'property_account_receivable_id': 'chart_cz_311000',
            'property_account_payable_id': 'chart_cz_321000',
            'property_account_expense_categ_id': 'chart_cz_504000',
            'property_account_income_categ_id': 'chart_cz_604000',
            'property_stock_account_input_categ_id': 'chart_cz_131000',
            'property_stock_account_output_categ_id': 'chart_cz_504000',
            'property_stock_valuation_account_id': 'chart_cz_132000',
        }

    @template('cz', 'res.company')
    def _get_cz_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.cz',
                'bank_account_code_prefix': '221',
                'cash_account_code_prefix': '211',
                'transfer_account_code_prefix': '261',
                'income_currency_exchange_account_id': 'chart_cz_663000',
                'expense_currency_exchange_account_id': 'chart_cz_563000',
                'account_journal_suspense_account_id': 'chart_cz_261000',
                'default_cash_difference_income_account_id': 'chart_cz_668000',
                'default_cash_difference_expense_account_id': 'chart_cz_568000',
                'account_default_pos_receivable_account_id': 'chart_cz_311001',
                'account_sale_tax_id': 'l10n_cz_21_domestic_supplies',
                'account_purchase_tax_id': 'l10n_cz_21_receipt_domestic_supplies',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_cz
from . import res_company
from . import account_move
from . import account_move_line

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_form_l10n_cz" model="ir.ui.view">
        <field name="name">account.move.form.l10n_cz</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='due_date']" position="after">
                <field name="taxable_supply_date" invisible="country_code != 'CZ'" readonly="state != 'draft'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <!-- add company id to partner details-->
        <xpath expr="//div[@id='partner_vat_address_not_same_as_shipping']" position="before">
            <div t-if="o.partner_id.company_registry and o.company_id.country_code == 'CZ'">
                Company ID: <span t-field="o.partner_id.company_registry"/>
            </div>
        </xpath>

        <xpath expr="//div[@id='partner_vat_address_same_as_shipping']" position="before">
            <div t-if="o.partner_id.company_registry and o.company_id.country_code == 'CZ'">
                Company ID: <span t-field="o.partner_id.company_registry"/>
            </div>
        </xpath>

        <xpath expr="//div[@id='partner_vat_no_shipping']" position="before">
            <div t-if="o.partner_id.company_registry and o.company_id.country_code == 'CZ'">
                Company ID: <span t-field="o.partner_id.company_registry"/>
            </div>
        </xpath>

        <!-- add taxable supply -->
        <xpath expr="//div[@name='due_date']" position="after">
            <div t-attf-class="#{'col-auto col-3 mw-100' if report_type != 'html' else 'col'} mb-2" t-if="o.taxable_supply_date and o.company_id.country_code == 'CZ'" name="taxable_supply_date">
                <strong>Taxable Supply:</strong>
                <p class="m-0" t-field="o.taxable_supply_date"/>
            </div>
        </xpath>

        <!-- add trade registry-->
        <xpath expr="//div[@name='comment']" position="after">
            <p t-if="o.company_id.trade_registry and o.company_id.country_code == 'CZ'" name="trade_registry">
                <strong>Trade registry: </strong><t t-out="o.company_id.trade_registry"/>
            </p>
        </xpath>
    </template>
</odoo>

```

## File: views\report_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="l10n_cz_external_layout_standard" inherit_id="web.external_layout_standard">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.company_registry and company.account_fiscal_country_id.code == 'CZ'">
                Company ID: <span t-field="company.company_registry"/>
            </li>
            <li t-if="company.vat and company.account_fiscal_country_id.code == 'CZ'">
                <t t-esc="company.country_id.vat_label or 'Tax ID'"/>:
                <span t-esc="company.vat"/>
            </li>
        </xpath>
    </template>

    <template id="l10n_cz_external_layout_bold" inherit_id="web.external_layout_bold">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.company_registry and company.account_fiscal_country_id.code == 'CZ'">
                Company ID: <span t-field="company.company_registry"/>
            </li>
            <li t-if="company.vat and company.account_fiscal_country_id.code == 'CZ'">
                <t t-esc="company.country_id.vat_label or 'Tax ID'"/>:
                <span t-esc="company.vat"/>
            </li>
        </xpath>
    </template>

    <template id="l10n_cz_external_layout_boxed" inherit_id="web.external_layout_boxed">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.company_registry and company.account_fiscal_country_id.code == 'CZ'">
                Company ID: <span t-field="company.company_registry"/>
            </li>
            <li t-if="company.vat and company.account_fiscal_country_id.code == 'CZ'">
                <t t-esc="company.country_id.vat_label or 'Tax ID'"/>:
                <span t-esc="company.vat"/>
            </li>
        </xpath>
    </template>

    <template id="l10n_cz_external_layout_striped" inherit_id="web.external_layout_striped">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.company_registry and company.account_fiscal_country_id.code == 'CZ'">
                Company ID: <span t-field="company.company_registry"/>
            </li>
            <li t-if="company.vat and company.account_fiscal_country_id.code == 'CZ'">
                <t t-esc="company.country_id.vat_label or 'Tax ID'"/>:
                <span t-esc="company.vat"/>
            </li>
        </xpath>
    </template>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form_inherit_l10n_ck" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_ck</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="model">res.company</field>
        <field name="arch" type="xml">
            <field name="currency_id" position="after">
                <field name="trade_registry" invisible="country_code != 'CZ'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_view_form_inherit_l10n_cz" model="ir.ui.view">
        <field name="name">res.partner.view.form.inherit.l10n.cz</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="before">
                <field name="company_registry" invisible="country_code != 'CZ'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

