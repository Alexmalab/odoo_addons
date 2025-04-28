# Odoo Module: l10n_zm_account

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
    "name": "Zambia - Accounting",
    "countries": ["zm"],
    "version": "1.0.0",
    "category": "Accounting/Localizations/Account Charts",
    "license": "LGPL-3",
    "description": """
This is the basic Zambian localization necessary to run Odoo in ZM:
================================================================================
    - Chart of Accounts
    - Taxes
    - Fiscal Positions
    - Default Settings
    """,
    "depends": [
        "account",
    ],
    "data": [
        "data/account_tax_report_data.xml",
        "views/report_invoice.xml",
    ],
    "demo": [
        "demo/demo_company.xml",
    ]
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="zm_tax_report" model="account.report">
        <field name="name">VAT Return</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.zm"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="zm_tax_report_no_vat" model="account.report.column">
                <field name="name">a. VAT Exclusive Value</field>
                <field name="expression_label">no_vat</field>
            </record>
            <record id="zm_tax_report_vat" model="account.report.column">
                <field name="name">b. VAT</field>
                <field name="expression_label">vat</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="zm_tax_report_title_sales" model="account.report.line">
                <field name="name">Sales</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="zm_tax_report_line_08" model="account.report.line">
                        <field name="name">8 - Standard rated local sales</field>
                        <field name="code">zm_vat_08</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_08_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_08_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_08_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_08_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_09" model="account.report.line">
                        <field name="name">9 - Local sales taxed at minimum taxable value (MTV)</field>
                        <field name="code">zm_vat_09</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_09_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_09_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_09_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_09_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_10" model="account.report.line">
                        <field name="name">10 - Disposal of capital assets</field>
                        <field name="code">zm_vat_10</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_10_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_10_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_10_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_10_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_11" model="account.report.line">
                        <field name="name">11 - Total standard Rated Outputs</field>
                        <field name="code">zm_vat_11</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_11_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_08.no_vat + zm_vat_09.no_vat + zm_vat_10.no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_11_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_08.vat + zm_vat_09.vat + zm_vat_10.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_12" model="account.report.line">
                        <field name="name">12 - Export of standard rated goods and services</field>
                        <field name="code">zm_vat_12</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_12_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_12_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_13" model="account.report.line">
                        <field name="name">13 - Export of zero-rated goods and services</field>
                        <field name="code">zm_vat_13</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_13_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_13_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_14" model="account.report.line">
                        <field name="name">14 - Other zero-rated outputs (e.g. supplies to donor funded projects or supplies to diplomatic missions etc.)</field>
                        <field name="code">zm_vat_14</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_14_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_14_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_15" model="account.report.line">
                        <field name="name">15 - Total zero-Rated Outputs</field>
                        <field name="code">zm_vat_15</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_15_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_12.no_vat + zm_vat_13.no_vat + zm_vat_14.no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_16" model="account.report.line">
                        <field name="name">16 - Total taxable sales</field>
                        <field name="code">zm_vat_16</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_16_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_11.no_vat + zm_vat_15.no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_17" model="account.report.line">
                        <field name="name">17 - Imported services (Reverse VAT)</field>
                        <field name="code">zm_vat_17</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_17_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_17_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_17_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_17_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_18" model="account.report.line">
                        <field name="name">18 - Total tax due on outputs</field>
                        <field name="code">zm_vat_18</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_18_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_11.vat + zm_vat_17.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_19" model="account.report.line">
                        <field name="name">19 - Local exempt sales</field>
                        <field name="code">zm_vat_19</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_19_novat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_19_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_20" model="account.report.line">
                        <field name="name">20 - Export of exempt goods and services</field>
                        <field name="code">zm_vat_20</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_20_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_20_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_21" model="account.report.line">
                        <field name="name">21 - Total exempt sales</field>
                        <field name="code">zm_vat_21</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_21_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_19.no_vat + zm_vat_20.no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_22" model="account.report.line">
                        <field name="name">22 - Total sales</field>
                        <field name="code">zm_vat_22</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_22_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_16.no_vat + zm_vat_21.no_vat</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="zm_tax_report_title_purchases" model="account.report.line">
                <field name="name">Purchases</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="zm_tax_report_line_24" model="account.report.line">
                        <field name="name">24 - Standard rated local purchases and allowable administrative expenses taxed at normal taxable value</field>
                        <field name="code">zm_vat_24</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_24_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_24_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_24_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_24_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_25" model="account.report.line">
                        <field name="name">25 - Local purchases taxed at minimum taxable value (MTV)</field>
                        <field name="code">zm_vat_25</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_25_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_25_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_25_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_25_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_26" model="account.report.line">
                        <field name="name">26 - Zero rated local purchases</field>
                        <field name="code">zm_vat_26</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_26_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_26_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_27" model="account.report.line">
                        <field name="name">27 - Standard rated imports</field>
                        <field name="code">zm_vat_27</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_27_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_27_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_27_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_27_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_28" model="account.report.line">
                        <field name="name">28 - Zero-rated imports</field>
                        <field name="code">zm_vat_28</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_28_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_28_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_29" model="account.report.line">
                        <field name="name">29 - Standard rated capital expenditure</field>
                        <field name="code">zm_vat_29</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_29_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_29_no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_29_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_29_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_30" model="account.report.line">
                        <field name="name">30 - Total taxable inputs</field>
                        <field name="code">zm_vat_30</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_30_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_24.no_vat + zm_vat_25.no_vat + zm_vat_26.no_vat + zm_vat_27.no_vat + zm_vat_28.no_vat + zm_vat_29.no_vat</field>
                            </record>
                            <record id="zm_tax_report_line_30_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_24.vat + zm_vat_25.vat + zm_vat_27.vat + zm_vat_29.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_31" model="account.report.line">
                        <field name="name">31 - Exempt local purchases</field>
                        <field name="code">zm_vat_31</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_31_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_31_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_32" model="account.report.line">
                        <field name="name">32 - Exempt imports</field>
                        <field name="code">zm_vat_32</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_32_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_32_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_33" model="account.report.line">
                        <field name="name">33 - Non-deductible purchases</field>
                        <field name="code">zm_vat_33</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_33_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_33_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_34" model="account.report.line">
                        <field name="name">34 - Purchases from unregistered suppliers</field>
                        <field name="code">zm_vat_34</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_34_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_34_no_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_35" model="account.report.line">
                        <field name="name">35 - Total purchases</field>
                        <field name="code">zm_vat_35</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_35_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_30.no_vat + zm_vat_31.no_vat + zm_vat_32.no_vat + zm_vat_33.no_vat + zm_vat_34.no_vat</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="zm_tax_report_title_input_tax" model="account.report.line">
                <field name="name">Calculation of Input Tax Allowed</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="zm_tax_report_line_37" model="account.report.line">
                        <field name="name">37 - Input Tax directly attributable to Taxable Sales</field>
                        <field name="code">zm_vat_37</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_37_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_38" model="account.report.line">
                        <field name="name">38 - Input Tax directly attributable to Exempt Sales</field>
                        <field name="code">zm_vat_38</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_38_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line39selection" model="account.report.line">
                        <field name="name">39 - Apportionment of input tax (Please indicate whether or not to use Apportionment of input tax.)</field>
                        <field name="code">zm_vat_39</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line39selection_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable</field>
                                <field name="figure_type">boolean</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_39a" model="account.report.line">
                        <field name="name">39a - First Method of apportionment (Please select which method of apportionment to use. Yes is 39a, No is 39b.)</field>
                        <field name="code">zm_vat_39a</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_39a_no_vat" model="account.report.expression">
                                <field name="label">no_vat</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="subformula">editable</field>
                                <field name="figure_type">boolean</field>
                            </record>
                            <record id="zm_tax_report_line_39a_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">( zm_vat_16.no_vat / zm_vat_22.no_vat ) * zm_vat_30.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_39b" model="account.report.line">
                        <field name="name">39b - Second Method of apportionment</field>
                        <field name="code">zm_vat_39b</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_39b_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">( zm_vat_37.vat + ( zm_vat_30.vat - zm_vat_37.vat - zm_vat_38.no_vat ) * ( zm_vat_16.no_vat / zm_vat_22.no_vat ))</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_40" model="account.report.line">
                        <field name="name">40 - Input Tax Credit Allowed</field>
                        <field name="code">zm_vat_40</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_40_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">(1 - zm_vat_39.vat) * zm_vat_37.vat + zm_vat_39.vat * ((zm_vat_39a.no_vat * zm_vat_39a.vat) + ((1 - zm_vat_39a.no_vat) * zm_vat_39b.vat))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="zm_tax_report_title_tax_due" model="account.report.line">
                <field name="name">Calculation of Tax Due</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="zm_tax_report_line_41" model="account.report.line">
                        <field name="name">41 - Total Output Tax</field>
                        <field name="code">zm_vat_41</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_41_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_18.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_42" model="account.report.line">
                        <field name="name">42 - Input Tax Allowed</field>
                        <field name="code">zm_vat_42</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_42_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_27.vat + zm_vat_40.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_43" model="account.report.line">
                        <field name="name">43 - Net VAT Payable/Claimable</field>
                        <field name="code">zm_vat_43</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_43_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_41.vat - zm_vat_42.vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_44" model="account.report.line">
                        <field name="name">44 - Withholding VAT Credit (From Sch.-XIII)</field>
                        <field name="code">zm_vat_44</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_44_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">zm_vat_44_vat</field>
                            </record>
                        </field>
                    </record>
                    <record id="zm_tax_report_line_45" model="account.report.line">
                        <field name="name">45 - Net VAT Payable/Claimable</field>
                        <field name="code">zm_vat_45</field>
                        <field name="expression_ids">
                            <record id="zm_tax_report_line_45_vat" model="account.report.expression">
                                <field name="label">vat</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">zm_vat_43.vat + zm_vat_44.vat</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-zm.csv

