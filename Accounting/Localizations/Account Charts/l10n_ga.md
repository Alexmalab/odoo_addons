# Odoo Module: l10n_ga

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Gabon - Accounting",
    'countries': ['ga'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Gabon.
=================================================================

The Chart of Accounts is from SYSCOHADA.

    """,
    'depends': [
        'l10n_syscohada',
    ],
    'data': [
        'data/account_tax_report_data.xml'
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
    <record id="account_tax_report_ga" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ga"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_ga_base" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_ga_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_ga_vat" model="account.report.line">
                <field name="name">I. Value added tax</field>
                <field name="sequence">10</field>
                <field name="code">GA_VAT</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ga_sales" model="account.report.line">
                        <field name="name">1- Realized sales</field>
                        <field name="sequence">20</field>
                        <field name="code">GA_SALES</field>
                            <field name="children_ids">
                                <record id="account_tax_report_line_ga_sales_non_impo" model="account.report.line">
                                    <field name="name">1. Non-imposable operations</field>
                                    <field name="sequence">30</field>
                                    <field name="code">GA_NON_IMPO</field>
                                    <field name="expression_ids">
                                        <record id="account_tax_report_line_ga_sales_non_impo_base" model="account.report.expression">
                                            <field name="label">base</field>
                                            <field name="engine">tax_tags</field>
                                            <field name="formula">GA_non_impo</field>
                                        </record>
                                    </field>
                                </record>
                                <record id="account_tax_report_line_ga_sales_export" model="account.report.line">
                                    <field name="name">2. Exports</field>
                                    <field name="sequence">40</field>
                                    <field name="code">GA_EXPORT</field>
                                    <field name="expression_ids">
                                        <record id="account_tax_report_line_ga_sales_exports_base" model="account.report.expression">
                                            <field name="label">base</field>
                                            <field name="engine">tax_tags</field>
                                            <field name="formula">GA_export</field>
                                        </record>
                                    </field>
                                </record>
                                <record id="account_tax_report_line_ga_sales_taxable" model="account.report.line">
                                    <field name="name">3. Taxable operations</field>
                                    <field name="sequence">50</field>
                                    <field name="code">GA_TAXABLE</field>
                                    <field name="expression_ids">
                                        <record id="account_tax_report_line_ga_sales_taxable_base" model="account.report.expression">
                                            <field name="label">base</field>
                                            <field name="engine">aggregation</field>
                                            <field name="formula">GA_TAXABLE_18.base + GA_TAXABLE_10.base + GA_TAXABLE_5.base</field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_taxable_tax" model="account.report.expression">
                                            <field name="label">tax</field>
                                            <field name="engine">aggregation</field>
                                            <field name="formula">GA_TAXABLE_18.tax + GA_TAXABLE_10.tax + GA_TAXABLE_5.tax</field>
                                        </record>
                                    </field>
                                    <field name="children_ids">
                                        <record id="account_tax_report_line_ga_sales_taxable_18" model="account.report.line">
                                            <field name="name">at 18%</field>
                                            <field name="sequence">60</field>
                                            <field name="code">GA_TAXABLE_18</field>
                                            <field name="expression_ids">
                                                <record id="account_tax_report_line_ga_sales_taxable_18_base" model="account.report.expression">
                                                    <field name="label">base</field>
                                                    <field name="engine">tax_tags</field>
                                                    <field name="formula">GA_base_3_18</field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_taxable_18_tax" model="account.report.expression">
                                                    <field name="label">tax</field>
                                                    <field name="engine">tax_tags</field>
                                                    <field name="formula">GA_tax_3_18</field>
                                                </record>
                                            </field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_taxable_10" model="account.report.line">
                                            <field name="name">at 10%</field>
                                            <field name="sequence">70</field>
                                            <field name="code">GA_TAXABLE_10</field>
                                            <field name="expression_ids">
                                                <record id="account_tax_report_line_ga_sales_taxable_10_base" model="account.report.expression">
                                                    <field name="label">base</field>
                                                    <field name="engine">tax_tags</field>
                                                    <field name="formula">GA_base_3_10</field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_taxable_10_tax" model="account.report.expression">
                                                    <field name="label">tax</field>
                                                    <field name="engine">tax_tags</field>
                                                    <field name="formula">GA_tax_3_10</field>
                                                </record>
                                            </field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_taxable_5" model="account.report.line">
                                            <field name="name">at 5%</field>
                                            <field name="sequence">80</field>
                                            <field name="code">GA_TAXABLE_5</field>
                                            <field name="expression_ids">
                                                <record id="account_tax_report_line_ga_sales_taxable_5_base" model="account.report.expression">
                                                    <field name="label">base</field>
                                                    <field name="engine">tax_tags</field>
                                                    <field name="formula">GA_base_3_5</field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_taxable_5_tax" model="account.report.expression">
                                                    <field name="label">tax</field>
                                                    <field name="engine">tax_tags</field>
                                                    <field name="formula">GA_tax_3_5</field>
                                                </record>
                                            </field>
                                        </record>
                                    </field>
                                </record>
                                <record id="account_tax_report_line_ga_sales_additional_taxable" model="account.report.line">
                                    <field name="name">5. Additional taxable operations</field>
                                    <field name="sequence">100</field>
                                    <field name="code">GA_ADDITIONAL_TAXABLE</field>
                                    <field name="expression_ids">
                                        <record id="account_tax_report_line_ga_sales_additional_taxable_base" model="account.report.expression">
                                            <field name="label">base</field>
                                            <field name="engine">aggregation</field>
                                            <field name="formula">GA_ADDITIONAL_TAXABLE_18.base + GA_ADDITIONAL_TAXABLE_10.base + GA_ADDITIONAL_TAXABLE_5.base</field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_additional_taxable_tax" model="account.report.expression">
                                            <field name="label">tax</field>
                                            <field name="engine">aggregation</field>
                                            <field name="formula">GA_ADDITIONAL_TAXABLE_18.tax + GA_ADDITIONAL_TAXABLE_10.tax + GA_ADDITIONAL_TAXABLE_5.tax</field>
                                        </record>
                                    </field>
                                    <field name="children_ids">
                                        <record id="account_tax_report_line_ga_sales_additional_taxable_18" model="account.report.line">
                                            <field name="name">at 18%</field>
                                            <field name="sequence">110</field>
                                            <field name="code">GA_ADDITIONAL_TAXABLE_18</field>
                                            <field name="expression_ids">
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_18_base" model="account.report.expression">
                                                    <field name="label">base</field>
                                                    <field name="engine">aggregation</field>
                                                    <field name="formula">GA_TAXABLE_STATE_18.base + GA_TAXABLE_OTHER_18.base</field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_18_tax" model="account.report.expression">
                                                    <field name="label">tax</field>
                                                    <field name="engine">aggregation</field>
                                                    <field name="formula">GA_TAXABLE_STATE_18.tax + GA_TAXABLE_OTHER_18.tax</field>
                                                </record>
                                            </field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_additional_taxable_10" model="account.report.line">
                                            <field name="name">at 10%</field>
                                            <field name="sequence">120</field>
                                            <field name="code">GA_ADDITIONAL_TAXABLE_10</field>
                                            <field name="expression_ids">
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_10_base" model="account.report.expression">
                                                    <field name="label">base</field>
                                                    <field name="engine">aggregation</field>
                                                    <field name="formula">GA_TAXABLE_STATE_10.base + GA_TAXABLE_OTHER_10.base</field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_10_tax" model="account.report.expression">
                                                    <field name="label">tax</field>
                                                    <field name="engine">aggregation</field>
                                                    <field name="formula">GA_TAXABLE_STATE_10.tax + GA_TAXABLE_OTHER_10.tax</field>
                                                </record>
                                            </field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_additional_taxable_5" model="account.report.line">
                                            <field name="name">at 5%</field>
                                            <field name="sequence">130</field>
                                            <field name="code">GA_ADDITIONAL_TAXABLE_5</field>
                                            <field name="expression_ids">
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_5_base" model="account.report.expression">
                                                    <field name="label">base</field>
                                                    <field name="engine">aggregation</field>
                                                    <field name="formula">GA_TAXABLE_STATE_5.base + GA_TAXABLE_OTHER_5.base</field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_5_tax" model="account.report.expression">
                                                    <field name="label">tax</field>
                                                    <field name="engine">aggregation</field>
                                                    <field name="formula">GA_TAXABLE_STATE_5.tax + GA_TAXABLE_OTHER_5.tax</field>
                                                </record>
                                            </field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_taxable_state" model="account.report.line">
                                            <field name="name">5a. Taxable operations with the State</field>
                                            <field name="sequence">140</field>
                                            <field name="code">GA_TAXABLE_STATE</field>
                                            <field name="children_ids">
                                                <record id="account_tax_report_line_ga_sales_taxable_state_18" model="account.report.line">
                                                    <field name="name">at 18%</field>
                                                    <field name="sequence">150</field>
                                                    <field name="code">GA_TAXABLE_STATE_18</field>
                                                    <field name="expression_ids">
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_state_18_base" model="account.report.expression">
                                                            <field name="label">base</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_base_5a_18</field>
                                                        </record>
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_state_18_tax" model="account.report.expression">
                                                            <field name="label">tax</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_tax_5a_18</field>
                                                        </record>
                                                    </field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_taxable_state_10" model="account.report.line">
                                                    <field name="name">at 10%</field>
                                                    <field name="sequence">160</field>
                                                    <field name="code">GA_TAXABLE_STATE_10</field>
                                                    <field name="expression_ids">
                                                        <record id="account_tax_report_line_ga_sales_taxable_state_10_base" model="account.report.expression">
                                                            <field name="label">base</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_base_5a_10</field>
                                                        </record>
                                                        <record id="account_tax_report_line_ga_sales_taxable_state_10_tax" model="account.report.expression">
                                                            <field name="label">tax</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_tax_5a_10</field>
                                                        </record>
                                                    </field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_taxable_state_5" model="account.report.line">
                                                    <field name="name">at 5%</field>
                                                    <field name="sequence">170</field>
                                                    <field name="code">GA_TAXABLE_STATE_5</field>
                                                    <field name="expression_ids">
                                                        <record id="account_tax_report_line_ga_sales_taxable_state_5_base" model="account.report.expression">
                                                            <field name="label">base</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_base_5a_5</field>
                                                        </record>
                                                        <record id="account_tax_report_line_ga_sales_taxable_state_5_tax" model="account.report.expression">
                                                            <field name="label">tax</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_tax_5a_5</field>
                                                        </record>
                                                    </field>
                                                </record>
                                            </field>
                                        </record>
                                        <record id="account_tax_report_line_ga_sales_additional_taxable_other" model="account.report.line">
                                            <field name="name">5b. Other taxable operations</field>
                                            <field name="sequence">180</field>
                                            <field name="code">GA_TAXABLE_OTHER</field>
                                            <field name="children_ids">
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_other_18" model="account.report.line">
                                                    <field name="name">at 18%</field>
                                                    <field name="sequence">190</field>
                                                    <field name="code">GA_TAXABLE_OTHER_18</field>
                                                    <field name="expression_ids">
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_other_18_base" model="account.report.expression">
                                                            <field name="label">base</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_base_5b_18</field>
                                                        </record>
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_other_18_tax" model="account.report.expression">
                                                            <field name="label">tax</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_tax_5b_18</field>
                                                        </record>
                                                    </field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_other_10" model="account.report.line">
                                                    <field name="name">at 10%</field>
                                                    <field name="sequence">200</field>
                                                    <field name="code">GA_TAXABLE_OTHER_10</field>
                                                    <field name="expression_ids">
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_other_10_base" model="account.report.expression">
                                                            <field name="label">base</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_base_5b_10</field>
                                                        </record>
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_other_10_tax" model="account.report.expression">
                                                            <field name="label">tax</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_tax_5b_10</field>
                                                        </record>
                                                    </field>
                                                </record>
                                                <record id="account_tax_report_line_ga_sales_additional_taxable_other_5" model="account.report.line">
                                                    <field name="name">at 5%</field>
                                                    <field name="sequence">210</field>
                                                    <field name="code">GA_TAXABLE_OTHER_5</field>
                                                    <field name="expression_ids">
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_other_5_base" model="account.report.expression">
                                                            <field name="label">base</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_base_5b_5</field>
                                                        </record>
                                                        <record id="account_tax_report_line_ga_sales_additional_taxable_other_5_tax" model="account.report.expression">
                                                            <field name="label">tax</field>
                                                            <field name="engine">tax_tags</field>
                                                            <field name="formula">GA_tax_5b_5</field>
                                                        </record>
                                                    </field>
                                                </record>
                                            </field>
                                        </record>
                                    </field>
                                </record>
                                <record id="account_tax_report_line_ga_sales_total" model="account.report.line">
                                    <field name="name">4. Total of operations (line 1+2+3+5)</field>
                                    <field name="sequence">90</field>
                                    <field name="code">GA_TOTAL</field>
                                    <field name="expression_ids">
                                        <record id="account_tax_report_line_ga_sales_total_base" model="account.report.expression">
                                            <field name="label">base</field>
                                            <field name="engine">aggregation</field>
                                            <field name="formula">GA_NON_IMPO.base + GA_EXPORT.base + GA_TAXABLE.base + GA_ADDITIONAL_TAXABLE.base</field>
                                        </record>
                                    </field>
                                </record>
                                <record id="account_tax_report_line_ga_sales_gross" model="account.report.line">
                                    <field name="name">6. Gross VAT (line 3+5)</field>
                                    <field name="sequence">220</field>
                                    <field name="code">GA_GROSS</field>
                                    <field name="expression_ids">
                                        <record id="account_tax_report_line_ga_sales_gross_tax" model="account.report.expression">
                                            <field name="label">tax</field>
                                            <field name="engine">aggregation</field>
                                            <field name="formula">GA_TAXABLE.tax + GA_ADDITIONAL_TAXABLE.tax</field>
                                        </record>
                                    </field>
                                </record>
                            </field>
                    </record>
                    <record id="account_tax_report_line_ga_deduction" model="account.report.line">
                        <field name="name">2- Deductions</field>
                        <field name="sequence">230</field>
                        <field name="code">GA_DEDUCTION</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_ga_deduction_goods_service" model="account.report.line">
                                <field name="name">Deductions on goods and services</field>
                                <field name="sequence">240</field>
                                <field name="code">GA_DEDUCTION_GOODS_SERVICES</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_ga_deduction_goods_service_importation" model="account.report.line">
                                        <field name="name">7a. on importation</field>
                                        <field name="sequence">250</field>
                                        <field name="code">GA_DEDU_GOODS_IMPORTATION</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_ga_deduction_goods_service_importation_18" model="account.report.line">
                                                <field name="name">at 18%</field>
                                                <field name="sequence">260</field>
                                                <field name="code">GA_DEDU_GOODS_IMPORTATION_18</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_goods_service_importation_18_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_7a_18</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_goods_service_importation_10" model="account.report.line">
                                                <field name="name">at 10%</field>
                                                <field name="sequence">270</field>
                                                <field name="code">GA_DEDU_GOODS_IMPORTATION_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_goods_service_importation_10_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_7a_10</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_goods_service_importation_5" model="account.report.line">
                                                <field name="name">at 5%</field>
                                                <field name="sequence">280</field>
                                                <field name="code">GA_DEDU_GOODS_IMPORTATION_5</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_goods_service_importation_5_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_7a_5</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_deduction_goods_service_domestic" model="account.report.line">
                                        <field name="name">7b. on domestic markets</field>
                                        <field name="sequence">290</field>
                                        <field name="code">GA_DEDU_GOODS_DOMESTIC</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_ga_deduction_goods_service_domestic_18" model="account.report.line">
                                                <field name="name">at 18%</field>
                                                <field name="sequence">300</field>
                                                <field name="code">GA_DEDU_GOODS_DOMESTIC_18</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_goods_service_domestic_18_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_7b_18</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_goods_service_domestic_10" model="account.report.line">
                                                <field name="name">at 10%</field>
                                                <field name="sequence">310</field>
                                                <field name="code">GA_DEDU_GOODS_DOMESTIC_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_goods_service_domestic_10_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_7b_10</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_goods_service_domestic_5" model="account.report.line">
                                                <field name="name">at 5%</field>
                                                <field name="sequence">320</field>
                                                <field name="code">GA_DEDU_GOODS_DOMESTIC_5</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_goods_service_domestic_5_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_7b_5</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_deduction_goods_service_total" model="account.report.line">
                                        <field name="name">7. Total (7a+7b)</field>
                                        <field name="sequence">330</field>
                                        <field name="code">GA_DEDU_GOODS_TOTAL</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_deduction_goods_service_total_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">GA_DEDU_GOODS_IMPORTATION_18.tax + GA_DEDU_GOODS_IMPORTATION_10.tax + GA_DEDU_GOODS_IMPORTATION_5.tax + GA_DEDU_GOODS_DOMESTIC_18.tax + GA_DEDU_GOODS_DOMESTIC_10.tax + GA_DEDU_GOODS_DOMESTIC_5.tax</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_deduction_assets" model="account.report.line">
                                <field name="name">Deductions on assets</field>
                                <field name="sequence">340</field>
                                <field name="code">GA_DEDUCTION_ASSETS</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_ga_deduction_assets_importation" model="account.report.line">
                                        <field name="name">8a. on importation</field>
                                        <field name="sequence">350</field>
                                        <field name="code">GA_DEDU_ASSETS_IMPORTATION</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_ga_deduction_assets_importation_18" model="account.report.line">
                                                <field name="name">at 18%</field>
                                                <field name="sequence">360</field>
                                                <field name="code">GA_DEDU_ASSETS_IMPORTATION_18</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_assets_importation_18_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_18</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_assets_importation_10" model="account.report.line">
                                                <field name="name">at 10%</field>
                                                <field name="sequence">370</field>
                                                <field name="code">GA_DEDU_ASSETS_IMPORTATION_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_assets_importation_10_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_10</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_assets_importation_5" model="account.report.line">
                                                <field name="name">at 5%</field>
                                                <field name="sequence">380</field>
                                                <field name="code">GA_DEDU_ASSETS_IMPORTATION_5</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_assets_importation_5_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_5</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_deduction_assets_domestic" model="account.report.line">
                                        <field name="name">8b. on domestic markets</field>
                                        <field name="sequence">390</field>
                                        <field name="code">GA_DEDU_ASSETS_DOMESTIC</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_ga_deduction_assets_domestic_18" model="account.report.line">
                                                <field name="name">at 18%</field>
                                                <field name="sequence">400</field>
                                                <field name="code">GA_DEDU_ASSETS_DOMESTIC_18</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_assets_domestic_18_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8b_18</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_assets_domestic_10" model="account.report.line">
                                                <field name="name">at 10%</field>
                                                <field name="sequence">410</field>
                                                <field name="code">GA_DEDU_ASSETS_DOMESTIC_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_assets_domestic_10_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8b_10</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_deduction_assets_domestic_5" model="account.report.line">
                                                <field name="name">at 5%</field>
                                                <field name="sequence">420</field>
                                                <field name="code">GA_DEDU_ASSETS_DOMESTIC_5</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_deduction_assets_domestic_5_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8b_5</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_deduction_assets_total" model="account.report.line">
                                        <field name="name">8. Total (8a+8b)</field>
                                        <field name="sequence">430</field>
                                        <field name="code">GA_DEDU_ASSETS_TOTAL</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_deduction_assets_total_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">GA_DEDU_ASSETS_IMPORTATION_18.tax + GA_DEDU_ASSETS_IMPORTATION_10.tax + GA_DEDU_ASSETS_IMPORTATION_5.tax + GA_DEDU_ASSETS_DOMESTIC_18.tax + GA_DEDU_ASSETS_DOMESTIC_10.tax + GA_DEDU_ASSETS_DOMESTIC_5.tax</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_regularisation" model="account.report.line">
                                <field name="name">Regularisations</field>
                                <field name="sequence">440</field>
                                <field name="code">GA_REGULARISATION</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_ga_regularisation_withholding" model="account.report.line">
                                        <field name="name">9a. State withholding</field>
                                        <field name="sequence">450</field>
                                        <field name="code">GA_REGULARISATION_WITHHOLDING</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_ga_regularisation_withholding_18" model="account.report.line">
                                                <field name="name">at 18%</field>
                                                <field name="sequence">460</field>
                                                <field name="code">GA_REGULARISATION_WITHHOLDING_18</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_withholding_18_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_18</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_regularisation_withholding_10" model="account.report.line">
                                                <field name="name">at 10%</field>
                                                <field name="sequence">470</field>
                                                <field name="code">GA_REGULARISATION_WITHHOLDING_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_withholding_10_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_10</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_regularisation_withholding_5" model="account.report.line">
                                                <field name="name">at 5%</field>
                                                <field name="sequence">480</field>
                                                <field name="code">GA_REGULARISATION_WITHHOLDING_5</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_withholding_5_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_5</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_regularisation_exemptions" model="account.report.line">
                                        <field name="name">9b. VAT exemptions</field>
                                        <field name="sequence">490</field>
                                        <field name="code">GA_REGULARISATION_EXEMPTIONS</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_ga_regularisation_exemptions_18" model="account.report.line">
                                                <field name="name">at 18%</field>
                                                <field name="sequence">500</field>
                                                <field name="code">GA_REGULARISATION_EXEMPTIONS_18</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_exemptions_18_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8b_18</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_regularisation_exemptions_10" model="account.report.line">
                                                <field name="name">at 10%</field>
                                                <field name="sequence">510</field>
                                                <field name="code">GA_REGULARISATION_EXEMPTIONS_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_exemptions_10_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8b_10</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_regularisation_exemptions_5" model="account.report.line">
                                                <field name="name">at 5%</field>
                                                <field name="sequence">520</field>
                                                <field name="code">GA_REGULARISATION_EXEMPTIONS_5</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_domestic_5_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8b_5</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_regularisation_additional_deduction" model="account.report.line">
                                        <field name="name">9c. Additional deduction</field>
                                        <field name="sequence">530</field>
                                        <field name="code">GA_REGULARISATION_ADDITIONAL_DEDU</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_ga_regularisation_additional_deduction_18" model="account.report.line">
                                                <field name="name">at 18%</field>
                                                <field name="sequence">540</field>
                                                <field name="code">GA_REGULARISATION_ADDITIONAL_DEDU_18</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_additional_deduction_18_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_18</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_regularisation_additional_deduction_10" model="account.report.line">
                                                <field name="name">at 10%</field>
                                                <field name="sequence">550</field>
                                                <field name="code">GA_REGULARISATION_ADDITIONAL_DEDU_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_additional_deduction_10_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_10</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_ga_regularisation_additional_deduction_5" model="account.report.line">
                                                <field name="name">at 5%</field>
                                                <field name="sequence">560</field>
                                                <field name="code">GA_REGULARISATION_ADDITIONAL_DEDU_5</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_ga_regularisation_additional_deduction_5_tax" model="account.report.expression">
                                                        <field name="label">tax</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">GA_8a_5</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_regularisation_total" model="account.report.line">
                                        <field name="name">9. Total (9a+9b+9c)</field>
                                        <field name="sequence">570</field>
                                        <field name="code">GA_REGULARISATION_TOTAL</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_regularisation_total_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">GA_REGULARISATION_WITHHOLDING_18.tax + GA_REGULARISATION_WITHHOLDING_10.tax + GA_REGULARISATION_WITHHOLDING_5.tax + GA_REGULARISATION_EXEMPTIONS_18.tax + GA_REGULARISATION_EXEMPTIONS_10.tax + GA_REGULARISATION_EXEMPTIONS_5.tax + GA_REGULARISATION_ADDITIONAL_DEDU_18.tax + GA_REGULARISATION_ADDITIONAL_DEDU_10.tax + GA_REGULARISATION_ADDITIONAL_DEDU_5.tax</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_refund_requested" model="account.report.line">
                                        <field name="name">10a. refund requested the previous month</field>
                                        <field name="sequence">580</field>
                                        <field name="code">GA_REFUND_REQUESTED</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_refund_requested_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">GA_10a</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_repayment" model="account.report.line">
                                        <field name="name">10b. repayment to be made</field>
                                        <field name="sequence">590</field>
                                        <field name="code">GA_REPAYMENTS</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_repayment_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">GA_10b</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_total_payments" model="account.report.line">
                                        <field name="name">10. Total (10a+10b)</field>
                                        <field name="sequence">600</field>
                                        <field name="code">GA_TOTAL_PAYMENTS</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_total_payments_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">GA_REFUND_REQUESTED.tax + GA_REPAYMENTS.tax</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_report_credit" model="account.report.line">
                                <field name="name">11. Credit carried over from previous month</field>
                                <field name="sequence">610</field>
                                <field name="code">GA_REPORT_CREDIT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ga_report_credit_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">GA_REPORT_CREDIT._applied_carryover_tax</field>
                                    </record>
                                    <record id="account_tax_report_line_ga_report_credit_tax_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_deduction_total" model="account.report.line">
                                <field name="name">12. Total (7+8+9-10+11)</field>
                                <field name="sequence">620</field>
                                <field name="code">GA_DEDU_TOTAL</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ga_deduction_total_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">GA_DEDU_GOODS_TOTAL.tax + GA_DEDU_ASSETS_TOTAL.tax + GA_REGULARISATION_TOTAL.tax - GA_TOTAL_PAYMENTS.tax + GA_REPORT_CREDIT.tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ga_settlement" model="account.report.line">
                        <field name="name">3- Settlement of VAT payable</field>
                        <field name="sequence">630</field>
                        <field name="code">GA_SETTLEMENT</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_ga_gross_payable" model="account.report.line">
                                <field name="name">13. Gross VAT</field>
                                <field name="sequence">640</field>
                                <field name="code">GA_GROSS_PAYABLE</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_ga_gross_18" model="account.report.line">
                                        <field name="name">at 18%</field>
                                        <field name="sequence">650</field>
                                        <field name="code">GA_GROSS_18</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_gross_18_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">GA_TAXABLE_18.tax + GA_ADDITIONAL_TAXABLE_18.tax</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_gross_10" model="account.report.line">
                                        <field name="name">at 10%</field>
                                        <field name="sequence">660</field>
                                        <field name="code">GA_GROSS_10</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_gross_10_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">GA_TAXABLE_10.tax + GA_ADDITIONAL_TAXABLE_10.tax</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_ga_gross_5" model="account.report.line">
                                        <field name="name">at 5%</field>
                                        <field name="sequence">670</field>
                                        <field name="code">GA_GROSS_5</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_ga_gross_5_tax" model="account.report.expression">
                                                <field name="label">tax</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">GA_TAXABLE_5.tax + GA_ADDITIONAL_TAXABLE_5.tax</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_total_gross" model="account.report.line">
                                <field name="name">14. Total gross VAT</field>
                                <field name="sequence">680</field>
                                <field name="code">GA_TOTAL_GROSS</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ga_total_gross_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">GA_GROSS_18.tax + GA_GROSS_10.tax + GA_GROSS_5.tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_deductible" model="account.report.line">
                                <field name="name">15. Deductible VAT</field>
                                <field name="sequence">690</field>
                                <field name="code">GA_DEDUCTIBLE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ga_deductible_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">GA_DEDU_TOTAL.tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_net_to_pay" model="account.report.line">
                                <field name="name">16. Net VAT to pay (14-15)</field>
                                <field name="sequence">700</field>
                                <field name="code">GA_NET_TO_PAY</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ga_net_to_pay_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">GA_TOTAL_GROSS.tax - GA_DEDUCTIBLE.tax</field>
                                        <field name="subformula">if_above(XAF(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ga_credit_to_report" model="account.report.line">
                                <field name="name">17. Credit to report (15-14)</field>
                                <field name="sequence">710</field>
                                <field name="code">GA_CREDIT_TO_REPORT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ga_credit_to_report_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">GA_DEDUCTIBLE.tax - GA_TOTAL_GROSS.tax</field>
                                        <field name="subformula">if_above(XAF(0))</field>
                                        <field name="carryover_target" eval="False"/>
                                    </record>
                                    <record id="account_tax_report_line_ga_credit_to_report_carryover" model="account.report.expression">
                                        <field name="label">_carryover_tax</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">GA_CREDIT_TO_REPORT.tax</field>
                                        <field name="carryover_target">GA_REPORT_CREDIT._applied_carryover_tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="account_tax_report_line_ga_css" model="account.report.line">
                <field name="name">II. Special solidarity contribution</field>
                <field name="sequence">720</field>
                <field name="code">GA_CSS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ga_taxable" model="account.report.line">
                        <field name="name">1. Taxable operations</field>
                        <field name="sequence">730</field>
                        <field name="code">GA_CSS_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ga_css_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">GA_css</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ga_css_taxable_state" model="account.report.line">
                        <field name="name">2. Taxable operations with state</field>
                        <field name="sequence">740</field>
                        <field name="code">GA_CSS_TAXABLE_STATE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ga_css_taxable_state_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">GA_css_state</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ga_css_taxable_other" model="account.report.line">
                        <field name="name">3. Other taxable operations</field>
                        <field name="sequence">750</field>
                        <field name="code">GA_CSS_TAXABLE_OTHER</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ga_css_taxable_other_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">GA_css_other</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ga_css_base_taxable" model="account.report.line">
                        <field name="name">4. Taxable base</field>
                        <field name="sequence">760</field>
                        <field name="code">GA_CSS_TAXABLE_BASE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ga_css_base_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GA_CSS_TAXABLE.base + GA_CSS_TAXABLE_STATE.base + GA_CSS_TAXABLE_OTHER.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ga_css_rate" model="account.report.line">
                        <field name="name">5. Rate: 1%</field>
                        <field name="sequence">770</field>
                        <field name="code">GA_CSS_RATE</field>
                    </record>
                    <record id="account_tax_report_line_ga_css_due" model="account.report.line">
                        <field name="name">6. Amount due</field>
                        <field name="sequence">780</field>
                        <field name="code">GA_CSS_DUE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ga_css_due_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">GA_css_tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-ga.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.ga","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_19","tva_export_0"
"","","","","","","","tva_sale_11","tva_export_0"
"","","","","","","","tva_sale_6","tva_export_0"
"","","","","","","","tva_purchase_19","tva_import_0"
"","","","","","","","tva_purchase_11","tva_import_0"
"","","","","","","","tva_purchase_6","tva_import_0"

```

## File: data\template\account.fiscal.position-ga_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.ga","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_19","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_11","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_6","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_19","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_11","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_6","syscebnl_tva_import_0"

```

## File: data\template\account.tax-ga.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_19","19%","18% VAT + 1% CSS","","","","group","sale","tax_group_19","tva_sale_18,css_sale_1","","","","","","","18% TVA + 1% CSS"
"tva_purchase_19","19%","18% VAT + 1% CSS","","","","group","purchase","tax_group_19","tva_purchase_18,css_purchase_1","","","","","","","18% TVA + 1% CSS"
"tva_sale_11","11%","10% VAT + 1% CSS","","","","group","sale","tax_group_11","tva_sale_10,css_sale_1","","","","","","","10% TVA + 1% CSS"
"tva_purchase_11","11%","10% VAT + 1% CSS","","","","group","purchase","tax_group_11","tva_purchase_10,css_purchase_1","","","","","","","10% TVA + 1% CSS"
"tva_sale_6","6%","5% VAT + 1% CSS","","","","group","sale","tax_group_6","tva_sale_5,css_sale_1","","","","","","","5% TVA + 1% CSS"
"tva_purchase_6","6%","5% VAT + 1% CSS","","","","group","purchase","tax_group_6","tva_purchase_5,css_purchase_1","","","","","","","5% TVA + 1% CSS"
"tva_sale_lasm_19","19% SD","18% VAT + 1% CSS self-delivery","False","","","group","sale","tax_group_19","tva_lasm_18,css_lasm_1","","","","","","19% LASM","18% TVA + 1% CSS livraison à soi-même"
"tva_sale_18","18%","","False","","18.0","percent","sale","tax_group_18","","base","invoice","","+GA_base_3_18","","",""
"","","","","","","","","","","tax","invoice","pcg_4431","+GA_tax_3_18","","",""
"","","","","","","","","","","base","refund","","-GA_base_3_18","","",""
"","","","","","","","","","","tax","refund","pcg_4431","-GA_tax_3_18","","",""
"tva_sale_10","10%","","False","","10.0","percent","sale","tax_group_10","","base","invoice","","+GA_base_3_10","","",""
"","","","","","","","","","","tax","invoice","pcg_4431","+GA_tax_3_10","","",""
"","","","","","","","","","","base","refund","","-GA_base_3_10","","",""
"","","","","","","","","","","tax","refund","pcg_4431","-GA_tax_3_10","","",""
"tva_sale_5","5%","","False","","5.0","percent","sale","tax_group_5","","base","invoice","","+GA_base_3_5","","",""
"","","","","","","","","","","tax","invoice","pcg_4431","+GA_tax_3_5","","",""
"","","","","","","","","","","base","refund","","-GA_base_3_5","","",""
"","","","","","","","","","","tax","refund","pcg_4431","-GA_tax_3_5","","",""
"css_sale_1","1% CSS","1% Special solidarity contribution","False","","1.0","percent","sale","tax_group_1","","base","invoice","","+GA_css","","","1% Contribution spéciale de solidarité"
"","","","","","","","","","","tax","invoice","pcg_4431","+GA_css_tax","","",""
"","","","","","","","","","","base","refund","","-GA_css","","",""
"","","","","","","","","","","tax","refund","pcg_4431","-GA_css_tax","","",""
"tva_lasm_18","18% SD","18% (self-delivery)","False","","18.0","percent","sale","tax_group_18","","base","invoice","","+GA_base_5b_18","","18% LASM","18% (livraison à soi-même)"
"","","","","","","","","","","tax","invoice","pcg_4431","-GA_tax_5b_18","-100","",""
"","","","","","","","","","","tax","invoice","pcg_4452","+GA_7a_18","","",""
"","","","","","","","","","","base","refund","","-GA_base_5b_18||-GA_css_other","","",""
"","","","","","","","","","","tax","refund","pcg_4431","+GA_tax_5b_18","-100","",""
"","","","","","","","","","","tax","refund","pcg_4452","-GA_7a_18","","",""
"tva_lasm_10","10% SD","10% (self-delivery)","False","","10.0","percent","sale","tax_group_10","","base","invoice","","+GA_base_5b_10","","10% LASM","10% (livraison à soi-même)"
"","","","","","","","","","","tax","invoice","pcg_4431","-GA_tax_5b_10","-100","",""
"","","","","","","","","","","tax","invoice","pcg_4452","+GA_7a_10","","",""
"","","","","","","","","","","base","refund","","-GA_base_5b_10","","",""
"","","","","","","","","","","tax","refund","pcg_4431","+GA_tax_5b_10","-100","",""
"","","","","","","","","","","tax","refund","pcg_4452","-GA_7a_10","","",""
"tva_lasm_5","5% SD","5% (self-delivery)","False","","5.0","percent","sale","tax_group_5","","base","invoice","","+GA_base_5b_5","","5% LASM","5% (livraison à soi-même)"
"","","","","","","","","","","tax","invoice","pcg_4431","-GA_tax_5b_5","-100","",""
"","","","","","","","","","","tax","invoice","pcg_4452","+GA_7a_5","","",""
"","","","","","","","","","","base","refund","","-GA_base_5b_5","","",""
"","","","","","","","","","","tax","refund","pcg_4431","+GA_tax_5b_5","-100","",""
"","","","","","","","","","","tax","refund","pcg_4452","-GA_7a_5","","",""
"css_lasm_1","1% CSS SD","1% Special solidarity contribution, self-delivery","False","","1.0","percent","sale","tax_group_1","","base","invoice","","+GA_css_other","","1% CSS LASM","1% Contribution spéciale de solidarité, livraison à soi-même"
"","","","","","","","","","","tax","invoice","pcg_4431","+GA_css_tax","","",""
"","","","","","","","","","","base","refund","","-GA_css_other","","",""
"","","","","","","","","","","tax","refund","pcg_4431","-GA_css_tax","","",""
"tva_purchase_18","18%","","False","","18.0","percent","purchase","tax_group_18","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","pcg_4452","+GA_7b_18","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","pcg_4452","-GA_7b_18","","",""
"tva_purchase_10","10%","","False","","10.0","percent","purchase","tax_group_10","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","pcg_4452","+GA_7b_10","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","pcg_4452","-GA_7b_10","","",""
"tva_purchase_5","5%","","False","","5.0","percent","purchase","tax_group_5","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","pcg_4452","+GA_7b_5","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","pcg_4452","-GA_7b_5","","",""
"css_purchase_1","1%","","False","","1.0","percent","purchase","tax_group_1","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","pcg_4452","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","pcg_4452","","","",""
"tva_export_0","0% EX","0% (export)","","","0.0","","sale","tax_group_0","","base","invoice","","+GA_export","","","0% (exportation)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","-GA_export","","",""
"","","","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","","","0.0","","purchase","tax_group_0","","base","invoice","","","","","0% (importation)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","","","0.0","","sale","tax_group_0","","base","invoice","","+GA_non_impo","","","0% (exonéré)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","-GA_non_impo","","",""
"","","","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","","0.0","","purchase","tax_group_0","","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-ga_syscebnl.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_19","19%","18% VAT + 1% CSS","","","","group","sale","syscebnl_tax_group_19","syscebnl_tva_sale_18,css_sale_1","","","","","","","18% TVA + 1% CSS"
"syscebnl_tva_purchase_19","19%","18% VAT + 1% CSS","","","","group","purchase","syscebnl_tax_group_19","syscebnl_tva_purchase_18,css_purchase_1","","","","","","","18% TVA + 1% CSS"
"syscebnl_tva_sale_11","11%","10% VAT + 1% CSS","","","","group","sale","syscebnl_tax_group_11","syscebnl_tva_sale_10,css_sale_1","","","","","","","10% TVA + 1% CSS"
"syscebnl_tva_purchase_11","11%","10% VAT + 1% CSS","","","","group","purchase","syscebnl_tax_group_11","syscebnl_tva_purchase_10,css_purchase_1","","","","","","","10% TVA + 1% CSS"
"syscebnl_tva_sale_6","6%","5% VAT + 1% CSS","","","","group","sale","syscebnl_tax_group_6","syscebnl_tva_sale_5,css_sale_1","","","","","","","5% TVA + 1% CSS"
"syscebnl_tva_purchase_6","6%","5% VAT + 1% CSS","","","","group","purchase","syscebnl_tax_group_6","syscebnl_tva_purchase_5,css_purchase_1","","","","","","","5% TVA + 1% CSS"
"syscebnl_tva_sale_lasm_19","19% SD","18% VAT + 1% CSS self-delivery","False","","","group","sale","syscebnl_tax_group_19","syscebnl_tva_lasm_18,css_lasm_1","","","","","","19% LASM","18% TVA + 1% CSS livraison à soi-même"
"syscebnl_tva_sale_18","18%","","False","","18.0","percent","sale","syscebnl_tax_group_18","","base","invoice","","+GA_base_3_18","","",""
"","","","","","","","","","","tax","invoice","syscebnl_443","+GA_tax_3_18","","",""
"","","","","","","","","","","base","refund","","-GA_base_3_18","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","-GA_tax_3_18","","",""
"syscebnl_tva_sale_10","10%","","False","","10.0","percent","sale","syscebnl_tax_group_10","","base","invoice","","+GA_base_3_10","","",""
"","","","","","","","","","","tax","invoice","syscebnl_443","+GA_tax_3_10","","",""
"","","","","","","","","","","base","refund","","-GA_base_3_10","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","-GA_tax_3_10","","",""
"syscebnl_tva_sale_5","5%","","False","","5.0","percent","sale","syscebnl_tax_group_5","","base","invoice","","+GA_base_3_5","","",""
"","","","","","","","","","","tax","invoice","syscebnl_443","+GA_tax_3_5","","",""
"","","","","","","","","","","base","refund","","-GA_base_3_5","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","-GA_tax_3_5","","",""
"css_sale_1","1% CSS","1% Special solidarity contribution","False","","1.0","percent","sale","syscebnl_tax_group_1","","base","invoice","","+GA_css","","","1% Contribution spéciale de solidarité"
"","","","","","","","","","","tax","invoice","syscebnl_443","+GA_css_tax","","",""
"","","","","","","","","","","base","refund","","-GA_css","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","-GA_css_tax","","",""
"syscebnl_tva_lasm_18","18% SD","18% (self-delivery)","False","","18.0","percent","sale","syscebnl_tax_group_18","","base","invoice","","+GA_base_5b_18","","18% LASM","18% (livraison à soi-même)"
"","","","","","","","","","","tax","invoice","syscebnl_443","-GA_tax_5b_18","-100","",""
"","","","","","","","","","","tax","invoice","syscebnl_445","+GA_7a_18","","",""
"","","","","","","","","","","base","refund","","-GA_base_5b_18||-GA_css_other","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","+GA_tax_5b_18","-100","",""
"","","","","","","","","","","tax","refund","syscebnl_445","-GA_7a_18","","",""
"syscebnl_tva_lasm_10","10% SD","10% (self-delivery)","False","","10.0","percent","sale","syscebnl_tax_group_10","","base","invoice","","+GA_base_5b_10","","10% LASM","10% (livraison à soi-même)"
"","","","","","","","","","","tax","invoice","syscebnl_443","-GA_tax_5b_10","-100","",""
"","","","","","","","","","","tax","invoice","syscebnl_445","+GA_7a_10","","",""
"","","","","","","","","","","base","refund","","-GA_base_5b_10","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","+GA_tax_5b_10","-100","",""
"","","","","","","","","","","tax","refund","syscebnl_445","-GA_7a_10","","",""
"syscebnl_tva_lasm_5","5% SD","5% (self-delivery)","False","","5.0","percent","sale","syscebnl_tax_group_5","","base","invoice","","+GA_base_5b_5","","5% LASM","5% (livraison à soi-même)"
"","","","","","","","","","","tax","invoice","syscebnl_443","-GA_tax_5b_5","-100","",""
"","","","","","","","","","","tax","invoice","syscebnl_445","+GA_7a_5","","",""
"","","","","","","","","","","base","refund","","-GA_base_5b_5","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","+GA_tax_5b_5","-100","",""
"","","","","","","","","","","tax","refund","syscebnl_445","-GA_7a_5","","",""
"css_lasm_1","1% CSS SD","1% Special solidarity contribution, self-delivery","False","","1.0","percent","sale","syscebnl_tax_group_1","","base","invoice","","+GA_css_other","","1% CSS LASM","1% Contribution spéciale de solidarité, livraison à soi-même"
"","","","","","","","","","","tax","invoice","syscebnl_443","+GA_css_tax","","",""
"","","","","","","","","","","base","refund","","-GA_css_other","","",""
"","","","","","","","","","","tax","refund","syscebnl_443","-GA_css_tax","","",""
"syscebnl_tva_purchase_18","18%","","False","","18.0","percent","purchase","syscebnl_tax_group_18","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","syscebnl_445","+GA_7b_18","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","syscebnl_445","-GA_7b_18","","",""
"syscebnl_tva_purchase_10","10%","","False","","10.0","percent","purchase","syscebnl_tax_group_10","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","syscebnl_445","+GA_7b_10","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","syscebnl_445","-GA_7b_10","","",""
"syscebnl_tva_purchase_5","5%","","False","","5.0","percent","purchase","syscebnl_tax_group_5","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","syscebnl_445","+GA_7b_5","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","syscebnl_445","-GA_7b_5","","",""
"css_purchase_1","1%","","False","","1.0","percent","purchase","syscebnl_tax_group_1","","base","invoice","","","","",""
"","","","","","","","","","","tax","invoice","syscebnl_445","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","syscebnl_445","","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","","0.0","","sale","syscebnl_tax_group_0","","base","invoice","","+GA_export","","","0% (exportation)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","-GA_export","","",""
"","","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","","","0.0","","purchase","syscebnl_tax_group_0","","base","invoice","","","","","0% (importation)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","","0.0","","sale","syscebnl_tax_group_0","","base","invoice","","+GA_non_impo","","","0% (exonéré)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","-GA_non_impo","","",""
"","","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","","0.0","","purchase","syscebnl_tax_group_0","","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-ga.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_6","Tax 6%","Taxe 6%","pcg_4431","pcg_4452"
"tax_group_11","Tax 11%","Taxe 11%","pcg_4431","pcg_4452"
"tax_group_19","Tax 19%","Taxe 19%","pcg_4431","pcg_4452"
"tax_group_1","CSS 1%","CSS 5%","pcg_4431","pcg_4452"
"tax_group_5","VAT 5%","T.V.A. 5%","pcg_4431","pcg_4452"
"tax_group_10","VAT 10%","T.V.A. 10%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-ga_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_6","Tax 6%","Taxe 6%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_11","Tax 11%","Taxe 11%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_19","Tax 19%","Taxe 19%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_1","CSS 1%","CSS 5%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_5","VAT 5%","T.V.A. 5%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_10","VAT 10%","T.V.A. 10%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_ga.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ga')
    def _get_ga_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('ga', 'res.company')
    def _get_ga_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.ga',
                'account_sale_tax_id': 'tva_sale_19',
                'account_purchase_tax_id': 'tva_purchase_19',
            }
        )
        return company_values

    @template('ga', 'account.account')
    def _get_ga_account_account(self):
        return self._parse_csv('ga', 'account.account', module='l10n_syscohada')

```

## File: models\template_ga_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ga_syscebnl')
    def _get_ga_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('ga_syscebnl', 'res.company')
    def _get_ga_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.ga',
                'account_sale_tax_id': 'syscebnl_tva_sale_19',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_19',
            }
        )
        return company_values

    @template('ga_syscebnl', 'account.account')
    def _get_ga_syscebnl_account_account(self):
        return self._parse_csv('ga_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_ga
from . import template_ga_syscebnl

```