```csv
"id","name","code","account_type","reconcile"
"zm_account_1000000","Sales","1","income","False"
"zm_account_2000000","Cost of Sales / Purchases","2","expense_direct_cost","False"
"zm_account_2100000","Inventory Adjustment","21","expense_direct_cost","False"
"zm_account_2150000","Inventory Count Variance","215","expense_direct_cost","False"
"zm_account_2200000","Purchase Variance","22","expense_direct_cost","False"
"zm_account_2400000","Recovery Account","240000","expense_direct_cost","False"
"zm_account_2400010","Labour Cost Manufacturing Variance","240001","expense_direct_cost","False"
"zm_account_2400020","Direct Cost Manufacturing Variance","240002","expense_direct_cost","False"
"zm_account_2400030","Other Cost Manufacturing Variance","240003","expense_direct_cost","False"
"zm_account_2700000","Discount Received for Cash","27","income_other","False"
"zm_account_2750000","Interest Received","275","income_other","False"
"zm_account_2800000","Profit on Sale of Non Current Assets","28","income_other","False"
"zm_account_2850000","Bad Debts Recovered","285","income_other","False"
"zm_account_2900000","Foreign Exchange Gain","29","income_other","False"
"zm_account_3050000","Advertising & Promotions","305","expense","False"
"zm_account_3150000","Bad Debts","315","expense","False"
"zm_account_3200000","Bank Charges","32","expense","False"
"zm_account_3250000","Cleaning","325","expense","False"
"zm_account_3300000","Computer Expenses","33","expense","False"
"zm_account_3350000","Consulting Fees","335","expense","False"
"zm_account_3400000","Courier & Postage","34","expense","False"
"zm_account_3450000","Depreciation","345","expense","False"
"zm_account_3550000","Discount Allowed for Cash","355","expense","False"
"zm_account_3600000","Donations","36","expense","False"
"zm_account_3650000","Electricity & Water","365","expense","False"
"zm_account_3700000","Entertainment Expenses","37","expense","False"
"zm_account_3750000","Finance Charges","375","expense","False"
"zm_account_3800000","General Expenses","38","expense","False"
"zm_account_3850000","Insurance","385","expense","False"
"zm_account_3900000","Interest Paid","39","expense","False"
"zm_account_3950000","Leasing Charges","395","expense","False"
"zm_account_4000000","Legal Fees","4","expense","False"
"zm_account_4050000","Levies","405","expense","False"
"zm_account_4150000","Motor Vehicle Expenses","415","expense","False"
"zm_account_4200000","Printing & Stationery","42","expense","False"
"zm_account_4210000","Foreign Exchange Loss","421","expense","False"
"zm_account_4300000","Rent Paid","43","expense","False"
"zm_account_4350000","Repairs & Maintenance","435","expense","False"
"zm_account_4400000","Salaries & Wages","44","expense","False"
"zm_account_4450000","Staff Training","445","expense","False"
"zm_account_4500000","Staff Welfare","45","expense","False"
"zm_account_4550000","Subscriptions","455","expense","False"
"zm_account_4560000","License costs","456","expense","False"
"zm_account_4600000","Telephone & Fax","46","expense","False"
"zm_account_4650000","Travel & Accommodation","465","expense","False"
"zm_account_4700000","Loss on Sale of Non Current Assets","47","expense","False"
"zm_account_5200000","Retained Income / (Accumulated Loss)","52","liability_current","False"
"zm_account_5500000","Long Term Liabilities","55","liability_non_current","False"
"zm_account_5600000","Instalment Sale Creditors Other","56","liability_non_current","False"
"zm_account_6100000","Land & Buildings - Net Value","610000","asset_fixed","False"
"zm_account_6100010","Land & Buildings - At Cost","610001","asset_fixed","False"
"zm_account_6100020","Land & Buildings - Accumulated Depreciation","610002","asset_fixed","False"
"zm_account_6200000","Motor Vehicles - Net Value","620000","asset_fixed","False"
"zm_account_6200010","Motor Vehicles - At Cost","620001","asset_fixed","False"
"zm_account_6200020","Motor Vehicles - Accumulated Depreciation","620002","asset_fixed","False"
"zm_account_6250000","Computer Equipment - Net Value","625000","asset_fixed","False"
"zm_account_6250010","Computer Equipment - At Cost","625001","asset_fixed","False"
"zm_account_6250020","Computer Equipment - Accumulated Depreciation","625002","asset_fixed","False"
"zm_account_6300000","Office Equipment - Net Value","630000","asset_fixed","False"
"zm_account_6300010","Office Equipment - At Cost","630001","asset_fixed","False"
"zm_account_6300020","Office Equipment - Accumulated Depreciation","630002","asset_fixed","False"
"zm_account_6350000","Furniture & Fittings - Net value","635000","asset_fixed","False"
"zm_account_6350010","Furniture & Fittings - At Cost","635001","asset_fixed","False"
"zm_account_6350020","Furniture & Fittings - Accumulated Depreciation","635002","asset_fixed","False"
"zm_account_6600000","Other Fixed Assets - Net Value","660000","asset_fixed","False"
"zm_account_6600010","Other Fixed Assets - At Cost","660001","asset_fixed","False"
"zm_account_6600020","Other Fixed Assets - Accumulated Depreciation","660002","asset_fixed","False"
"zm_account_7000000","Goodwill / Other Intangible Assets","7","asset_fixed","False"
"zm_account_7100000","Investments","71","asset_current","False"
"zm_account_7700000","Inventory Control Account","77","liability_current","False"
"zm_account_8000000","Customers","8","asset_receivable","True"
"zm_account_8100000","Customes (POS)","81","asset_receivable","True"
"zm_account_8200000","Sundry Customers","82","asset_current","False"
"zm_account_8300000","VAT Purchase Tax Control Account","83","asset_current","False"
"zm_account_8300001","Withholding Tax on Sales","8300001","asset_current","False"
"zm_account_8300002","Withholding Tax on Sales - Transition Account","8300002","asset_current","False"
"zm_account_8310000","VAT Receivable - Closing Entry","831","asset_current","True"
"zm_account_8900000","Deferred Expenses","89","asset_current","True"
"zm_account_9000000","Suppliers","9","liability_payable","True"
"zm_account_9100000","GRN Accrual Account","91","liability_payable","True"
"zm_account_9200000","Sundry Suppliers","92","liability_current","False"
"zm_account_9400000","Provision for Future Expenses","940000","liability_current","False"
"zm_account_9400010","Provision for Insurance","940001","liability_current","False"
"zm_account_9400020","Provision for Salary Bonus","940002","liability_current","False"
"zm_account_9500000","VAT Sales Tax Control Account","95","liability_current","False"
"zm_account_9510000","Vat / Tax Provision Account","951","liability_payable","True"
"zm_account_9520000","VAT Payable - Closing Entry","952","liability_current","True"
"zm_account_9900000","Deferred Income","99","liability_current","True"
"zm_account_9999990","Undistributed Profits/Losses","999999","equity_unaffected","False"

```

## File: data\template\account.fiscal.position-zm.csv

```csv
"id","sequence","name","auto_apply","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"zm_fiscal_position_template_national","1","National","1","base.zm",,
"zm_fiscal_position_template_international","2","International","1",,"zm_tax_sale_16","zm_tax_sale_0_export_standard"
,,,,,"zm_tax_sale_0","zm_tax_sale_0_export"
,,,,,"zm_tax_sale_0_exempt","zm_tax_sale_0_export_exempt"
,,,,,"zm_tax_purchase_16","zm_tax_purchase_16_import_standard"
,,,,,"zm_tax_purchase_0","zm_tax_purchase_0_import"
,,,,,"zm_tax_purchase_0_exempt","zm_tax_purchase_0_import_exempt"

```

## File: data\template\account.group-zm.csv

```csv
"id","code_prefix_start","code_prefix_end","name"
"zm_group_1","1","","Sales income"
"zm_group_2","2","","Sales extra incomes and fees"
"zm_group_20","20","","Cost of Sales"
"zm_group_27","27","","Discount and Interests"
"zm_group_28","28","","Non Current Assets Income"
"zm_group_29","29","","Sundry Income"
"zm_group_3","3","","Accounting Fees"
"zm_group_30","30","","Accounting, Advertising and Promotion Costs"
"zm_group_31","31","","Bad Debts"
"zm_group_32","32","","Bank Charges and Cleaning"
"zm_group_33","33","","Computer Expenses and Consulting Fees"
"zm_group_34","34","","Courier, Postage and Depreciation"
"zm_group_35","35","","Cash Discount"
"zm_group_36","36","","Donations and Electricity"
"zm_group_37","37","","Enternainement and Finance Expenses"
"zm_group_38","38","","General Expenses and Insurance"
"zm_group_39","39","","Interests and Leasing Charges"
"zm_group_40","40","","Legal Fees and Levies"
"zm_group_41","41","","Motor Vehicle Expenses"
"zm_group_42","42","","Printing, Stationery and Foreing Exchange Gains and Losses"
"zm_group_43","43","","Rent Paid, Repirs and Maintenance"
"zm_group_440","440","","Salaries and Wages"
"zm_group_445","445","","Staff Training"
"zm_group_45","45","","Staff Wellfare and Subcriptions"
"zm_group_46","46","","Telephone, Fax, Travel and Accomodations"
"zm_group_52","52","","Retained Income"
"zm_group_55","55","","Long Term Liabilities"
"zm_group_56","56","","Instalment Sale Creditors Other"
"zm_group_61","61","","Land and Building"
"zm_group_62","62","","Motor Vehicles and Computer Equipement"
"zm_group_63","63","","Office Equipement and Furnitures and Fittings"
"zm_group_66","66","","Other Fixed Assets"
"zm_group_70","70","","Goodwill or Other Intangible Assets"
"zm_group_71","71","","Investments"
"zm_group_77","77","","Inventory Control Account"
"zm_group_80","80","","Customer Account"
"zm_group_81","81","","POS Cash Control"
"zm_group_82","82","","Sundry Customers"
"zm_group_83","83","","VAT Purchase Tax Control Account"
"zm_group_84","84","","Bank and Outstanding Receipts and Payments"
"zm_group_90","90","","Supplier Account"
"zm_group_91","91","","GRN Accrual Account"
"zm_group_92","92","","Sundry Suppliers"
"zm_group_94","94","","Provisions for Future Expenses, Insurance and Salary Bonuses"
"zm_group_95","95","","VAT Sales Control and Provisions Account"
"zm_group_9990","9990","","Suspense Account"
"zm_group_9999","9999","","Undistributed Profits and Losses"

```

## File: data\template\account.tax-zm.csv

```csv
"id","description","invoice_label","type_tax_use","name","active","amount_type","amount","tax_group_id","tax_exigibility","cash_basis_transition_account_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id"
"zm_tax_sale_16","Standard Rate","16%","sale","16%",,"percent","16.0","zm_tax_group_16",,,"base","invoice","+zm_vat_08_no_vat",
,,,,,,,,,,,"tax","invoice","+zm_vat_08_vat","zm_account_9500000"
,,,,,,,,,,,"base","refund","-zm_vat_08_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_08_vat","zm_account_9500000"
"zm_tax_sale_16_mtv","Local Sales Minimum Taxable Value Rate","16%","sale","16% MTV",,"percent","16.0","zm_tax_group_16",,,"base","invoice","+zm_vat_09_no_vat",
,,,,,,,,,,,"tax","invoice","+zm_vat_09_vat","zm_account_9500000"
,,,,,,,,,,,"base","refund","-zm_vat_09_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_09_vat","zm_account_9500000"
"zm_tax_sale_0_disposal","Disposal of Capital Assets Rate","0%","sale","0% Dispos","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_10_no_vat",
,,,,,,,,,,,"tax","invoice","+zm_vat_10_vat",
,,,,,,,,,,,"base","refund","-zm_vat_10_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_10_vat",
"zm_tax_sale_0_export_standard","Export Standard Rate","0%","sale","0% Std EX",,"percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_12_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_12_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_sale_0_export","Export Zero Rate","0%","sale","0% Zero EX","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_13_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_13_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_sale_0","Zero Rate","0%","sale","0%",,"percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_14_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_14_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_sale_0_exempt","Exempt Rate","0%","sale","0% Exmpt","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_19_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_19_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_sale_0_export_exempt","Export Exempt Rate","0%","sale","0% Exmpt EX","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_20_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_20_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_purchase_16","Standard Rate","16%","purchase","16%",,"percent","16.0","zm_tax_group_16",,,"base","invoice","+zm_vat_24_no_vat",
,,,,,,,,,,,"tax","invoice","+zm_vat_24_vat","zm_account_8300000"
,,,,,,,,,,,"base","refund","-zm_vat_24_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_24_vat","zm_account_8300000"
"zm_tax_purchase_16_mtv","Local Purchases Minimum Taxable Value Rate","16%","purchase","16% MTV",,"percent","16.0","zm_tax_group_16",,,"base","invoice","+zm_vat_25_no_vat",
,,,,,,,,,,,"tax","invoice","+zm_vat_25_vat","zm_account_8300000"
,,,,,,,,,,,"base","refund","-zm_vat_25_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_25_vat","zm_account_8300000"
"zm_tax_purchase_0","Zero Rate","0%","purchase","0%",,"percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_26_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_26_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_purchase_16_import_standard","Import Standard Rate","16%","purchase","16% EX",,"percent","16.0","zm_tax_group_16",,,"base","invoice","+zm_vat_27_no_vat",
,,,,,,,,,,,"tax","invoice","+zm_vat_27_vat",
,,,,,,,,,,,"base","refund","-zm_vat_27_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_27_vat",
"zm_tax_purchase_0_import","Import Zero Rate","0%","purchase","0% EX",,"percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_28_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_28_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_purchase_16_capex","Capital Expenditure Rate","16%","purchase","16% Capex","False","percent","16.0","zm_tax_group_16",,,"base","invoice","+zm_vat_29_no_vat",
,,,,,,,,,,,"tax","invoice","+zm_vat_29_vat","zm_account_8300000"
,,,,,,,,,,,"base","refund","-zm_vat_29_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_29_vat","zm_account_8300000"
"zm_tax_purchase_0_exempt","Exempt Rate","0%","purchase","0% Exmpt","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_31_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_31_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_purchase_0_import_exempt","Import Exempt Rate","0%","purchase","0% Exmpt EX","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_32_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_32_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_purchase_0_nondeductible","Non Deductible Rate","0%","purchase","0% NDED","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_33_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_33_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_purchase_0_unregistered","Unregistered Suppliers Rate","0%","purchase","0% UR","False","percent","0.0","zm_tax_group_0",,,"base","invoice","+zm_vat_34_no_vat",
,,,,,,,,,,,"tax","invoice",,
,,,,,,,,,,,"base","refund","-zm_vat_34_no_vat",
,,,,,,,,,,,"tax","refund",,
"zm_tax_sale_16_witholding","Withholding Tax on Sales","16%","sale","16% WH",,"percent","-16.0","zm_tax_group_16_wh","on_payment","zm_account_8300002","base","invoice",,
,,,,,,,,,,,"tax","invoice","+zm_vat_44_vat","zm_account_8300001"
,,,,,,,,,,,"base","refund","-zm_vat_08_no_vat",
,,,,,,,,,,,"tax","refund","-zm_vat_44_vat","zm_account_8300001"

```

## File: data\template\account.tax.group-zm.csv

```csv
id,name,country_id,"tax_receivable_account_id","tax_payable_account_id"
"zm_tax_group_0","VAT 0%","base.zm","zm_account_8310000","zm_account_9520000"
"zm_tax_group_16","VAT 16%","base.zm","zm_account_8310000","zm_account_9520000"
"zm_tax_group_16_wh","Withholding VAT","base.zm","zm_account_8310000","zm_account_9520000"

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_name_invoice_report(self):
        if self.company_id.account_fiscal_country_id.code == 'ZM':
            return 'l10n_zm_account.report_invoice_document'
        return super()._get_name_invoice_report()

```

## File: models\template_zm.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = "account.chart.template"

    @template('zm')
    def _get_zm_template_data(self):
        return {
            'code_digits': 7,
            'property_account_income_categ_id': 'zm_account_1000000',
            'property_account_expense_categ_id': 'zm_account_3800000',
            'property_account_receivable_id': 'zm_account_8000000',
            'property_account_payable_id': 'zm_account_9000000',
        }

    @template('zm', 'res.company')
    def _get_zm_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.zm',
                'bank_account_code_prefix': '840000',
                'cash_account_code_prefix': '840000',
                'transfer_account_code_prefix': '840000',
                'income_currency_exchange_account_id': 'zm_account_4210000',
                'expense_currency_exchange_account_id': 'zm_account_4210000',
                'account_default_pos_receivable_account_id': 'zm_account_8100000',
                'account_journal_early_pay_discount_loss_account_id': 'zm_account_3550000',
                'account_journal_early_pay_discount_gain_account_id': 'zm_account_2700000',
                'deferred_expense_account_id': 'zm_account_8900000',
                'deferred_revenue_account_id': 'zm_account_9900000',
                'account_sale_tax_id': 'zm_tax_sale_16',
                'account_purchase_tax_id': 'zm_tax_purchase_16',
            }
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import template_zm

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//div[@id='informations']" position="before">
            <t t-set="forced_vat" t-value="o.company_id.vat"/>
        </xpath>
        <xpath expr="//div[hasclass('page')]/h2" position="replace">
            <h2>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'">Fiscal Tax Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'draft'">Draft Tax Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'cancel'">Cancelled Tax Invoice</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'posted'">Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'draft'">Draft Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'cancel'">Cancelled Credit Note</span>
                <span t-elif="o.move_type == 'in_refund'">Vendor Credit Note</span>
                <span t-elif="o.move_type == 'in_invoice'">Vendor Bill</span>
                <span t-if="o.name != '/'" t-field="o.name"/>
            </h2>
        </xpath>
    </template>
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_zm_account.report_invoice_document'"
               t-call="l10n_zm_account.report_invoice_document"
               t-lang="lang"/>
        </xpath>
    </template>
</odoo>

```

