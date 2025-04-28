# Odoo Module: l10n_hr

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
    'name': 'Croatia - Accounting (Euro)',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['hr'],
    'description': """
Croatian Chart of Accounts updated (RRIF ver.2021)

Sources:
https://www.rrif.hr/dok/preuzimanje/Bilanca-2016.pdf
https://www.rrif.hr/dok/preuzimanje/RRIF-RP2021.PDF
https://www.rrif.hr/dok/preuzimanje/RRIF-RP2021-ENG.PDF
    """,
    'version': '13.0',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account',
        'base_vat',
    ],
    'data': [
        'data/l10n_hr_chart_data.xml',
        'data/account_tax_report_data.xml',
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
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.hr"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_tax_base" model="account.report.column">
                <field name="name">Tax base</field>
                <field name="expression_label">tax_base</field>
            </record>
            <record id="tax_report_tax_amount" model="account.report.column">
                <field name="name">Tax amount</field>
                <field name="expression_label">tax_amount</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_title_vat_calculation" model="account.report.line">
                <field name="name">Vat calculation of supplied goods and services - total value(i. + ii.)</field>
                <field name="code">c_0_total</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_0_0_tag_column1" model="account.report.expression">
                        <field name="label">tax_base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c_I_total.tax_base + c_II_total.tax_base</field>
                    </record>
                    <record id="tax_report_line_0_0_tag_column2" model="account.report.expression">
                        <field name="label">tax_amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c_II_total.tax_amount</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_transactions0" model="account.report.line">
                <field name="name">I. Transactions not subject to vat and exempt - total value (1.+2.+3.+4.+5.+6.+7.+8.+9.+10.)</field>
                <field name="code">c_I_total</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_I_0_tag_column1" model="account.report.expression">
                        <field name="label">tax_base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr01.tax_base + hr02.tax_base + hr03.tax_base + hr04.tax_base + hr05.tax_base + hr06.tax_base + hr07.tax_base + hr08.tax_base + hr09.tax_base + hr10.tax_base </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_I_1" model="account.report.line">
                        <field name="name">1. Supplies in the republic of croatia in respect of which the receiver calculates vat (domestic reverse charge)</field>
                        <field name="code">hr01</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_1_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.1_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_2" model="account.report.line">
                        <field name="name">2. Supplies carried out in another member state</field>
                        <field name="code">hr02</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_2_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.2_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_3" model="account.report.line">
                        <field name="name">3. Supplies of goods within the eu</field>
                        <field name="code">hr03</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_3_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.3_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_4" model="account.report.line">
                        <field name="name">4. Supplies of services within the eu</field>
                        <field name="code">hr04</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_4_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.4_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_5" model="account.report.line">
                        <field name="name">5. Supplies to persons not established in the republic of croatia</field>
                        <field name="code">hr05</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_5_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.5_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_6" model="account.report.line">
                        <field name="name">6. Installed and assembled goods in another member state</field>
                                <field name="code">hr06</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_6_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.6_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_7" model="account.report.line">
                        <field name="name">7. Supplies of new means of transport to another member state</field>
                        <field name="code">hr07</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_7_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.7_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_8" model="account.report.line">
                        <field name="name">8. Domestic supplies</field>
                        <field name="code">hr08</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_8_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.8_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_9" model="account.report.line">
                        <field name="name">9. Exportation</field>
                        <field name="code">hr09</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_9_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.9_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_I_10" model="account.report.line">
                        <field name="name">10. Other exemptions</field>
                        <field name="code">hr10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_I_10_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">I.10_base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_transactions1" model="account.report.line">
                <field name="name">II. Taxable transactions – total amount (1.+2.+3.+4.+5.+6.+7.+8.+9.+10.+11.+12.+13.+14.+15.)</field>
                <field name="code">c_II_total</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_II_0_tag_column1" model="account.report.expression">
                        <field name="label">tax_base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr11.tax_base + hr12.tax_base + hr13.tax_base + hr14.tax_base + hr15.tax_base + hr16.tax_base + hr17.tax_base + hr18.tax_base + hr19.tax_base + hr20.tax_base + hr21.tax_base + hr22.tax_base + hr23.tax_base + hr24.tax_base + hr25.tax_base</field>
                    </record>
                    <record id="tax_report_line_II_0_tag_column2" model="account.report.expression">
                        <field name="label">tax_amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr11.tax_amount + hr12.tax_amount + hr13.tax_amount + hr14.tax_amount + hr15.tax_amount + hr16.tax_amount + hr17.tax_amount + hr18.tax_amount + hr19.tax_amount + hr20.tax_amount + hr21.tax_amount + hr22.tax_amount + hr23.tax_amount + hr24.tax_amount + hr25.tax_amount</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_II_1" model="account.report.line">
                        <field name="name">1. Supplies of goods and services in the republic of croatia at a rate 5%</field>
                        <field name="code">hr11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_1_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.1_base</field>
                            </record>
                            <record id="tax_report_line_II_1_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.1_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_2" model="account.report.line">
                        <field name="name">2. Supplies of goods and services in the republic of croatia at a rate 13%</field>
                        <field name="code">hr12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_2_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.2_base</field>
                            </record>
                            <record id="tax_report_line_II_2_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.2_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_3" model="account.report.line">
                        <field name="name">3. Supplies of goods and services in the republic of croatia at a rate 25%</field>
                        <field name="code">hr13</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_3_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.3_base</field>
                            </record>
                            <record id="tax_report_line_II_3_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.3_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_4" model="account.report.line">
                        <field name="name">4. Supplies received in the republic of croatia in respect of which the receiver calculates vat (domestic reverse charge)</field>
                        <field name="code">hr14</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_4_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.4_base</field>
                            </record>
                            <record id="tax_report_line_II_4_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.4_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_5" model="account.report.line">
                        <field name="name">5. Acquisition of goods within the eu at a rate 5%</field>
                        <field name="code">hr15</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_5_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.5_base</field>
                            </record>
                            <record id="tax_report_line_II_5_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.5_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_6" model="account.report.line">
                        <field name="name">6. Acquisition of goods within the eu at a rate 13%</field>
                        <field name="code">hr16</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_6_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.6_base</field>
                            </record>
                            <record id="tax_report_line_II_6_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.6_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_7" model="account.report.line">
                        <field name="name">7. Acquisition of goods within the eu at a rate 25%</field>
                        <field name="code">hr17</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_7_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.7_base</field>
                            </record>
                            <record id="tax_report_line_II_7_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.7_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_8" model="account.report.line">
                        <field name="name">8. Services received from eu at a rate 5%</field>
                        <field name="code">hr18</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_8_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.8_base</field>
                            </record>
                            <record id="tax_report_line_II_8_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.8_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_9" model="account.report.line">
                        <field name="name">9. Services received from eu at a rate 13%</field>
                        <field name="code">hr19</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_9_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.9_base</field>
                            </record>
                            <record id="tax_report_line_II_9_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.9_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_10" model="account.report.line">
                        <field name="name">10. Services received from eu at a rate 25%</field>
                        <field name="code">hr20</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_10_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.10_base</field>
                            </record>
                            <record id="tax_report_line_II_10_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.10_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_11" model="account.report.line">
                        <field name="name">11. Supplies of goods and services received from taxable persons not established in the republic of croatia at a rate 5%</field>
                        <field name="code">hr21</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_11_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.11_base</field>
                            </record>
                            <record id="tax_report_line_II_11_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.11_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_12" model="account.report.line">
                        <field name="name">12. Supplies of goods and services received from taxable persons not established in the republic of croatia at a rate 13%</field>
                        <field name="code">hr22</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_12_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.12_base</field>
                            </record>
                            <record id="tax_report_line_II_12_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.12_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_13" model="account.report.line">
                        <field name="name">13. Supplies of goods and services received from taxable persons not established in the republic of croatia at a rate 25%</field>
                        <field name="code">hr23</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_13_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.13_base</field>
                            </record>
                            <record id="tax_report_line_II_13_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.13_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_14" model="account.report.line">
                        <field name="name">14. Subsequent exemption on exportation related to personal  passenger transportation</field>
                        <field name="code">hr24</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_14_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.14_base</field>
                            </record>
                            <record id="tax_report_line_II_14_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.14_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_II_15" model="account.report.line">
                        <field name="name">15. Vat calculated on importation</field>
                        <field name="code">hr25</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_II_15_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.15_base</field>
                            </record>
                            <record id="tax_report_line_II_15_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II.15_tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_calculed_vat" model="account.report.line">
                <field name="name">III. Calculated input vat – total amount (1.+2.+3.+4.+5.+6.+7.+8.+9.+10.+11.+12.+13. +14.+15.)</field>
                <field name="code">c_III_total</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_III_0_tag_column1" model="account.report.expression">
                        <field name="label">tax_base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr26.tax_base + hr27.tax_base + hr28.tax_base + hr29.tax_base + hr30.tax_base + hr31.tax_base + hr32.tax_base + hr33.tax_base + hr34.tax_base + hr35.tax_base + hr36.tax_base + hr37.tax_base + hr38.tax_base + hr39.tax_base</field>
                    </record>
                    <record id="tax_report_line_III_0_tag_column2" model="account.report.expression">
                        <field name="label">tax_amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr26.tax_amount + hr27.tax_amount + hr28.tax_amount + hr29.tax_amount + hr30.tax_amount + hr31.tax_amount + hr32.tax_amount + hr33.tax_amount + hr34.tax_amount + hr35.tax_amount + hr36.tax_amount + hr37.tax_amount + hr38.tax_amount + hr39.tax_amount + hr40.tax_amount</field>
                    </record>
                </field>
                <field name="children_ids">
                     <record id="tax_report_line_III_1" model="account.report.line">
                        <field name="name">1. Input vat related to supplies received in the country at a rate 5%</field>
                        <field name="code">hr26</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_1_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.1_base</field>
                            </record>
                            <record id="tax_report_line_III_1_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.1_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_2" model="account.report.line">
                        <field name="name">2. Input vat related to supplies received in the country at a rate 13%</field>
                        <field name="code">hr27</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_2_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.2_base</field>
                            </record>
                            <record id="tax_report_line_III_2_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.2_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_3" model="account.report.line">
                        <field name="name">3. Input vat related to supplies received in the country at a rate 25%</field>
                        <field name="code">hr28</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_3_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.3_base</field>
                            </record>
                            <record id="tax_report_line_III_3_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.3_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_4" model="account.report.line">
                        <field name="name">4. Input vat related to supplies received in the country in respect of which the receiver calculates vat (domestic reverse charge)</field>
                        <field name="code">hr29</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_4_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.4_base</field>
                            </record>
                            <record id="tax_report_line_III_4_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.4_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_5" model="account.report.line">
                        <field name="name">5. Input vat related to acquisition of goods within the eu at a rate 5%</field>
                        <field name="code">hr30</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_5_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.5_base</field>
                            </record>
                            <record id="tax_report_line_III_5_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.5_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_6" model="account.report.line">
                        <field name="name">6. Input vat related to acquisition of goods within the eu at a rate 13%</field>
                        <field name="code">hr31</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_6_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.6_base</field>
                            </record>
                            <record id="tax_report_line_III_6_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.6_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_7" model="account.report.line">
                        <field name="name">7. Input vat related to acquisition of goods within the eu at a rate 25%</field>
                        <field name="code">hr32</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_7_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.7_base</field>
                            </record>
                            <record id="tax_report_line_III_7_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.7_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_8" model="account.report.line">
                        <field name="name">8. Input vat related to services received from eu at a rate 5%</field>
                        <field name="code">hr33</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_8_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.8_base</field>
                            </record>
                            <record id="tax_report_line_III_8_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.8_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_9" model="account.report.line">
                        <field name="name">9. Input vat related to services received from eu at a rate 13%</field>
                        <field name="code">hr34</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_9_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.9_base</field>
                            </record>
                            <record id="tax_report_line_III_9_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.9_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_10" model="account.report.line">
                        <field name="name">10. Input vat related to services received from eu at a rate 25%</field>
                        <field name="code">hr35</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_10_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.10_base</field>
                            </record>
                            <record id="tax_report_line_III_10_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                            <field name="formula">III.10_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_11" model="account.report.line">
                        <field name="name">11. Input vat related to supplies of goods and services received from non-established taxable persons at a rate 5%</field>
                        <field name="code">hr36</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_11_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.11_base</field>
                            </record>
                            <record id="tax_report_line_III_11_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                            <field name="formula">III.11_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_12" model="account.report.line">
                        <field name="name">12. Input vat related to supplies of goods and services received from non-established taxable persons at a rate 13%</field>
                        <field name="code">hr37</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_12_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.12_base</field>
                            </record>
                            <record id="tax_report_line_III_12_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                            <field name="formula">III.12_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_13" model="account.report.line">
                        <field name="name">13. Input vat related to supplies of goods and services received from non-established taxable persons at a rate 25%</field>
                        <field name="code">hr38</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_13_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.13_base</field>
                            </record>
                            <record id="tax_report_line_III_13_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                            <field name="formula">III.13_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_14" model="account.report.line">
                        <field name="name">14. Input vat related to importation</field>
                        <field name="code">hr39</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_14_tag_column1" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">III.14_base</field>
                            </record>
                            <record id="tax_report_line_III_14_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                            <field name="formula">III.14_tax</field>
                            </record>
                        </field>
                     </record>
                     <record id="tax_report_line_III_15" model="account.report.line">
                        <field name="name">15. Adjustments of deductions</field>
                        <field name="code">hr40</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_III_15_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">tax_tags</field>
                            <field name="formula">III.15_tax</field>
                            </record>
                        </field>
                     </record>
                </field>
            </record>
            <record id="tax_report_title_vat_liab" model="account.report.line">
                <field name="name">IV. Vat liability in the calculation period: to pay (ii. - iii.) or to refund (iii. - ii.)</field>
                <field name="code">c_IV_total</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_IV_0_tag_column2" model="account.report.expression">
                        <field name="label">tax_amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr41.tax_amount + hr42.tax_amount</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_IV_1" model="account.report.line">
                        <field name="name">1. To pay</field>
                        <field name="code">hr41</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_IV_1_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">c_II_total.tax_amount - c_III_total.tax_amount</field>
                                <field name="subformula">if_above(HRK(0))</field>
                            </record>
                        </field>
                     </record>
                    <record id="tax_report_line_IV_2" model="account.report.line">
                        <field name="name">2. To refund</field>
                        <field name="code">hr42</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_IV_2_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">c_III_total.tax_amount - c_II_total.tax_amount</field>
                                <field name="subformula">if_above(HRK(0))</field>
                            </record>
                        </field>
                     </record>
                </field>
            </record>
            <record id="tax_report_title_prev_calculation" model="account.report.line">
                <field name="name">V. According to the previous calculation: unpaid vat to the day of submitting of this return-overpaid-tax credit</field>
                <field name="code">c_V_total</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_V_0_tag_column2" model="account.report.expression">
                        <field name="label">tax_amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr43.tax_amount + hr44.tax_amount </field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_V_1" model="account.report.line">
                        <field name="name">1. Claims for the difference of higher input tax than liabilities in the taxation period</field>
                        <field name="code">hr43</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_V_1_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">account_codes</field>
                                <field name="formula">1407</field>
                            </record>
                        </field>
                     </record>
                    <record id="tax_report_line_V_2" model="account.report.line">
                        <field name="name">2. Liability for the difference between tax and input tax in the taxation period</field>
                        <field name="code">hr44</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_V_2_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">account_codes</field>
                                <field name="formula">-2407</field>
                            </record>
                        </field>
                     </record>
                </field>
            </record>
            <record id="tax_report_title_total_dif" model="account.report.line">
                <field name="name">VI. Total difference: to pay/to refund</field>
                <field name="code">c_VI_total</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_VI_0_tag_column2" model="account.report.expression">
                        <field name="label">tax_amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">hr45.tax_amount + hr46.tax_amount</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_VI_1" model="account.report.line">
                        <field name="name">1. Total to pay</field>
                        <field name="code">hr45</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_VI_1_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">hr41.tax_amount - hr42.tax_amount + hr43.tax_amount - hr44.tax_amount</field>
                                <field name="subformula">if_above(HRK(0))</field>
                            </record>
                        </field>
                     </record>
                    <record id="tax_report_line_VI_2" model="account.report.line">
                        <field name="name">2. Total to refund</field>
                        <field name="code">hr46</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_VI_2_tag_column2" model="account.report.expression">
                                <field name="label">tax_amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">-hr41.tax_amount + hr42.tax_amount - hr43.tax_amount + hr44.tax_amount</field>
                                <field name="subformula">if_above(HRK(0))</field>
                            </record>
                        </field>
                     </record>
                </field>
            </record>
            <record id="tax_report_title_annual_deductible" model="account.report.line">
                <field name="name">VII. Annual deductible proportion(%)</field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_hr_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_hr_statements_menu" name="Croatia" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
    <data>
        <!-- Account Chart Template -->
        </data>
</odoo>

```

## File: data\template\account.account-hr.csv

```csv
"id","code","name","account_type","reconcile","name@hr"
"hr_000000","000000","Receivables for recoded but not paid in capital in joint-stock company","asset_non_current","False","Potraživanja za upisani a neuplaćeni dionički kapital (analitika po upisnicima)"
"hr_001000","001000","Receivables for repeated share issues for recorded but not paid capital","asset_non_current","False","Potraživanja iz ponovljene emisije dionica za upisane a neuplaćene svote kapitala (razrada po emisijama dionica)"
"hr_002000","002000","Receivables for recoded but not paid in capital in private limited company(analytics by owners)","asset_non_current","False","Potraživanja za upisani a neuplaćeni kapital u d.o.o. (analitika po članovima društva)"
"hr_003000","003000","Receivables for stake in capital of limited partnership","asset_non_current","False","Potraživanja za temeljni ulog komanditora"
"hr_004000","004000","Receivables for other business stakes in capital","asset_non_current","False","Potraživanje za ostale uloge u kapital"
"hr_010000","010000","Research and development","asset_non_current","False","Izdatci za razvoj"
"hr_011000","011000","Concession rights, patents, commodity and service brands","asset_non_current","False","Koncesije, patenti, licencije, robne i uslužne marke"
"hr_012000","012000","Software and other rights","asset_non_current","False","Softver i ostala prava"
"hr_013000","013000","Goodwill","asset_non_current","False","Goodwill"
"hr_014000","014000","Other intangible assets","asset_non_current","False","Ostala nematerijalna imovina"
"hr_015000","015000","Advance payments for purchase of intangible assets","asset_non_current","False","Predujmovi za nabavu nematerijalne imovine"
"hr_016000","016000","Cryptocurrencies","asset_non_current","False","Kriptovalute"
"hr_017000","017000","Intangible assets in preparation","asset_non_current","False","Nematerijalna imovina u pripremi (analitika prema vrsti računa skupine 01)"
"hr_018000","018000","Value adjustment of intangible assets","asset_non_current","False","Vrijednosno usklađenje nematerijalne imovine (analitika prema vrsti računa skupine 01)"
"hr_019000","019000","Accumulated depreciation of intangible assets","asset_non_current","False","Akumulirana amortizacija nematerijalne imovine"
"hr_020000","020000","Land","asset_fixed","False","Zemljišta"
"hr_021000","021000","Rights to use land","asset_fixed","False","Zemljišna prava (višegodišnja)"
"hr_023000","023000","Buildings","asset_fixed","False","Građevinski objekti (za vlastite potrebe)"
"hr_024000","024000","Flats for employees","asset_fixed","False","Stanovi za vlastite zaposlenike"
"hr_026000","026000","Advance payments for purchase of land and buildings","asset_fixed","False","Predujmovi za nabavu nekretnina"
"hr_027000","027000","Land and buildings under construction","asset_fixed","False","Nekretnine u pripremi"
"hr_028000","028000","Value adjustment of land and buildings","asset_fixed","False","Vrijednosno usklađenje nekretnina"
"hr_029000","029000","Accumulated depreciation of buildings","asset_fixed","False","Akumulirana amortizacija ulaganja u građevine"
"hr_030000","030000","Plant","asset_fixed","False","Postrojenja"
"hr_031000","031000","Equipment","asset_fixed","False","Oprema"
"hr_032000","032000","Tools, transportation equipment and vehicle","asset_fixed","False","Alati, pogonski inventar i transportna imovina"
"hr_033000","033000","Not deductible VAT prepayments (for cars, purchases after 1.1.2018. )","asset_fixed","False","Pretporez koji se ne može odbiti (kao dio vrijednosti osobnih automobila za nabave od 1.1. 2018.)"
"hr_034000","034000","Agricultural equipment and machinery","asset_fixed","False","Poljoprivredna oprema i mehanizacija"
"hr_035000","035000","Other property, plant and equipment","asset_fixed","False","Ostala materijalna imovina"
"hr_036000","036000","Advance payments for purchase of property, plant and equipment","asset_fixed","False","Advance payments for purchase of property, plant and equipment"
"hr_037000","037000","Plant and equipment under construction","asset_fixed","False","Materijalna imovina u pripremi"
"hr_038000","038000","Value adjustment of plant and equipment","asset_fixed","False","Vrijednosno usklađenje postrojenja i opreme"
"hr_039000","039000","Accumulated depreciation of plant and equipment","asset_fixed","False","Akumulirana amortizacija postrojenja i opreme"
"hr_040000","040000","Biological assets- growing crops","asset_non_current","False","Biološka imovina - bilje - višegodišnji nasadi"
"hr_041000","041000","Biological assets- live stock","asset_current","False","Biološka imovina - životinje - osnovno stado"
"hr_046000","046000","Advance payments for purchase of biological assets","asset_non_current","False","Predujmovi za biološku imovinu"
"hr_047000","047000","Biological assets in preparation (not put in use)","asset_non_current","False","Biološka imovina u pripremi"
"hr_048000","048000","Value adjustment of biological assets","asset_non_current","False","Vrijednosno usklađenje biološke imovine"
"hr_049000","049000","Accumulated depreciation of biological assets","asset_non_current","False","Akumulirana amortizacija biološke imovine"
"hr_050000","050000","Investment property-land","asset_non_current","False","Ulaganja u nekretnine - zemljišta"
"hr_051000","051000","Investment property- buildings","asset_non_current","False","Ulaganja u nekretnine - građevine"
"hr_056000","056000","Advance payments for investment properties","asset_non_current","False","Predujmovi za ulaganja u nekretnine"
"hr_057000","057000","Investments property under construction","asset_non_current","False","Ulaganja u nekretnine u pripremi"
"hr_058000","058000","Value adjustment of investment properties","asset_non_current","False","Vrijednosno usklađenje ulaganja u nekretnine"
"hr_059000","059000","Accumulated depreciation of buildings","asset_non_current","False","Akumulirana amortizacija ulaganja u građevine"
"hr_060000","060000","Investments in shares (shares) of entrepreneurs within the group","asset_non_current","False","Ulaganja u udjele (udjele) poduzetnika unutar grupe"
"hr_060100","060100","Investments in other securities of entrepreneurs within the group","asset_non_current","False","Investments in other securities of entrepreneurs within the group"
"hr_061000","061000","Loans, deposit etc given to a Group","asset_non_current","False","Dani zajmovi, depoziti i slično poduzetnicima unutar grupe"
"hr_062000","062000","Investments in shares (shares) of companies connected by participating interests","asset_non_current","False","Ulaganja u dionice (udjele) društava povezanih udjelima"
"hr_062100","062100","Investments in other securities of companies connected by participating interests","asset_non_current","False","Ulaganja u ostale vrijednosne papire društava povezanih sudjelujućim interesima"
"hr_063000","063000","Loans, deposit etc given to companies in associated undertakings","asset_non_current","False","Dani zajmovi, depoziti i slično društvima povezanim sudjelujućim interesom"
"hr_064000","064000","Investments in a securities","asset_non_current","False","Ulaganja u vrijednosne papire (dugotrajne)"
"hr_065000","065000","Loans given to third party","asset_non_current","False","Dani zajmovi, depoziti i sl."
"hr_066000","066000","Investments in associates accounted for using the equity method","asset_non_current","False","Ulaganja koja se obračunavaju metodom udjela"
"hr_067000","067000","Other long-term financial assets","asset_non_current","False","Ostala dugotrajna financijska imovina"
"hr_068000","068000","Value adjustment of long-term financial assets","asset_non_current","False","Vrijednosno usklađenje financijske imovine - dugotrajne (analitika prema oblicima imovine)"
"hr_069000","069000","Unrealized interests on loans etc.","asset_non_current","False","Nezarađene kamate u kreditima i sl."
"hr_070000","070000","Receivables in a Group","asset_receivable","True","Potraživanja od poduzetnika unutar grupe"
"hr_071000","071000","Receivable from associated undertakings","asset_receivable","True","Potraživanja od društava povezanih sudjelujućim interesom"
"hr_072000","072000","Long term trade receivables","asset_receivable","True","Potraživanja od kupaca - dugotrajna"
"hr_073000","073000","Receivables from factoring","asset_receivable","True","Potraživanja iz faktoringa"
"hr_074000","074000","Receivables for a given guarantees","asset_receivable","True","Potraživanja za jamčevine"
"hr_075000","075000","Receivables for legal cases in company's favour and other risk receivables","asset_receivable","True","Potraživanja u sporu i rizična potraživanja"
"hr_076000","076000","Receivables for advance payments for services","asset_receivable","True","Potraživanja za predujmove za usluge"
"hr_077000","077000","Other long-term receivables","asset_receivable","True","Ostala potraživanja - dugotrajna"
"hr_078000","078000","Value adjustment of long-term receivables","asset_receivable","True","Vrijednosno usklađivanje dugotrajnih potraživanja"
"hr_079000","079000","Receivables for unrealized interests in loans","asset_receivable","True","Potraživanja za nezarađenu kamatu"
"hr_080000","080000","Deferred temporary tax difference on income tax","asset_receivable","True","Odgođena privremena razlika poreza na dobitak (analitika po godinama)"
"hr_081000","081000","Other deferred tax asset","asset_receivable","True","Ostala odgođena porezna imovina"
"hr_100000","100000","Cash in bank (transaction accounts)","asset_cash","False","Transakcijski računi u bankama"
"hr_101000","101000","Letter of credit issued in domestic bank","asset_cash","False","Otvoreni akreditiv u domaćoj banci"
"hr_102000","102000","Cash on hand","asset_cash","False","Blagajne"
"hr_103000","103000","Foreign currency accounts","asset_cash","False","Devizni računi"
"hr_104000","104000","Letter of credit issued in foreign bank","asset_cash","False","Otvoreni akreditiv u stranim valutama"
"hr_105000","105000","Foreign cash on hand","asset_cash","False","Devizna blagajna"
"hr_106000","106000","Cash assigned to foreign currency purchases","asset_cash","False","Novac za kupnju deviza"
"hr_108000","108000","Other cash","asset_cash","False","Ostala novčana sredstva"
"hr_109000","109000","Value adjustment of bank deposits","asset_cash","False","Vrijednosno usklađenje depozita u bankama"
"hr_110000","110000","Investments in shares (shares) of entrepreneurs within the group","asset_current","False","Ulaganja u udjele (udjele) poduzetnika unutar grupe"
"hr_110100","110100","Investments in other securities of entrepreneurs within the group","asset_current","False","Investments in other securities of entrepreneurs within the group"
"hr_111000","111000","Loans, deposits etc in a Group","asset_current","False","Dani zajmovi, depoziti i slično poduzetnicima unutar grupe"
"hr_112000","112000","Investments in shares (shares) of companies connected by participating interests","asset_current","False","Ulaganja u dionice (udjele) društava povezanih udjelima"
"hr_112100","112100","Investments in other securities  of companies connected by participating interests","asset_current","False","Ulaganja u ostale vrijednosne papire društava povezanih sudjelujućim interesima"
"hr_113000","113000","Loans, deposits etc to companies in associated undertakings","asset_current","False","Dani zajmovi, depoziti i slično društvima povezanim sudjelujućim interesom"
"hr_114000","114000","Investments in securities","asset_current","False","Ulaganja u vrijednosne papire"
"hr_115000","115000","Leases given, deposits and other","asset_current","False","Dani zajmovi, depoziti i sl."
"hr_116000","116000","Receivables from investments funds","asset_current","False","Potraživanja iz ulaganja u investicijske fondove"
"hr_117000","117000","Other financial assets","asset_current","False","Ostala financijska imovina"
"hr_118000","118000","Receivables in dispute (e.g.. disputed, in bankruptcy etc, from financial assets)","asset_current","False","Potraživanja u sporu (npr. utužena, u predstečaju ili u stečaju i sl. iz fin. imovine)"
"hr_119000","119000","Value adjustment of current financial assets (value adjustment by analytics of this group of accounts)","asset_current","False","Vrijednosno usklađivanje financijske imovine - kratkotrajne (analitika po otpisima iz ove skupine računa)"
"hr_120000","120000","Trade receivables","asset_receivable","True","Potraživanja od kupaca"
"hr_120100","120100","Trade receivables (PoS)","asset_receivable","True","Potraživanja od kupaca (PoS)"
"hr_121000","121000","Foreign trade customers-EU and foreign","asset_receivable","True","Kupci iz EU i iz trećih zemalja"
"hr_122000","122000","Group companies receivables","asset_receivable","True","Potraživanja od poduzetnika unutar grupe"
"hr_123000","123000","Associated undertakings interest receivables","asset_receivable","True","Potraživanja od društava povezanih sudjelujućim interesom"
"hr_124000","124000","Interest receivables","asset_receivable","True","Potraživanja za kamate"
"hr_125000","125000","Receivables for advance payments for services","asset_receivable","True","Potraživanja za predujmove za usluge"
"hr_126000","126000","Foreign trade asset receivables","asset_receivable","True","Potraživanja iz vanjskotrgovačkog poslovanja"
"hr_127000","127000","Receivables from other activities","asset_receivable","True","Potraživanja s osnove ostalih aktivnosti"
"hr_128000","128000","Other current receivables","asset_receivable","True","Ostala kratkoročna potraživanja"
"hr_129000","129000","Value adjustment of receivables","asset_receivable","True","Vrijednosno usklađenje potraživanja"
"hr_130000","130000","Receivables from employees","asset_receivable","True","Potraživanja od zaposlenih"
"hr_131000","131000","Receivables from third party","asset_receivable","True","Potraživanja od vanjskih suradnika"
"hr_133000","133000","Receivables from equity owners","asset_receivable","True","Potraživanja od članova poduzetnika"
"hr_134000","134000","Receivables in dispute and risky receivables","asset_receivable","True","Potraživanja u sporu i rizična potraživanja (iz skupine 12 i 13)"
"hr_135000","135000","Receivables for assets held on free-trade zone","asset_receivable","True","Potraživanja za sredstva u slobodnoj zoni"
"hr_136000","136000","Receivables from banks based on customer's loans","asset_receivable","True","Potraživanja za prodaju na potrošački kredit"
"hr_137000","137000","Receivables from foreign business units","asset_receivable","True","Potraživanja od poslovnih jedinica u inozemstvu"
"hr_138000","138000","Other receivables from operating activities","asset_receivable","True","Ostala poslovna potraživanja"
"hr_139000","139000","Value adjustment of asset receivables from employees and other receivables","asset_receivable","True",""
"hr_140000","140000","Advance tax on received deliveries and advances in the Republic of Croatia","asset_current","False","Predujam poreza na primljene isporuke i predujmove u Republici Hrvatskoj"
"hr_140010","140010","Advance tax - 5%","asset_current","False","Pretporez - 5%"
"hr_140011","140011","Advance tax - 13%","asset_current","False","Pretporez - 13%"
"hr_140012","140012","Advance tax - 25%","asset_current","False","Pretporez - 25%"
"hr_140020","140020","Advance tax from advances - 5%","asset_current","False","Akontacija od akontacija - 5%"
"hr_140021","140021","Advance tax from advances - 13%","asset_current","False","Akontacija od akontacija - 13%"
"hr_140022","140022","Advance tax from advances - 25%","asset_current","False","Akontacija od akontacija - 25%"
"hr_140030","140030","Input tax corrections","asset_current","False","Ispravci ulaznog poreza"
"hr_140031","140031","Correction of input tax due to conversion of goods (exempt deliveries)","asset_current","False","Ispravak pretporeza zbog konverzije dobara (oslobođene isporuke)"
"hr_140032","140032","Input tax correction due to a change in input tax recognition percentage - adjustment","asset_current","False","Ispravak pretporeza zbog promjene postotka priznavanja pretporeza - ispravak"
"hr_140100","140100","Advance tax from transferred tax liability in the country - 25%","asset_current","False","Predujam poreza od prenesene porezne obveze u tuzemstvu - 25%"
"hr_140200","140200","Advance tax on the acquisition of goods from the EU - 5%","asset_current","False","Advance tax on the acquisition of goods from the EU - 5%"
"hr_140210","140210","Advance tax on the acquisition of goods from the EU - 13%","asset_current","False","Akontacija poreza na stjecanje dobara iz EU - 13%"
"hr_140220","140220","Advance tax on the acquisition of goods from the EU - 25%","asset_current","False","Akontacija poreza na stjecanje dobara iz EU - 25%"
"hr_140300","140300","Withholding tax from services received from the EU - 5%","asset_current","False","Porez po odbitku od usluga primljenih iz EU - 5%"
"hr_140310","140310","Withholding tax from services received from the EU - 13%","asset_current","False","Porez po odbitku za usluge primljene iz EU - 13%"
"hr_140320","140320","Withholding tax from services received from the EU - 25%","asset_current","False","Porez po odbitku od usluga primljenih iz EU - 25%"
"hr_140400","140400","Withholding tax from taxpayers without headquarters in the Republic of Croatia - 5%","asset_current","False","Porez po odbitku od poreznih obveznika bez sjedišta u Republici Hrvatskoj - 5%"
"hr_140410","140410","Withholding tax from taxpayers without headquarters in the Republic of Croatia - 13%","asset_current","False","Porez po odbitku od poreznih obveznika bez sjedišta u Republici Hrvatskoj - 13%"
"hr_140420","140420","Withholding tax from taxpayers without headquarters in the Republic of Croatia - 25%","asset_current","False","Porez po odbitku od poreznih obveznika bez sjedišta u Republici Hrvatskoj - 25%"
"hr_140500","140500","VAT on the import of goods","asset_current","False","PDV na uvoz robe"
"hr_140700","140700","Claims for the difference of higher input tax than liabilities in the taxation period","asset_current","False","Potraživanja razlike većeg pretporeza od obveza u poreznom razdoblju"
"hr_140800","140800","Input tax not yet recognized","asset_current","False","Ulazni porez još nije priznat"
"hr_141000","141000","Receivables from tax and surtax regarding salaries and other incomes","asset_receivable","True","Potraživanja za porez i prirez na dohodak iz plaća i drugih primanja"
"hr_142000","142000","Receivables for prepayments made regarding mandatory contributions on personal income tax","asset_receivable","True","Potraživanja za više plaćene doprinose iz plaća i na plaće"
"hr_143000","143000","Receivables for corporate income tax and withholding tax","asset_receivable","True","Potraživanja za porez na dobitak i po odbitku"
"hr_144000","144000","Receivables for excise duties and other taxes","asset_receivable","True","Potraživanja za posebne poreze, trošarine i druge poreze od države"
"hr_145000","145000","Receivables for National tourism office fee","asset_receivable","True","Potraživanja za plaćenu članarinu turističkim zajed."
"hr_146000","146000","Receivables for Croatian Economy Chamber or similar Chambers","asset_receivable","True","Potraživanja za članarine komori (HGK ili HOK i dr.)"
"hr_147000","147000","Receivables for duties and prepayments to customs duty","asset_receivable","True","Potraživanja za carinu i više plaćene carinske pristojbe"
"hr_148000","148000","Receivables for Country and municipal tax","asset_receivable","True","Potraživanja za županijski i općinski (gradski) porez"
"hr_149000","149000","Receivables for other not mentioned taxes, contributions and duties","asset_receivable","True","Potraživanja za ostale nespomenute poreze, doprinose, takse i pristojbe"
"hr_150000","150000","Receivables from Croatian institute for health insurance regarding employee's sick leave payments","asset_receivable","True","Potraživanje za nadoknade bolovanja od HZZO"
"hr_151000","151000","Receivables for Pension insurance institute","asset_receivable","True","Potraživanja od mirovinskog osiguranja"
"hr_152000","152000","Receivables from local government","asset_receivable","True","Potraživanja od lokalne samouprave"
"hr_153000","153000","Receivables for bonuses, premiums and government grants","asset_receivable","True","Potraživ. za regrese, premije, stimulac. i držav. potpore"
"hr_154000","154000","Receivables for Fund for packaging repurchase","asset_receivable","True","Potraživanje od Fonda za otkupljenu ambalažu"
"hr_155000","155000","Other receivables from the Government","asset_receivable","True","Ostala potraživanja od državnih institucija"
"hr_156000","156000","Receivables for more paid fees for disabled person","asset_receivable","True","Potraživanja za više plaćenu naknadu s temelja invaliditeta"
"hr_159000","159000","Value adjustment of receivables from the Government and other institutions","asset_receivable","True","Vrijednosno usklađenje potraživanja od države i drugih institucija"
"hr_190000","190000","Prepaid expense and accrued income","asset_current","False","Unaprijed plaćeni troškovi"
"hr_191000","191000","Accrued income","asset_current","False","Obračunani prihodi (budućeg razdoblja)"
"hr_192000","192000","Prepaid expense and accrued income","asset_current","False","Unaprijed plaćeni troškovi"
"hr_193000","193000","Accrued interest which relates to future periods","asset_current","False","Troškovi kamata iz budućeg razdoblja"
"hr_194000","194000","Prepaid expense for concession rights","asset_current","False","Unaprijed plaćeni troškovi koncesija (za razdoblje do 12 mj.)"
"hr_195000","195000","Prepaid expense for franchise fee","asset_current","False","Unaprijed plaćene franšize, trgovačko znakovlje, prava i sl. (do 12 mj.)"
"hr_196000","196000","Prepaid expense for licences and patents","asset_current","False","Unaprijed plaćene licencije i patenti (do 12 mj.)"
"hr_197000","197000","Derivatives- hedging instruments","asset_current","False","Derivativ - instrument zaštite"
"hr_199000","199000","Other prepaid expense","asset_current","False","Ostali plaćeni troškovi budućeg razdoblja"
"hr_200000","200000","Obligations towards Entrepreneurs within the Group","liability_current","False","Obveze prema poduzetnicima unutar Grupe"
"hr_200100","200100","Obligations for loans, deposits and the like of entrepreneurs within the group","liability_current","False","Obveze za kredite, depozite i sl. poduzetnika unutar grupe"
"hr_201000","201000","Liabilities due to share in results","liability_current","False","Obveze s osnove udjela u rezultatu"
"hr_210000","210000","Liabilities due to issued cheques","liability_current","False","Obveze za izdane čekove"
"hr_211000","211000","Liabilities due to issued promissory notes","liability_current","False","Obveze za izdane mjenice"
"hr_212000","212000","Liabilities due to issued securities","liability_current","False","Obveze po izdanim vrijednosnim papirima"
"hr_213000","213000","Obligations towards companies connected by participating interests","liability_current","False","Obveze prema društvima povezanim udjelima"
"hr_213100","213100","Obligations for loans, deposits and the like of companies connected by a participating interest","liability_current","False","Obveze za zajmove, depozite i sl. društava povezanih udjelom"
"hr_214000","214000","Lease liabilities and deposits received","liability_current","False","Obveze s osnove zajmova, depozita i sl."
"hr_215000","215000","Liabilities towards banks and other financial institutions","liability_current","False","Obveze prema bankama i drugim financijskim institucijama"
"hr_216000","216000","Liabilities towards credit card institutions","liability_current","False","Obveze prema izdavateljima kreditnih kartica (analitika prema kartičarima)"
"hr_217000","217000","Liabilities from discount services","liability_current","False","Obveze iz eskontnih poslova"
"hr_218000","218000","Liabilities due to asset_non_current held for sale (this account relates to due loan liabilities and account payable for asset which is held for sale (acc.69))","liability_current","False","Obveze s osnove dugotrajne imovine namijenjene prodaji (analitika prema izvorima)"
"hr_220000","220000","Domestic trade payable","liability_payable","True","Obveze prema dobavljačima u zemlji (analitika prema dobavljačima)"
"hr_221000","221000","Account payable from EU and third countries","liability_payable","True","Dobavljači iz EU i inozemstva (analitika prema dobavljačima)"
"hr_222000","222000","Trade payable- private person","liability_payable","True","Dobavljači fizičke osobe"
"hr_223000","223000","Trade payable for utility services","liability_payable","True","Dobavljači komunalnih usluga"
"hr_224000","224000","Liabilities for not invoiced but received goods and services","liability_payable","True","Obveze za nefakturirane a preuzete isporuke robe i usluge"
"hr_225000","225000","Liabilities for advances received","liability_payable","True","Obveze za predujmove"
"hr_230000","230000","Liabilities towards employees","liability_current","False","Obveze prema zaposlenicima"
"hr_231000","231000","Liability from procurements and grants","liability_current","False","Kratkoročne obveze iz nabava i potpora"
"hr_232000","232000","Interests payable","liability_current","False","Obveze za kamate (analitika prema dužnicima i vrstama obveza prema nepovezanim društvima)"
"hr_233000","233000","Liabilities for commission or consignment sale of goods","liability_current","False","Obveze po obračunu prodanih dobara primljenih u komisiju ili konsignaciju"
"hr_234000","234000","Liabilities towards insurance companies","liability_current","False","Obveze prema osiguravajućim društvima"
"hr_235000","235000","Liabilities due to operations in free-trade zone","liability_current","False","Obveze iz poslovanja u slobodnoj zoni"
"hr_236000","236000","Liabilities due to operations in foreign trade","liability_current","False","Obveze iz vanjskotrgovačkog poslovanja"
"hr_237000","237000","Liabilities towards foreign business units","liability_current","False","Obveze prema poslovnim jedinicama u inozemstvu"
"hr_238000","238000","Liabilities due to acquisitions of shares/stakes","liability_current","False","Obveze iz stjecanja udjela"
"hr_239000","239000","Other current liability","liability_current","False","Ostale kratkoročne obveze"
"hr_240000","240000","VAT obligations","liability_current","False","PDV obveze"
"hr_240010","240010","Obligation for VAT - 5%","liability_current","False","Obveza PDV-a - 5%"
"hr_240011","240011","Obligation for VAT - 13%","liability_current","False","Obveza PDV-a - 13%"
"hr_240012","240012","Obligation for VAT - 25%","liability_current","False","Obveza PDV-a - 25%"
"hr_240020","240020","VAT liability for advance payment - 5%","liability_current","False","Obveza PDV-a za akontaciju - 5%"
"hr_240021","240021","VAT liability for advance payment - 13%","liability_current","False","Obveza PDV-a za akontaciju - 13%"
"hr_240022","240022","VAT liability for advance payment - 25%","liability_current","False","Obveza PDV-a za akontaciju - 25%"
"hr_240030","240030","VAT liability for unbilled deliveries - 5%","liability_current","False","Obveza PDV-a za nenaplaćene isporuke - 5%"
"hr_240031","240031","VAT liability for unbilled deliveries - 13%","liability_current","False","Obveza PDV-a za nenaplaćene isporuke - 13%"
"hr_240032","240032","VAT liability for unbilled deliveries - 25%","liability_current","False","Obveza PDV-a za nenaplaćene isporuke - 25%"
"hr_240040","240040","VAT liability based on own consumption - 25%, 13% or 5%","liability_current","False","Obveza PDV-a na temelju vlastite potrošnje - 25%, 13% ili 5%"
"hr_240050","240050","Liability for corrected VAT due to conversion of goods","liability_current","False","Obveza za ispravljeni PDV zbog pretvorbe dobara"
"hr_240100","240100","Liability for VAT from the transferred tax liability from the home country - 25%","liability_current","False","Obveza za PDV iz prenesene porezne obveze iz tuzemstva - 25%"
"hr_240200","240200","VAT obligations for the acquisition of goods from the EU - 5%","liability_current","False","Obveze PDV-a za stjecanje dobara iz EU - 5%"
"hr_240210","240210","VAT obligations for the acquisition of goods from the EU - 13%","liability_current","False","Obveze PDV-a za stjecanje dobara iz EU - 13%"
"hr_240220","240220","VAT obligations for the acquisition of goods from the EU - 25%","liability_current","False","Obveze PDV-a za stjecanje dobara iz EU - 25%"
"hr_240300","240300","VAT obligations for services received from the EU - 5%","liability_current","False","Obveze PDV-a za primljene usluge iz EU - 5%"
"hr_240310","240310","VAT obligations for services received from the EU - 13%","liability_current","False","Obveze PDV-a za primljene usluge iz EU - 13%"
"hr_240320","240320","VAT obligations for services received from the EU - 25%","liability_current","False","Obveze PDV-a za primljene usluge iz EU - 25%"
"hr_240400","240400","VAT liability for goods and services received from taxpayers without headquarters in the Republic of Croatia - 5%","liability_current","False","Obveza PDV-a za primljena dobra i usluge od poreznih obveznika bez sjedišta u Republici Hrvatskoj - 5%"
"hr_240410","240410","VAT liability for received goods and services from taxpayers without headquarters in the Republic of Croatia - 13%","liability_current","False","Obveza PDV-a za primljena dobra i usluge od poreznih obveznika bez sjedišta u Republici Hrvatskoj - 13%"
"hr_240420","240420","VAT liability for goods and services received from taxpayers without headquarters in the Republic of Croatia - 25%","liability_current","False","Obveza PDV-a za primljena dobra i usluge od poreznih obveznika bez sjedišta u Republici Hrvatskoj - 25%"
"hr_240500","240500","Obligation for calculated VAT on import","liability_current","False","Obveza za obračunati PDV pri uvozu"
"hr_240700","240700","Liability for the difference between tax and input tax in the taxation period","liability_current","False","Obveza za razliku poreza i pretporeza u razdoblju oporezivanja"
"hr_240800","240800","Return of VAT from passenger traffic to natural persons from abroad","liability_current","False","Povrat PDV-a iz prometa putnika fizičkim osobama iz inozemstva"
"hr_241000","241000","Liabilities for personal income tax and surtax","liability_current","False","Obveze za porez i prirez na dohodak"
"hr_242000","242000","Liabilities for insurance contributions","liability_current","False","Obveze za doprinose za osiguranja"
"hr_243000","243000","Income tax payable, tax on investment income and withholding tax","liability_current","False","Obveze za porez na dobitak, dohodak od kapitala i porez po odbitku"
"hr_244000","244000","Liabilities for excise duties and other taxes","liability_current","False","Obveze za posebne poreze, trošarine i druge poreze prema državi"
"hr_245000","245000","Liabilities for fee to National tourism office","liability_current","False","Obveze za članarinu turističkim zajednicama"
"hr_246000","246000","Liabilities for Chamber membership fee","liability_current","False","Obveze za članarinu komori"
"hr_247000","247000","Liabilities for duties and customs duty","liability_current","False","Obveze za carinu i carinske pristojbe"
"hr_248000","248000","Liabilities for a Country's and municipal's taxes","liability_current","False","Obveze za lokalne poreze i druga davanja"
"hr_249000","249000","Liabilities for other not mentioned taxes, contributions and duties","liability_current","False","Ostale obveze javnih davanja"
"hr_250000","250000","Obligations towards Entrepreneurs within the Group","liability_current","False","Obveze prema poduzetnicima unutar Grupe"
"hr_250100","250100","Obligations for loans, deposits and the like of entrepreneurs within the group","liability_current","False","Obveze za kredite, depozite i sl. poduzetnika unutar grupe"
"hr_251000","251000","Obligations for loans, deposits and the like of companies connected by a participating","liability_current","False","Obveze za zajmove, depozite i sl. društava povezanih sudionicima"
"hr_251100","251100","Obligations for loans, deposits and the like interest","liability_current","False","Obveze za kredite, depozite i sl. kamate"
"hr_252000","252000","Liabilities towards banks and other financial institutions","liability_current","False","Obveze prema bankama i drugim financijskim institucijama"
"hr_253000","253000","Long term leasing liabilities","liability_current","False","Dugoročne obveze iz financijskog i operativnog lizinga"
"hr_254000","254000","Liabilities for advances received","liability_current","False","Obveze za predujmove"
"hr_255000","255000","Trade payable (unpaid long term trade payable)","liability_current","False","Obveze prema dobavljačima (neisplaćene dugoročne obveze prema vjerovnicima s osnove poslovanja)"
"hr_256000","256000","Liabilities for long term securities","liability_current","False","Obveze po vrijednosnim papirima (dugoročnim)"
"hr_257000","257000","Liabilities towards associated undertakings","liability_current","False","Obveze prema društvima povezanim sudjelujućim interesom"
"hr_258000","258000","Liabilities towards government","liability_current","False","Obveze prema Državi (realizirana jamstva, krediti, plaćene koncesije)"
"hr_259000","259000","Other long term liabilities","liability_current","False","Ostale dugoročne obveze"
"hr_260000","260000","Deferred tax liabilities","liability_current","False","Odgođena porezna obveza"
"hr_280000","280000","Provision for employee benefits","liability_non_current","False","Rezerviranja za otpremnine i mirovine"
"hr_281000","281000","Provision for tax liabilities","liability_non_current","False","Rezerviranja za porezne obveze"
"hr_282000","282000","Provisions for legal cases","liability_non_current","False","Rezerviranja za započete sudske sporove"
"hr_283000","283000","Provisions for renewal of natural resources","liability_non_current","False","Rezerviranja za troškove obnavljanja prirodnih bogatstava"
"hr_284000","284000","Provision for risks within warranty period","liability_non_current","False","Rezerviranja za troškove u jamstvenim rokovima"
"hr_285000","285000","Other provisions","liability_non_current","False","Druga rezerviranja"
"hr_290000","290000","Accrued expense","liability_non_current","False","Odgođeno plaćanje troškova"
"hr_291000","291000","Accrued expense for rights to use","liability_non_current","False","Obračunani troškovi korištenih prava"
"hr_292000","292000","Accrued expense for goods purchases","liability_non_current","False","Obračunani troškovi nabave dobara"
"hr_293000","293000","Deferred income (income which relates to future period)","liability_non_current","False","Obračunani prihodi budućeg razdoblja"
"hr_294000","294000","Deferred income from government grants","liability_non_current","False","Odgođeni prihodi iz državnih potpora (HSFI t. 15.37 i MRS 20, t. 12. i 24.)"
"hr_295000","295000","Deferred income","liability_non_current","False","Odgođeno priznavanje prihoda"
"hr_296000","296000","Deferred income relating to not invoiced but shipped goods and services","liability_non_current","False","Odgođeni prihod s osnove nefakturiranih isporuka dobara i usluga"
"hr_297000","297000","Unrealized gains on financial assets","liability_non_current","False","Nerealizirani dobitci iz financijske imovine (HSFI 9 i MSFI 9)"
"hr_298000","298000","Deferred invoiced income without delivery","liability_non_current","False","Odgođeni fakturirani prihod za koji nije nastala isporuka"
"hr_299000","299000","Provision for unused vacation days","liability_non_current","False","Ostala pasivna vremenska razgraničenja"
"hr_300000","300000","Payable purchase price","asset_current","False","Kupovna cijena dobavljača"
"hr_301000","301000","Other inventory dependant costs","asset_current","False","Ovisni troškovi nabave (u svezi s dovođenjem na zalihu)"
"hr_302000","302000","Duties and other customs duty","asset_current","False","Carina i druge uvozne pristojbe"
"hr_303000","303000","Excise duties which can not be deducted","asset_current","False","Posebni porezi (trošarine) koji se ne mogu odbiti"
"hr_309000","309000","Cost of inventory purchasing calculation","asset_current","False","Obračun troškova kupnje"
"hr_310000","310000","Raw material on stock","asset_current","False","Sirovine i materijal u skladištu"
"hr_311000","311000","Material in processing, finishing and manipulation phases","asset_current","False","Materijal u doradi, obradi i manipulaciji"
"hr_312000","312000","Material sent for finishing to partners","asset_current","False","Materijal na doradi kod ortaka"
"hr_313000","313000","Materials used in agriculture","asset_current","False","Zalihe materijala za poljoprivredu"
"hr_318000","318000","Inventory price adjustment","asset_current","False","Odstupanje od cijene zaliha"
"hr_319000","319000","Value adjustment of raw materials","asset_current","False","Vrijednosno usklađenje zaliha sirovina i materijala"
"hr_320000","320000","Spare parts on stock","asset_current","False","Rezervni dijelovi na zalihi"
"hr_328000","328000","Spare part price adjustment","asset_current","False","Odstupanje od cijene rez. dijelova na zalihi"
"hr_329000","329000","Value adjustment of spare parts","asset_current","False","Vrijednosno usklađenje zaliha rezervnih dijelova"
"hr_350000","350000","Small inventory on stock","asset_current","False","Sitan inventar na zalihi"
"hr_351000","351000","Packaging on stock","asset_current","False","Ambalaža na zalihi (samo vlastita i višekratna, analitika prema vrstama)"
"hr_352000","352000","Tires on stock","asset_current","False","Autogume na zalihi"
"hr_358000","358000","Small inventory's price adjustment","asset_current","False","Odstupanje od cijene"
"hr_359000","359000","Value adjustment of small inventory","asset_current","False","Vrijednosno usklađenje zaliha"
"hr_360000","360000","Small inventory in use","asset_current","False","Sitan inventar u uporabi"
"hr_361000","361000","Packaging in use","asset_current","False","Ambalaža u uporabi"
"hr_362000","362000","Tires in use","asset_current","False","Autogume u uporabi"
"hr_363000","363000","Small inventory write off","asset_current","False","Otpis sitnog inventara"
"hr_364000","364000","Packaging write off","asset_current","False","Otpis ambalaže"
"hr_365000","365000","Tires write off","asset_current","False","Otpis autoguma"
"hr_369000","369000","Value adjustment of small inventory, packaging and tires","asset_current","False","Vrijednosno usklađenje sitnog inventara, ambalaže i autoguma"
"hr_370000","370000","Advance payments for material","asset_current","False","Predujmovi dobavljačima materijala"
"hr_371000","371000","Advance payments for spare parts","asset_current","False","Predujmovi dobavljačima rezervnih dijelova"
"hr_372000","372000","Advance payments for small inventory","asset_current","False","Predujmovi dobavljačima sitnog inventara"
"hr_373000","373000","Advance payments to importer for raw materials, spare parts and small inventory","asset_current","False","Predujmovi dani uvozniku za nabavu sirovina i materijala, dijelova i inventara"
"hr_374000","374000","Advance payments to foreign suppliers","asset_current","False","Predujmovi inozemnim dobavljačima"
"hr_379000","379000","Value adjustment of advance payments","asset_current","False","Vrijednosno usklađenje danih predujmova"
"hr_400000","400000","Cost of raw material","expense","False","Nabavna vrijednost prodanih nekretnina i umjetnina"
"hr_401000","401000","Cost of material used in administration and sale department","expense","False","Materijalni troškovi administracije, uprave i prodaje"
"hr_402000","402000","Research and development cost","expense","False","Materijalni troškovi istraživanja i razvoja"
"hr_403000","403000","Packaging cost","expense","False","Troškovi ambalaže"
"hr_404000","404000","Cost of small inventory, packaging and tires","expense","False","Trošak sitnog inventara, ambalaže i autoguma"
"hr_405000","405000","Spare parts and material used on maintenance","expense","False","Rezervni dijelovi i materijal za održavanje"
"hr_406000","406000","Energy used during production of goods and services","expense","False","Potrošena energija u proizvodnji dobara i usluga"
"hr_407000","407000","Energy - administration and sale department","expense","False","Potrošena energija u administraciji, upravi i prodaji"
"hr_408000","408000","Costs for samples","expense","False","Troškovi uzoraka"
"hr_409000","409000","Standard cost adjustments","expense","False","Odstupanja od standardnog troška"
"hr_410000","410000","Telephone and transportation costs","expense","False","Troškovi telefona, prijevoza i sl."
"hr_411000","411000","External service costs used in goods production and providing services","expense","False","Troškovi vanjskih usluga pri izradi dobara i obavljanju usluga"
"hr_412000","412000","Maintenance and securities services","expense","False","Usluge održavanja i zaštite (servisne usluge)"
"hr_413000","413000","Vehicle registration and permission costs","expense","False","Usluge registracije prijevoznih sredstava i troškovi dozvola"
"hr_414000","414000","Leasing costs","expense","False","Usluge zakupa - leasinga"
"hr_415000","415000","Promotion , sponsorship and fairs costs","expense","False","Usluge promidžbe, sponzorstva i troškovi sajmova"
"hr_416000","416000","Intellectual services","expense","False","Intelektualne i osobne usluge"
"hr_417000","417000","Utilities and similar costs","expense","False","Troškovi komunalnih i sličnih usluga"
"hr_418000","418000","Representation costs - hosting","expense","False","Usluge reprezentacije - ugošćivanja"
"hr_419000","419000","Other external costs","expense","False","Troškovi ostalih vanjskih usluga"
"hr_420000","420000","Wages and salaries","expense","False","Neto plaće i nadoknade"
"hr_421000","421000","Income tax and surtax costs","expense","False","Troškovi poreza i prireza"
"hr_422000","422000","Contributions from salaries costs","expense","False","Troškovi doprinosa iz plaća"
"hr_423000","423000","Contributions on salaries costs","expense","False","Doprinosi na plaće"
"hr_424000","424000","Gross salaries","expense","False","Bruto plaće"
"hr_430000","430000","Amortisation of intangible assets","expense_depreciation","False","Amortizacija nematerijalne imovine"
"hr_431000","431000","Depreciation of property, plant and equipment","expense_depreciation","False","Amortizacija materijalne imovine"
"hr_432000","432000","Depreciation of cars and other vehicles for personal transportation","expense_depreciation","False","Amortizacija osobnih automobila i dr. sredstava za osobni prijevoz"
"hr_433000","433000","Depreciation of property, plant and - management and selling department","expense_depreciation","False","Amortizacija objekata i opreme uprave i prodaje"
"hr_434000","434000","Depreciation surplus due to revaluation of assets","expense_depreciation","False","Povećana amortizacija s temelja revalorizacije"
"hr_435000","435000","Depreciation of biological assets (vineyards, orchards, live stock)","expense_depreciation","False","Amortizacija biološke imovine (vinogradi, voćnjaci, osnovno stado i sl.)"
"hr_436000","436000","Tax not-deductible depreciation","expense_depreciation","False","Amortizacija iznad porezno dopuštene"
"hr_440000","440000","Value adjustment of non-current intangible assets (acc. 018)","expense","False","Vrijednosno usklađenje dugotrajne nematerijalne imovine (018)"
"hr_441000","441000","Value adjustment of property, plant and equipment","expense","False","Vrijednosno usklađenje dugotrajne materijalne imovine"
"hr_442000","442000","Value adjustment of non-current receivables (acc. 078)","expense","False","Vrijednosno usklađenje dugotrajnih potraživanja (veza sa 078)"
"hr_444000","444000","Value adjustment of deposit on banks, bills of exchange, cheques (acc.109 and acc. 119)","expense","False","Vrijednosno usklađenje depozita u bankama, mjenica, čekova i sl. (109 i dio 119)"
"hr_445000","445000","Value adjustment of current receivables","expense","False","Vrijednosno usklađenje kratkotrajnih potraživanja"
"hr_446000","446000","Value adjustment of inventory","expense","False","Vrijednosno usklađenje zaliha (veza s 319, 329, 359 i 369)"
"hr_447000","447000","Value adjustment of advance payments","expense","False","Vrijednosno usklađenje danih predujmova"
"hr_448000","448000","Value adjustment of receivables from bankruptcy","expense","False","Vrijednosno usklađenje potraživanja iz predstečajne nagodbe"
"hr_449000","449000","Inventory write off","expense","False","Otpis zaliha materijala"
"hr_450000","450000","Long-term provision for pensions, severance payments and other employment benefits","expense","False","Troškovi dugoročnog rezerviranja za mirovine, otpremnine i sl. obveze (čl. 11. st. 2. ZoPD-a)"
"hr_451000","451000","Long-term provision for tax obligations","expense","False","Rezerviranja za porezne obveze"
"hr_452000","452000","Long term provision for litigation losses","expense","False","Troškovi dugoročnog rezerviranja za gubitke po započetim sudskim sporovima (čl. 11. st. 2. ZoPD-a)"
"hr_453000","453000","Long-term provision for renewal of natural resources","expense","False","Troškovi dugoročnog rezerviranja za obnovu prirodnog bogatstva (čl. 11. st. 2. ZoPD-a)"
"hr_454000","454000","The cost of long term provision for risks within warranty period","expense","False","Troškovi dugoročnog rezerviranja za rizike u jamstvenom (garancijskom) roku (čl. 11. st. 2. ZoPD-a)"
"hr_455000","455000","The costs of long-term provision for Company's restructuring","expense","False","Troškovi dugoročnog rezerviranja za restrukturiranje poduzeća (MRS 37, t. 72. i HSFI t. 16.22)"
"hr_456000","456000","The costs of provision for onerous contracts","expense","False","Troškovi rezerviranja po štetnim ugovorima (HSFI 16.21.)"
"hr_457000","457000","The costs of other provision for other risk and charges","expense","False","Troškovi ostalih dugoročnih rezerviranja i troškovi rizika"
"hr_460000","460000","Daily allowances for business trips and other travel expense","expense","False","Dnevnice za službena putovanja i putni troškovi"
"hr_461000","461000","Cost reimbursements, allowances and scholarships","expense","False","Nadoknade troškova, darovi i potpore"
"hr_462000","462000","Severance payments, gifts, performance awards, grants, insurance premiums and jubilee awards","expense","False","Troškovi članova uprave"
"hr_463000","463000","Cost of internal representation and promotion","expense","False","Troškovi reprezentacije i promidžbe (interne)"
"hr_464000","464000","Insurance premiums","expense","False","Premije osiguranja"
"hr_465000","465000","Bank charges and costs of payment operations","expense","False","Bankovne usluge i troškovi platnog prometa"
"hr_466000","466000","Fees, compensations and other expense","expense","False","Članarine, nadoknade i slična davanja"
"hr_467000","467000","Taxes which does not depend on results and charges","expense","False","Porezi koji ne ovise o dobitku i pristojbe"
"hr_468000","468000","Cost of usage rights (excluding leases) and expense of board members","expense","False","Troškovi prava korištenja (osim najmova)"
"hr_469000","469000","Other operating costs- intangible","expense","False","Ostali troškovi poslovanja - nematerijalni"
"hr_470000","470000","Interest expense from Group companies","expense","False","Kamate s poduzetnicima unutar grupe"
"hr_471000","471000","Foreign exchange difference and other expense in a Group","expense","False","Tečajne razlike i drugi rashodi s poduzetnicima unutar grupe"
"hr_472000","472000","Other costs from a Group companies","expense","False","Ostali troškovi s poduzetnicima unutar grupe"
"hr_473000","473000","Interest expense from operations with third party","expense","False","Kamate iz odnosa s nepovezanim poduzetnicima"
"hr_474000","474000","Penalty interest","expense","False","Zatezne kamate"
"hr_475000","475000","Exchange difference on translation of foreign operations with third party","expense","False","Tečajne razlike iz odnosa s nepovezanim poduzetnicima"
"hr_476000","476000","Loss from sale of investments in shares, stakes, bonds and other securities (which are sold below cost)","expense","False","Gubitci iz ulaganja u dionice, udjele, obveznice i dr. vrijednosne papire (koji su prodani ispod troška nabave - čl. 10. ZoPD)"
"hr_477000","477000","Cost of discounts given","expense","False","Mjesta i nositelji troškova"
"hr_478000","478000","Unrealized loss (expense) from financial assets","expense","False","Nerealizirani gubitci (rashodi) od financ. imovine"
"hr_478200","478200","Unrealized losses (expenses) from financial assets","expense","False","Nerealizirani gubici (rashodi) od financijske imovine"
"hr_479000","479000","Other financial expense","expense","False","Ostali financijski troškovi"
"hr_480000","480000","Additional rebates , discounts, claims and costs of samples","expense","False","Trošak naknadnih popusta, sniženja, reklamacija i troškovi uzoraka"
"hr_481000","481000","Bad debts and other assets write off","expense","False","Otpisi vrijednosno neusklađenih potraživanja"
"hr_482000","482000","Disposals of property, plant and equipment and intangible assets","expense","False","Rashodi - otpisi nematerijalne i materijalne imovine"
"hr_483000","483000","Inventory shortages and burglary costs","expense","False","Manjkovi i provalne krađe na zalihama i drugim sredstvima"
"hr_484000","484000","Penalties, compensation for damages occurred and contract related costs","expense","False","Kazne, penali, nadoknade šteta i troškovi iz ugovora"
"hr_485000","485000","Additionally found operating expense","expense","False","Naknadno utvrđeni troškovi poslovanja"
"hr_486000","486000","Gifts/donations up to 2% of total income","expense","False","Darovanje do 2% od ukupnog prihoda"
"hr_487000","487000","Gifts/donations above 2% of total income and other donations","expense","False","Darovanja iznad 2% od UP i dr. darovanja"
"hr_488000","488000","Expense from other activities","expense","False","Troškovi iz drugih aktivnosti (izvan osnovne djelatnosti)"
"hr_489000","489000","Other costs-expense","expense","False","Ostali troškovi - rashodi"
"hr_490000","490000","Allocation of costs to cost of conversion (cost of production) - (on acc. 60, 62, 63)","expense","False","Raspored troškova za obračun proizvoda i usluga (prema HSFI 10 i MRS-u 2 i MRS-u 11) - uskladištivi troškovi (na račune 60, 62 i 63)"
"hr_491000","491000","Allocation of management and administrative overheads to expense for the year - (on acc. 72)","expense","False","Raspored troškova za pokriće upravnih, administrativnih, prodajnih i drugih troškova (na račune 70 i 71)"
"hr_500000","500000","Material used in management and administrative overheads","expense","False","Materijal u troškovima uprave i administracije"
"hr_501000","501000","Wages and other costs of administration","expense","False","Plaće u troškovima uprave i prodaje"
"hr_502000","502000","Depreciation used in management and administrative overheads","expense_depreciation","False","Amortizacija u troškovima uprave i administracije"
"hr_505000","505000","Other costs related to the administration and selling","expense","False","Drugi troškovi uprave i prodaje (Razrada npr. dnevnica, osiguranje, bankovni troškovi, reprezentacija itd.)"
"hr_510000","510000","Material used during selling process in total selling costs","expense","False","Materijal u troškovima prodaje"
"hr_511000","511000","Wages in selling costs","expense","False","Plaće u troškovima prodaje"
"hr_512000","512000","Depreciation in selling costs","expense_depreciation","False","Amortizacija u troškovima prodaje"
"hr_513000","513000","Rents in the cost of sales","expense","False","Zakupnine u troškovima prodaje"
"hr_515000","515000","Other costs related to the deliverables, invoicing and marketing, packaging, export","expense","False","Drugi troškovi prodaje (razrada npr. otpreme pošiljaka, fakturiranje, propaganda, naplata ambalaža, omotni papir, troškovi izvoza, troškovi posredništva itd.)"
"hr_590000","590000","Allocation of costs to cost of conversion (cost of production) - (on acc. 60, 62, 63)","expense","False","Raspored troškova za obračun proizvoda i usluga (prema HSFI 10 i MRS-u 2 i MRS-u 11) - uskladištivi troškovi (na račune 60, 62 i 63)"
"hr_591000","591000","Allocation of costs to expenditures - to be shown in this year income statement - (on acc. 72)","expense","False","Raspored troškova u rashode na teret prihoda"
"hr_600000","600000","Work in progress (allocation by cost driver, cost centres, working orders)","asset_current","False","Proizvodnja u tijeku (razrada po serijama, nositeljima troškova, mjestima, pogonima, gradilištima, objektima, radnim nalozima i sl.)"
"hr_601000","601000","Work in progress for services","asset_current","False","Vrijednost usluga (u tijeku ili nedovršenih na datum bilance - MRS 2, t. 16.)"
"hr_602000","602000","Production in cooperation","asset_current","False","Vanjska proizvodnja (kooperacija i dr.)"
"hr_605000","605000","Production process in free trade zone","asset_current","False","Proizvodnja u slobodnoj zoni"
"hr_606000","606000","Production in processing and manipulation phases","asset_current","False","Proizvodnja u doradi i manipulaciji"
"hr_607000","607000","Suspended production","asset_current","False","Obustavljena proizvodnja"
"hr_608000","608000","Work in progress under partnership agreement","asset_current","False","Proizvodnja u tijeku iz ortačkog ugovora"
"hr_609000","609000","Value adjustments of work in progress","asset_current","False","Vrijednosno usklađivanje proizvodnje - usluga"
"hr_610000","610000","Semi-manufacture goods on stock (analytics by the production stage)","asset_current","False","Zalihe poluproizvoda (analitika prema osnovnim skupinama ili po stupnju dovršenosti)"
"hr_611000","611000","Unfinished goods and semi-manufacture goods (analytics by products types)","asset_current","False","Nedovršeni proizvodi i poluproizvodi (analitika po vrstama proizvoda)"
"hr_619000","619000","Value adjustment of unfinished goods and semi-manufacture goods","asset_current","False","Vrijednosno usklađivanje nedovršenih proizvoda i poluproizvoda"
"hr_620000","620000","Biological assets in progress","asset_current","False","Zalihe biološke proizvodnje u toku"
"hr_621000","621000","Biological assets for sale","asset_current","False","Zaliha biološke imovine za prodaju"
"hr_629000","629000","Value adjustment of biological assets","asset_current","False","Vrijednosno usklađenje biološke imovine"
"hr_630000","630000","Finished goods in warehouse","asset_current","False","Gotovi proizvodi na skladištu (razrada za svako skladište a unutar toga po skupinama, tipovima, vrstama i sl. te poljoprivredni gotovi proizvodi)"
"hr_631000","631000","Finished goods in public warehouse","asset_current","False","Gotovi proizvodi u javnom skladištu, silosu, i dr."
"hr_632000","632000","Finished goods given in commission sale","asset_current","False","Gotovi proizvodi dani u komisijsku prodaju"
"hr_633000","633000","Finished goods given in consignment sale","asset_current","False","Gotovi proizvodi dani u konsignacijsku prodaju"
"hr_634000","634000","Finished goods in processing, finishing and manipulation phases","asset_current","False","Gotovi proizvodi u doradi, obradi i manipulaciji"
"hr_635000","635000","Finished goods in free-trade zone","asset_current","False","Gotovi proizvodi u slobodnoj zoni"
"hr_636000","636000","Obsolete and slow moving inventory","asset_current","False","Zalihe nekurentnih proizvoda i otpadaka"
"hr_637000","637000","Finished goods in storefront","asset_current","False","Gotovi proizvodi u izložbenim prostorima"
"hr_638000","638000","Finished goods from partnership agreement","asset_current","False","Gotovi proizvodi iz ortaštava"
"hr_639000","639000","Value adjustment of finished goods","asset_current","False","Vrijednosno usklađivanje zaliha gotovih proizvoda"
"hr_640000","640000","Finished goods in own stores","asset_current","False","Gotovi proizvodi u prodaji u vlastitim prodavaonicama"
"hr_641000","641000","VAT included in value of goods","asset_current","False","Uračunani PDV u vrijednosti proizvoda"
"hr_648000","648000","Margin included in sale price of finished goods","asset_current","False","Uračunana marža u prodajnoj cijeni gotovih proizvoda"
"hr_649000","649000","Value adjustment of finished goods in own stores","asset_current","False","Vrijednosno usklađivanje gotovih proizvoda u prodavaonicama"
"hr_660000","660000","Merchandise goods on stock","asset_current","False","Roba u skladištu"
"hr_661000","661000","Merchandise goods in others warehouse and in storefront","asset_current","False","Roba u tuđem skladištu i izlozima"
"hr_662000","662000","Merchandise goods given in commission and consignment sale","asset_current","False","Roba dana u komisijsku ili konsignacijsku prodaju"
"hr_663000","663000","Merchandise goods in stores","asset_current","False","Roba u prodavaonicama"
"hr_664000","664000","Tax included in value of merchandise goods","asset_current","False","Uračunani porezi u robi"
"hr_665000","665000","Merchandise goods placed in tax-free warehouse or in free-trade area","asset_current","False","Roba u poreznom skladištu ili u slobodnoj zoni"
"hr_666000","666000","Finished goods in processing, finishing and manipulation phases","asset_current","False","Gotovi proizvodi u doradi, obradi i manipulaciji"
"hr_667000","667000","Merchandise goods in transit","asset_current","False","Roba na putu"
"hr_668000","668000","Margin included in sale price of merchandise goods","asset_current","False","Uračunana razlika u cijeni robe"
"hr_669000","669000","Value adjustment of merchandise goods","asset_current","False","Vrijednosno usklađivanje zaliha robe"
"hr_670000","670000","Advance payments for merchandise goods","asset_current","False","Dani predujmovi za nabavu robe"
"hr_671000","671000","Advance payments for merchandise goods import","asset_current","False","Dani predujmovi uvozniku za nabavu robe"
"hr_672000","672000","Advance payments to related party for merchandise goods","asset_current","False","Dani predujmovi za robu povezanom društvu"
"hr_673000","673000","Advances for biological asset purchases","asset_current","False","Dani predujmovi za nabavu bioloških proizvoda"
"hr_679000","679000","Value adjustment of advance payments for merchandise goods","asset_current","False","Vrijednosno usklađenje danih predujmova za robu"
"hr_680000","680000","Property which is classified as held for sale- at cost","asset_current","False","Nabavna vrijednost nekretnina za preprodaju (s porezom na promet)"
"hr_681000","681000","Additions on the property held for sale","asset_current","False","Umjetnine u prodaji"
"hr_682000","682000","Arts held for sale","asset_current","False","Umjetnine u prodaji"
"hr_683000","683000","Property which is classified as held for sale","asset_current","False","Nekretnine za prodaju (uređene)"
"hr_684000","684000","Old timers for trading","asset_current","False","Oldtimeri za trgovanje"
"hr_685000","685000","VAT included in arts and old timers","asset_current","False","Uračunani PDV u umjetninama i oldtimerima"
"hr_687000","687000","Advance payments for the purchase of property held for sale","asset_current","False","Predujmovi za kupnju nekretnina i sl radi daljnje prodaje"
"hr_688000","688000","Margin included in the sale price of property and arts held for sale","asset_current","False","Uračunana razlika u cijeni nekretnina i umjetnina za daljnju prodaju"
"hr_689000","689000","Value adjustment of real estate and works of art, etc. in transactions and advances","asset_current","False","Vrijednosno usklađivanje nekretnina i umjetnina i sl u prometu i predujmova"
"hr_690000","690000","Intangible assets held for sale","asset_non_current","False","Nematerijalna imovina namijenjena prodaji"
"hr_691000","691000","Other property, plant and equipment held for sale","asset_non_current","False","Materijalna imovina namijenjena za prodaju"
"hr_699000","699000","Impairment of property, plant and equipment held for sale","asset_non_current","False","Vrijednosno usklađenje dugotrajne imovine namijenjena prodaji"
"hr_700000","700000","Cost of goods sold","expense","False","Trošak zaliha prodanih proizvoda (60, 62, 63 i 64)"
"hr_701000","701000","Costs of rendering services","expense","False","Troškovi realiziranih usluga (490 i 601)"
"hr_702000","702000","Costs of unused capacity","expense","False","Troškovi neiskorištenog kapaciteta (HSFI t. 10.18. i MRS 2 t. 13)"
"hr_703000","703000","Cost of sale of material and obsolete inventory","expense","False","Troškovi zaliha materijala i otpadaka (31, 32, 35 i 36)"
"hr_704000","704000","Value adjustment of inventory","expense","False","Vrijednosno usklađenje zaliha (veza s 319, 329, 359 i 369)"
"hr_705000","705000","Expense which relates to previous periods- by mistake not show in previous periods","expense","False","Greškom neiskazani rashodi proteklih razdoblja"
"hr_706000","706000","Construction contracts losses","expense","False","Gubitci iz ugovora o izgradnji (MRS 11, t. 36.)"
"hr_707000","707000","Costs in relation with partnership agreement","expense","False","Troškovi iz ugovora o ortaštvu"
"hr_708000","708000","Cost of value adjustments of work in progress, semi-finished goods and finished goods","expense","False","Troškovi vrijednosnog usklađenja proizvodnje u tijeku (609), poluproizvoda (629) i zaliha gotovih proizvoda (639 i 649)"
"hr_710000","710000","Costs of goods sold","expense","False","Nabavna vrijednost prodane robe"
"hr_711000","711000","Cost of property which is classified as held for sale","expense","False","Trošak imovine koja je klasificirana kao namijenjena prodaji"
"hr_712000","712000","Costs which relates to property, plant and equipment held for sale","expense","False","Troškovi dugotr. imov. namijenjeni prodaji (neto)"
"hr_713000","713000","Costs of goods shortages ( due to evaporate), damages , breakage and goods write off","expense","False","Troškovi kala, rastepa, kvara i loma na robi i otpisi robe"
"hr_714000","714000","Costs of replacement of goods within the warranty period","expense","False","Troškovi zamjene robe u jamstvenom roku"
"hr_715000","715000","Cost of good sold which relates to previous periods- by mistake not show in previous periods","expense","False","Greškom neiskazani rashodi prodane robe u proteklim razdobljima u trgovini"
"hr_718000","718000","Costs of value adjustments of merchandise goods and advance payments","expense","False","Troškovi vrijednosnog usklađenja trgovačke robe i predujmova (669, 679, 689)"
"hr_719000","719000","Impairment losses recognized on property, plant and equipment held for sale","expense","False","Troškovi vrijednosnog usklađenja dugotrajne imovine namijenjene prodaji (699)"
"hr_720000","720000","Cost of management, selling costs and administrative overheads","expense","False","Troškovi uprave, prodaje, administracije (491)"
"hr_721000","721000","Other operational expense","expense","False","Ostali poslovni rashodi - nespomenuti"
"hr_730000","730000","Loss on sale of property, plant and equipment (HSFI 8 t. 8.35.)","expense","False","Rashodi od prodaje dugotrajne imovine (HSFI 8 t. 8.35.)"
"hr_731000","731000","Expense based on material and unexpected asset disposals","expense","False","Otpisi od otuđenja imovine koji su nastali neočekivano i u visokoj vrijednosti (HSFI 4 t. 4.7. i MRS 10, t. 9.)"
"hr_732000","732000","Losses due to dispossession or natural disasters on material assets","expense","False","Gubitci zbog izvlaštenja ili zbog prirodnih katastrofa na važnom dijelu imovine"
"hr_733000","733000","Expense based on material and unexpected events","expense","False","Rashodi iz ostalih rijetkih i neobičnih događaja ili transakcija"
"hr_734000","734000","Other expense due to penalties, paid compensations and liabilities from subsequent events","expense","False","Izvanredne kazne, penali, odštete, naknadno utvrđ. obveze i sl."
"hr_735000","735000","Unrealised losses","expense","False","Nerealizirani gubitci"
"hr_736000","736000","The costs of goods sold in previous years (returned in this year)","expense","False","Troškovi vraćene robe prodane u prethodnim god."
"hr_737000","737000","Losses from biological assets arising from valuation","expense","False","Gubitci od procjene biološke imovine"
"hr_738000","738000","Losses from agricultural assets arising from valuation","expense","False","Gubitci od procjene poljoprivrednih proizvoda"
"hr_740000","740000","Share in the profit from the company connected by participating interest","income","False","Udio u dobiti od društva povezanog sudjelujućim interesom"
"hr_740100","740100","Share in joint profits undertaking","income","False","Udio u zajedničkoj dobiti poduzeća"
"hr_745000","745000","Share in the loss from the company connected by participating interest","income","False","Udio u gubitku društva povezanog sudjelujućim interesom"
"hr_745100","745100","Share of loss from joint undertaking","income","False","Udio gubitka od zajedničkog poduzeća"
"hr_750000","750000","Sale of goods (analytics by goods or profit centres)","income","False","Prihodi od prodaje proizvoda (analitika po proizvodima ili profitnim centrima)"
"hr_751000","751000","Revenue from the rendering of services","income","False","Prihodi od prodaje usluga"
"hr_752000","752000","Revenue from the rendering of construction services- construction services agreement","income","False","Prihodi od graditeljskih usluga - iz ugovora o izgradnji (građevina)"
"hr_753000","753000","Sales income from goods- foreign","income","False","Prihodi od prodaje dobara u inozemstvo"
"hr_754000","754000","Sales income from services - foreign","income","False","Prihodi od prodaje usluga u inozemstvo"
"hr_755000","755000","Sale income from internal usage of own goods and services","income","False","Prihodi s osnove upotrebe proizvoda i usluga za vlastite potrebe"
"hr_756000","756000","Revenue from leases and rents","income","False","Prihodi od najmova i najamnina"
"hr_757000","757000","Revenues form partnership agreement","income","False","Prihodi iz ortaštva"
"hr_758000","758000","Other income from other sale","income","False","Ostali prihodi od ostale prodaje"
"hr_759000","759000","Sale income from Group companies","income","False","Prihodi od prodaje proizvoda i usluga poduzetnicima unutar grupe"
"hr_760000","760000","Revenue from sale of goods","income","False","Prihodi od prodaje robe"
"hr_761000","761000","Sales income from goods- foreign","income","False","Prihodi od prodaje dobara u inozemstvo"
"hr_762000","762000","Revenue from the rendering of merchandise services","income","False","Prihodi od trgovačkih usluga"
"hr_763000","763000","Income from sale of slow-moving inventory","income","False","Prihodi od prodaje nekurentne robe (robe u kvaru, oštećenju, demodirana i sl.)"
"hr_764000","764000","Income from sale of goods on commodity loan granted","income","False","Prihodi od prodaje robe na robni kredit"
"hr_765000","765000","Sale income from internal usage of own merchandise goods","income","False","Prihodi s osnove upotrebe robe za vlastite potrebe"
"hr_766000","766000","Income from sale of goods on financial leasing","income","False","Prihodi od prodane robe u financijski lizing (najam)"
"hr_767000","767000","Income from sale of property, plant and equipment and arts (held for sale)","income","False","Prihodi od preprodaje nekretnina i umjetnina"
"hr_768000","768000","Other income from sale of goods and rendering of merchandise services","income","False","Ostali prihodi od prodaje roba i trgovačkih usluga"
"hr_769000","769000","Sale income from Group companies","income","False","Prihodi od prodaje proizvoda i usluga poduzetnicima unutar grupe"
"hr_770000","770000","Income from investments in shares (shares) of entrepreneurs within the group","income","False","Prihodi od ulaganja u udjele (udjele) poduzetnika unutar grupe"
"hr_770100","770100","Income from other long-term financial investments and loans to entrepreneurs within the group","income","False","Prihodi od ostalih dugoročnih financijskih ulaganja i kredita poduzetnicima unutar grupe"
"hr_770200","770200","Other income based on interest from relations with entrepreneurs within the group","income","False","Ostali prihodi po osnovi kamata iz odnosa s poduzetnicima unutar grupe"
"hr_770300","770300","Exchange rate differences and other financial income from relations with entrepreneurs within the group","income","False","Tečajne razlike i ostali financijski prihodi iz odnosa s poduzetnicima unutar grupe"
"hr_771000","771000","Interest income","income","False","Prihodi od kamata"
"hr_772000","772000","Income from exchange difference on translation of foreign operations and other income","income","False","Prihodi od tečajnih razlika i ostali prihodi"
"hr_773000","773000","Income from dividends and share in result","income","False","Prihodi od dividende i dobitaka iz udjela"
"hr_774000","774000","Financial income from long term financial asset and loans given","income","False","Prihodi od ostalih dugotrajnih financijskih ulaganja i zajmova"
"hr_775000","775000","Dividends income from associated undertakings","income","False","Prihodi od ulaganja u udjele (dionice) društava povezanih sudjelujućim interesom"
"hr_776000","776000","Unrealized gains from financial assets","income","False","Nerealizirani dobitci (prihodi) od financijske imovine"
"hr_777000","777000","Income from negative goodwill","income","False","Prihodi negativnog goodwilla"
"hr_778000","778000","Financial fees income","income","False","Prihodi od financijskih naknada"
"hr_779000","779000","Other financial income","income","False","Ostali financijski prihodi"
"hr_780000","780000","Income from liability write off and from discounts received","income_other","False","Prihodi od otpisa obveza i popusta"
"hr_781000","781000","Net gain on disposal of assets, surpluses and evaluation","income_other","False","Prihodi od rezidualnih imovinskih stavki, viškova i procjena"
"hr_782000","782000","Income from reversal of value adjustment and provisions and collected receivables written off","income_other","False","Prihodi od ukidanja rezerviranja i naknadno naplaćeni prihodi"
"hr_783000","783000","Income from grants, subsidies and compensations","income_other","False","Prihodi od refundacije, dotacija, subvencija i nadoknada"
"hr_780900","780900","Income from the write-off of liabilities to entrepreneurs within the group","income_other","False","Prihodi od otpisa obveza prema poduzetnicima unutar grupe"
"hr_783900","783900","Income from grants and compensation from entrepreneurs within the group","income_other","False","Prihodi od potpora i naknada od poduzetnika unutar grupe"
"hr_784000","784000","Income from losses and revaluations reversals","income_other","False","Prihodi od ukidanja gubitaka i revalorizacija"
"hr_785000","785000","Other business income","income_other","False","Ostali poslovni prihodi"
"hr_786000","786000","Income from government grants","income_other","False","Prihodi od državnih potpora (HSFI 15 i MRS 20)"
"hr_787000","787000","Gains from biological assets arising from valuation","income_other","False","Dobitci od procjene poljoprivrednih proizvoda i biološke imovine (HSFI 17, t. 17.12 i MRS 41)"
"hr_788000","788000","Revenues from the sale of a significant part of the assets","income_other","False","Prihodi od prodaje značajnog dijela imovine (društva, pogona, gosp. cjeline)"
"hr_789000","789000","Other income (free deliverables, non-operational income)","income_other","False","Ostali prihodi (npr. veliki besplatni primitak, prihod koji nije proizašao iz redovitog poslovanja)"
"hr_803000","803000","Income tax expense","expense","False","Porez na dobitak (gubitak)"
"hr_900000","900000","Issued capital - paid in","equity","False","Upisani temeljni kapital koji je plaćen"
"hr_901000","901000","Issued capital - non controlling interests","equity","False","Upisani temeljni kapital manjinskih članova"
"hr_902000","902000","Issued capital- not paid in","equity","False","Upisani kapital koji nije plaćen"
"hr_903000","903000","Shareholders share in public limited company capital","equity","False","Kapital (ulozi) članova javnog trgovačkog društva"
"hr_904000","904000","Partners business stakes in limited partnership","equity","False","Kapital (ulozi) komanditora komanditnog društva"
"hr_905000","905000","Government capital in company","equity","False","Državni kapital u udjelima"
"hr_906000","906000","Members business stake in cooperative capital","equity","False","Kapital članova zadruge"
"hr_910000","910000","Paid in stake/share above issued share capital","equity","False","Uplaćeni udjeli - dionice iznad svote temeljnog kapitala"
"hr_911000","911000","Capital reserves from additional payments in order to gain special rights in the Company (convertible bonds)","equity","False","Kapitalne pričuve iz dodatnih uplata radi stjecanja posebnih prava u društvu (ili zamjenjivih obveznica)"
"hr_912000","912000","Capital reserves from the additional payments of equity holders","equity","False","Kapitalne pričuve iz uplata dodatnih činidbi"
"hr_913000","913000","Decrease of share capital transferred to capital reserve","equity","False","Kapitalne pričuve iz ostatka pri smanjenju temeljnog kapitala"
"hr_914000","914000","Capital reserves from other sources","equity","False","Kapitalne pričuve iz drugih izvora"
"hr_915000","915000","Capital gain from sale of treasury shares - stakes","equity","False","Kapitalni dobitak iz prodaje vlastitih udjela - dionica"
"hr_916000","916000","Capital gains on the sale of shares issued","equity","False","Kapitalni dobitak na prodane emitirane dionice"
"hr_917000","917000","Investments in silent company transferred to capital reserves","equity","False","Kapitalne pričuve iz ulaganja tajnog člana društva (čl. 148. ZTD-a)"
"hr_918000","918000","Capital loss from sale of treasury shares - stakes","equity","False","Kapitalni gubitci iz vlastitih dionica i udjela"
"hr_919000","919000","Investments of entrepreneur transferred to capital reserves","equity","False","Kapital iz ulaganja obrtnika dobitaša"
"hr_920000","920000","Legal reserves","equity","False","Zakonske pričuve"
"hr_921000","921000","Reserves for treasury shares","equity","False","Pričuve za vlastite dionice i udjele (čl. 233. i 406.a ZTD-a)"
"hr_922000","922000","Treasury shares and stakes","equity","False","Vlastite dionice i udjeli (odbitna- dugovna stavka)"
"hr_923000","923000","Statutory reserves","equity","False","Statutarne pričuve"
"hr_924000","924000","Other reserves","equity","False","Ostale pričuve"
"hr_930000","930000","Revaluation reserves from tangible and intangible assets","equity","False","Revalorizacijske pričuve s osnove revalorizacije dugotrajne materijalne i nematerijalne imovine"
"hr_931000","931000","Fair value of available-for-sale financial assets","equity","False","Fer vrijednost financijske imovine raspoložive za prodaju"
"hr_931200","931200","An effective part of cash flow protection","equity","False","Učinkovit dio zaštite novčanog toka"
"hr_931300","931300","An effective part of the protection of net investment abroad","equity","False","Učinkovit dio zaštite neto ulaganja u inozemstvu"
"hr_932000","932000","Foreign currency from foreign translation","equity","False","Tečajne razlike iz preračuna inozemnog poslovanja"
"hr_933000","933000","Other accumulated comprehensive income / loss","equity","False","Udio u ostaloj sveobuhvatnoj dobiti/gubitku društava povezanih sudjelujućim interesom"
"hr_934000","934000","Actuarial gains/losses based on employment benefits","equity","False","Aktuarski dobici/gubici po planovima definiranih primanja"
"hr_935000","935000","Other changes in capital (minorities)","equity","False","Ostale nevlasničke promjene kapitala"
"hr_940000","940000","Retained earnings","equity","False","Zadržani dobitak (iz prethodnih godina)"
"hr_941000","941000","Accumulated loss","equity","False","Preneseni gubitak (kumuliran u prethodnim godinama - analitika po članovima)"
"hr_960000","960000","Non-controlling interest","equity","False","Manjinski interes"
"hr_990000","990000","Property plant and equipment in use","off_balance","False","Imovina - materijalna u optjecaju"
"hr_991000","991000","Rights and receivables to be used","off_balance","False","Prava i tražbine"
"hr_992000","992000","Securities","off_balance","False","Vrijednosni papiri"
"hr_993000","993000","Securities in circulation","off_balance","False","Vrijednosnice u manipulaciji"
"hr_994000","994000","Calculation of profit from investment in securities","off_balance","False","Obračun dobitka investicijskog pothvata"
"hr_995000","995000","Source of property plant and equipment","off_balance","False","Izvori materijalne imovine"
"hr_996000","996000","Source of rights and obligations","off_balance","False","Izvori prava i obveze"
"hr_997000","997000","Liabilities for securities which are not in circulation","off_balance","False","Obveze za vrijednosne papire koji nisu stavljeni u optjecaj"
"hr_998000","998000","Liabilities for securities in circulation","off_balance","False","Obveze za izdane vrijednosnice u manipulaciji"
"hr_999000","999000","Liabilities due to investment project","off_balance","False","Obveze s osnove investicijskog pothvata"

```

## File: data\template\account.fiscal.position-hr.csv

```csv
"id","name","sequence","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@hr"
"fiscal_position_hr_exempt","Exempt taxpayer","","","","","","VAT_S_IN_ROC_5","VAT_S_other_exempt_O","Oslobođeni porezni obveznik"
"","","","","","","","VAT_S_IN_ROC_13","VAT_S_other_exempt_O",""
"","","","","","","","VAT_S_IN_ROC_25","VAT_S_other_exempt_O",""
"","","","","","","","VAT_S_IN_ROC_5","VAT_P_O",""
"","","","","","","","VAT_S_IN_ROC_13","VAT_P_O",""
"","","","","","","","VAT_S_IN_ROC_25","VAT_P_O",""
"fiscal_position_hr_national","Domestic","10","1","1","base.hr","","","","Domaći"
"fiscal_position_hr_person_private","Partner private","20","1","","","base.europe","","","Partner privatno"
"fiscal_position_hr_eu","EU partner","30","1","1","","base.europe","VAT_S_IN_ROC_5","VAT_S_EU_G","ja partner"
"","","","","","","","VAT_S_IN_ROC_13","VAT_S_EU_G",""
"","","","","","","","VAT_S_IN_ROC_25","VAT_S_EU_G",""
"","","","","","","","VAT_P_IN_ROC_5","VAT_P_G_IN_EU_5",""
"","","","","","","","VAT_P_IN_ROC_13","VAT_P_G_IN_EU_13",""
"","","","","","","","VAT_P_IN_ROC_25","VAT_P_G_IN_EU_25",""
"fiscal_position_hr_eu_out","Partner outside the EU","40","1","","","","VAT_S_IN_ROC_5","VAT_S_EX_O","Partner izvan EU"
"","","","","","","","VAT_S_IN_ROC_13","VAT_S_EX_O",""
"","","","","","","","VAT_S_IN_ROC_25","VAT_S_EX_O",""
"","","","","","","","VAT_P_IN_ROC_5","VAT_P_NOT_IN_EU_5",""
"","","","","","","","VAT_P_IN_ROC_13","VAT_P_NOT_IN_EU_13",""
"","","","","","","","VAT_P_IN_ROC_25","VAT_P_NOT_IN_EU_25",""

```

## File: data\template\account.group-hr.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@hr"
"hr_group_0","0","","Receivables for issued capital and non-current assets","Potraživanja za upisani kapital i dugotrajna imovina"
"hr_group_00","00","","Receivables for recorded but not paid in capital and non-current financial assets","Potraživanja za upisani a neuplaćeni kapital"
"hr_group_01","01","","Intangible asset","Nematerijalna imovina"
"hr_group_02","02","","Land and buildings","Materijalna imovina - nekretnine"
"hr_group_03","03","","Plant, equipment, tools and transportation vehicles","Postrojenja, oprema, alati, inventar i transportna sredstva"
"hr_group_04","04","","Biological assets","Biološka imovina"
"hr_group_05","05","","Investment properties","Ulaganja u nekretnine"
"hr_group_06","06","","Non-current financial assets","Dugotrajna financijska imovina (s povratom dužim od jedne godine)"
"hr_group_07","07","","Long term receivables","Potraživanja (dulja od jedne godine)"
"hr_group_08","08","","Deferred tax asset","Odgođena porezna imovina"
"hr_group_1","1","","Cash, current financial assets,current receivables, prepaid expenses and accrued income","Novac, kratkotrajna financijska imovina, kratkotrajna potraživanja, troškovi i prihod budućeg razdoblja"
"hr_group_10","10","","Cash in bank and cash on hand","Novac u bankama i blagajnama"
"hr_group_11","11","","Current financial assets","Kratkotrajna financijska imovina (do jedne godine)"
"hr_group_12","12","","Current receivables","Potraživanja (kratkotrajna)"
"hr_group_13","13","","Receivables from employees and other receivables","Potraživanja od zaposlenih i ostala potraživanja (kratkotrajna)"
"hr_group_14","14","","Receivables from the government for taxes, duties and contributions","Potraživanja od države za poreze, carinu i doprinose"
"hr_group_15","15","","Receivables from government and other institutions","Ostala potraživanja od državnih i drugih institucija"
"hr_group_19","19","","Prepaid expenses and accrued income","Plaćeni troškovi budućeg razdoblja i obračunani budući prihodi"
"hr_group_2","2","","Current and non-current liabilities, long term provisions, accrued expenses and deferred income","Kratkoročne i dugoročne obveze, dugoročna rezerviranja, odgođeno plaćanje i prihodi budućeg razdoblja"
"hr_group_20","20","","Current liabilities towards group companies and associated undertakings","Kratkoročne obveze prema poduzetnicima unutar grupe is osnove udjela u rezultatu (do jedne godine)"
"hr_group_21","21","","Current financial liabilities","Kratkoročne financijske obveze"
"hr_group_22","22","","Account payables, advances received and other payables","Obveze prema dobavljačima, za predujmove i ostale obveze"
"hr_group_23","23","","Liabilities towards employees and other liabilities","Obveze prema zaposlenima i ostale obveze"
"hr_group_24","24","","Current liabilities for taxes, contributions and similar expenses","Kratkoročne obveze za poreze, doprinose i slična davanja"
"hr_group_25","25","","Long term liabilities","Dugoročne obveze (za duže od jedne godine)"
"hr_group_26","26","","Deferred taxes","Odgođeni porezi"
"hr_group_28","28","","Long term provisions for risks and charges","Dugoročna rezerviranja za rizike i troškove"
"hr_group_29","29","","Accrued expenses and deferred income","Odgođeno plaćanje troškova i prihod budućeg razdoblja"
"hr_group_3","3","","Raw materials, materials, spare parts and small inventory","Zalihe sirovina i materijala, rezervnih dijelova i sitnog inventara"
"hr_group_30","30","","Cost of inventory purchasing calculation","Obračun troškova kupnje"
"hr_group_31","31","","Raw materials","Sirovine i materijal na zalihi"
"hr_group_32","32","","Spare parts","Zalihe rezervnih dijelova"
"hr_group_35","35","","Small inventory on stock and tires","Zalihe sitnog inventara i autoguma"
"hr_group_36","36","","Small inventory on stock and tires in use","Zalihe sitnog inventara i autoguma u uporabi"
"hr_group_37","37","","Advance payments for raw materials, spare parts, small inventory and tires purchases","Predujmovi dobavljačima sirovina i materijala, rezervnih dijelova, sitnog inventara i autoguma"
"hr_group_4","4","","Costs by nature , financial and other expenses","Troškovi prema vrstama, financijski i ostali rashodi"
"hr_group_40","40","","Material cost","Materijalni troškovi"
"hr_group_41","41","","Other external costs (cost of services)","Ostali vanjski troškovi (troškovi usluga)"
"hr_group_42","42","","Staff costs-salaries","Troškovi osoblja - plaće"
"hr_group_43","43","","Depreciation","Amortizacija"
"hr_group_44","44","","Value adjustment of non-current and current assets","Vrijednosno usklađenje dugotrajne i kratkotrajne imovine"
"hr_group_45","45","","Provisions","Rezerviranja (v. skupinu 28)"
"hr_group_46","46","","Other operating costs","Ostali troškovi poslovanja"
"hr_group_47","47","","Financial expenses","Financijski rashodi"
"hr_group_48","48","","Other expenses","Ostali poslovni rashodi"
"hr_group_49","49","","Allocation of costs","Raspored troškova"
"hr_group_5","5","","Cost centres and cost driver","Mjesta troška i pokretač troškova"
"hr_group_50","50","","Management and administrative overheads","Troškovi uprave i administracija"
"hr_group_51","51","","Selling costs","Troškovi prodaje"
"hr_group_52","52","","Transportation","TRANSPORT"
"hr_group_53","53","","Management and administrative overheads","Troškovi uprave i administracija"
"hr_group_54","54","","Selling costs","Troškovi prodaje"
"hr_group_59","59","","Allocation of costs","Raspored troškova"
"hr_group_6","6","","Work in progress, biological assets, finished goods, merchandise goods and non-current assets held for sale","Proizvodnja, biološka imovina, gotovi proizvodi, roba i dugotrajna imovina namijenjena prodaji"
"hr_group_60","60","","Work in progress - conversion costs","Proizvodnja - troškovi konverzije"
"hr_group_61","61","","Unfinished goods and semi-manufacture goods","Nedovršeni proizvodi i poluproizvodi"
"hr_group_62","62","","Biological assets","Biološka imovina"
"hr_group_63","63","","Finished goods","Zalihe gotovih proizvoda"
"hr_group_64","64","","Finished goods in own stores","Gotovi proizvodi u prodaji u vlastitim prodavaonicama"
"hr_group_65","65","","Total cost of merchandise goods - total purchasing cost","Obračun troškova nabave robe - troškovi kupnje"
"hr_group_66","66","","Merchandise goods","Roba"
"hr_group_67","67","","Advance payments for merchandise goods","Dani predujmovi za nabavu robe"
"hr_group_68","68","","Properties and arts held for sale","Nekretnine i umjetnine za trgovanje"
"hr_group_69","69","","Non-current assets held for sale","Dugotrajna imovina namijenjena prodaji"
"hr_group_7","7","","Expenses and income for the year","Pokriće rashoda i prihodi razdoblja"
"hr_group_70","70","","Cost of finished goods sold and cost of services","Troškovi prodanih zaliha proizvoda i usluga"
"hr_group_71","71","","Cost of goods sold","Trošak zaliha prodanih proizvoda (60, 62, 63 i 64)"
"hr_group_72","72","","Management and administrative overheads","Troškovi uprave i administracija"
"hr_group_73","73","","Other expenses","Ostali poslovni rashodi"
"hr_group_74","74","","Share in profit/loss of a group and in associated undertaking","Udio u dobitku i gubitku društava povezanih sudjelujućim interesom i od zajedničkih pothvata"
"hr_group_75","75","","Revenue from sale of goods and rendering of services","Prihodi od prodaje proizvoda i usluga"
"hr_group_76","76","","Revenue from sale of merchandise goods","Prihodi od prodaje trgovačke robe"
"hr_group_77","77","","Financial income","Financijski prihodi"
"hr_group_78","78","","Other operational income","Ostali poslovni prihodi"
"hr_group_79","79","","Difference between income and expenses","Razlika prihoda i rashoda financijske godine"
"hr_group_8","8","","Financial result","Financijski rezultat poslovanja"
"hr_group_80","80","","Profit or loss for the year","Dobitak ili gubitak razdoblja"
"hr_group_81","81","","Profit or loss from discontinued operations (applicable for companies which use msfi and have discontinued operations)","Dobitak ili gubitak prekinutog poslovanja prije oporezivanja (koristi samo poduzetnik obveznik primjene msfi-ja koji ima prekinuto poslovanje)"
"hr_group_82","82","","Total profit or loss for the year (applicable for companies which use msfi and have discontinue operations)","Dobit ili gubitak ukupnog poslovanja prije oporezivanja (koristi samo poduzetnik obveznik primjene msfi-a koji ima prekinuto poslovanje)"
"hr_group_83","83","","Profit or loss for the year attributable to others","Dobitak ili gubitak razdoblja koji pripada drugima"
"hr_group_9","9","","Equity and reserves and of the balance sheet items","Kapital i pričuve te izvanbilančni zapisi"
"hr_group_90","90","","Share capital","Temeljni - (upisani) kapital"
"hr_group_91","91","","Capital reserves","Kapitalne pričuve"
"hr_group_92","92","","Reserves from retained earnings","Pričuve iz dobitka"
"hr_group_93","93","","Revaluation reserves and fair value reserves","Revalorizacijske pričuve i pričuve fer vrijednosti"
"hr_group_94","94","","Retained earnings or accumulated loss","Zadržani dobitak ili preneseni gubitak"
"hr_group_95","95","","Profit or loss for the year","Dobitak ili gubitak razdoblja"
"hr_group_96","96","","Non-controlling interests","Manjinski interes"
"hr_group_99","99","","Off-balance sheet balance","Izvanbilančni zapisi"

```

## File: data\template\account.tax-hr.csv

```csv
"id","sequence","description","invoice_label","name","price_include","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@hr"
"VAT_S_IN_ROC_25","1","Sale 25% in country","Sale 25% in country","25%","False","25.0","percent","sale","tax_group_pdv_25","100","base","invoice","+II.3_base","","Rasprodaja 25% u zemlji"
"","","","","","","","","","","100","tax","invoice","+II.3_tax","hr_240000",""
"","","","","","","","","","","100","base","refund","+II.3_base","",""
"","","","","","","","","","","100","tax","refund","+II.3_tax","hr_240000",""
"VAT_S_IN_ROC_13","2","Sale 13% in country","Sale 13% in country","13%","False","13.0","percent","sale","tax_group_pdv_13","100","base","invoice","+II.2_base","","Prodaja 13% u zemlji"
"","","","","","","","","","","100","tax","invoice","+II.2_tax","hr_240000",""
"","","","","","","","","","","100","base","refund","+II.2_base","",""
"","","","","","","","","","","100","tax","refund","+II.2_tax","hr_240000",""
"VAT_S_IN_ROC_5","3","Sale 5% in country","Sale 5% in country","5%","False","5.0","percent","sale","tax_group_pdv_5","100","base","invoice","+II.1_base","","Prodaja 5% u zemlji"
"","","","","","","","","","","100","tax","invoice","+II.1_tax","hr_240000",""
"","","","","","","","","","","100","base","refund","+II.1_base","",""
"","","","","","","","","","","100","tax","refund","+II.1_tax","hr_240000",""
"VAT_S_TAX_PERSON_25%","4","Taxable person not in ROC 25%","Taxable person not in ROC 25%","25% EX","False","25.0","percent","sale","tax_group_pdv_25","100","base","invoice","+II.13_base||+III.13_base","","Porezni obveznik koji nije u ROC 25%"
"","","","","","","","","","","100","tax","invoice","+II.13_tax","hr_240420",""
"","","","","","","","","","","-100","tax","invoice","-III.13_tax","hr_140420",""
"","","","","","","","","","","100","base","refund","+II.13_base||+III.13_base","",""
"","","","","","","","","","","100","tax","refund","+II.13_tax","hr_240420",""
"","","","","","","","","","","-100","tax","refund","-III.13_tax","hr_140420",""
"VAT_S_TAX_PERSON_13%","5","Taxable person not in ROC 13%","Taxable person not in ROC 13%","13% EX","False","13.0","percent","sale","tax_group_pdv_13","100","base","invoice","+II.12_base||+III.12_base","","Porezni obveznik koji nije u ROC 13%"
"","","","","","","","","","","100","tax","invoice","+II.12_tax","hr_240410",""
"","","","","","","","","","","-100","tax","invoice","-III.12_tax","hr_140410",""
"","","","","","","","","","","100","base","refund","+II.12_base||+III.12_base","",""
"","","","","","","","","","","100","tax","refund","+II.12_tax","hr_240410",""
"","","","","","","","","","","-100","tax","refund","-III.12_tax","hr_140410",""
"VAT_S_TAX_PERSON_5%","6","Taxable person not in ROC 5%","Taxable person not in ROC 5%","5% EX","False","5.0","percent","sale","tax_group_pdv_5","100","base","invoice","+II.11_base||+III.11_base","","Porezni obveznik koji nije u ROC 5%"
"","","","","","","","","","","100","tax","invoice","+II.11_tax","hr_240400",""
"","","","","","","","","","","-100","tax","invoice","-III.11_tax","hr_140400",""
"","","","","","","","","","","100","base","refund","+II.11_base||+III.11_base","",""
"","","","","","","","","","","100","tax","refund","+II.11_tax","hr_240400",""
"","","","","","","","","","","-100","tax","refund","-III.11_tax","hr_140400",""
"VAT_S_EU_G","7","Sale goods 0% in EU","Sale goods 0% in EU","0% EU G","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.3_base","","Rasprodaja robe 0% u EU"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.3_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_S_EU_S","8","Sale service 0% in EU","Sale service 0% in EU","0% EU S","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.4_base","","Prodaja usluga 0% u EU"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.4_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_S_EX_O","9","Sale 0% not in EU","Sale 0% not in EU","0% EX","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.9_base","","Rasprodaja 0% izvan EU"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.9_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_S_reverse_O","10","Reverse charge 0%","Reverse charge 0%","0% R C","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.1_base","","Obrnuto zaduženje 0%"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.1_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_S_carried_other_state_O","11","Supplies carried out in another member state","Supplies carried out in another member state","0% EU M","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.2_base","","Isporuke obavljene u drugoj državi članici"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.2_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_S_person_not_in_ROC_O","12","Supplies to persons not established in the republic of croatia","Supplies to persons not established in the republic of croatia","0% EX M","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.5_base","","Opskrba osoba bez sjedišta u Republici Hrvatskoj"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.5_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_S_new_transport_other_state_O","13","Supplies of new means of transport to another member state","Supplies of new means of transport to another member state","0% EU Trans","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.7_base","","Isporuke novih prijevoznih sredstava u drugu državu članicu"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.7_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_S_other_exempt_O","14","Other exempt","Other exempt","0% EXEMPT","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.10_base","","Ostalo izuzeto"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.10_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_P_install_assemb_goods_O","15","0% installed and assembled goods in another state","0% installed and assembled goods in another state","0% EU M INST","False","0.0","percent","sale","tax_group_pdv_0","100","base","invoice","+I.6_base","","0% ugrađena i montirana roba u drugoj državi"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.6_base","",""
"","","","","","","","","","","100","tax","refund","","",""
"VAT_P_IN_ROC_25","16","Purchase 25% in country","Purchase 25% in country","25%","False","25.0","percent","purchase","tax_group_pdv_25","100","base","invoice","+III.3_base","","Kupite 25% u zemlji"
"","","","","","","","","","","100","tax","invoice","+III.3_tax","hr_140000",""
"","","","","","","","","","","100","base","refund","+III.3_base","",""
"","","","","","","","","","","100","tax","refund","+III.3_tax","hr_140000",""
"VAT_P_IN_ROC_13","17","Purchase 13% in country","Purchase 13% in country","13%","False","13.0","percent","purchase","tax_group_pdv_13","100","base","invoice","+II.2_base","","Kupite 13% u zemlji"
"","","","","","","","","","","100","tax","invoice","+II.2_tax","hr_140000",""
"","","","","","","","","","","100","base","refund","+II.2_base","",""
"","","","","","","","","","","100","tax","refund","+II.2_tax","hr_140000",""
"VAT_P_IN_ROC_5","18","Purchase 5% in country","Purchase 5% in country","5%","False","5.0","percent","purchase","tax_group_pdv_5","100","base","invoice","+II.1_base","","Kupite 5% u zemlji"
"","","","","","","","","","","100","tax","invoice","+II.1_tax","hr_140000",""
"","","","","","","","","","","100","base","refund","+II.1_base","",""
"","","","","","","","","","","100","tax","refund","+II.1_tax","hr_140000",""
"VAT_P_G_IN_EU_25","19","Purchase goods 25% in EU","Purchase goods 25% in EU","25% EU G","False","25.0","percent","purchase","tax_group_pdv_25","100","base","invoice","+II.7_base||+III.7_base","","Kupujte robu 25% u EU"
"","","","","","","","","","","-100","tax","invoice","-II.7_tax","hr_240220",""
"","","","","","","","","","","100","tax","invoice","+III.7_tax","hr_140220",""
"","","","","","","","","","","100","base","refund","+II.7_base||+III.7_base","",""
"","","","","","","","","","","-100","tax","refund","-II.7_tax","hr_240220",""
"","","","","","","","","","","100","tax","refund","+III.7_tax","hr_140220",""
"VAT_P_S_IN_EU_25","20","Purchase service 25% in EU","Purchase service 25% in EU","25% EU S","False","25.0","percent","purchase","tax_group_pdv_25","100","base","invoice","+II.10_base||+III.10_base","","Usluga kupnje 25% u EU"
"","","","","","","","","","","-100","tax","invoice","-II.10_tax","hr_240320",""
"","","","","","","","","","","100","tax","invoice","+III.10_tax","hr_140320",""
"","","","","","","","","","","100","base","refund","+II.10_base||+III.10_base","",""
"","","","","","","","","","","-100","tax","refund","-II.10_tax","hr_240320",""
"","","","","","","","","","","100","tax","refund","+III.10_tax","hr_140320",""
"VAT_P_G_IN_EU_13","21","Purchase goods 13% in EU","Purchase goods 13% in EU","13% EU G","False","13.0","percent","purchase","tax_group_pdv_13","100","base","invoice","+II.6_base||+III.6_base","","Kupujte robu 13% u EU"
"","","","","","","","","","","-100","tax","invoice","-II.6_tax","hr_240210",""
"","","","","","","","","","","100","tax","invoice","+III.6_tax","hr_140210",""
"","","","","","","","","","","100","base","refund","+II.6_base||+III.6_base","",""
"","","","","","","","","","","-100","tax","refund","-II.6_tax","hr_240210",""
"","","","","","","","","","","100","tax","refund","+III.6_tax","hr_140210",""
"VAT_P_S_IN_EU_13","22","Purchase service 13% in EU","Purchase service 13% in EU","13% EU S","False","13.0","percent","purchase","tax_group_pdv_13","100","base","invoice","+II.9_base||+III.9_base","","Usluga kupnje 13% u EU"
"","","","","","","","","","","-100","tax","invoice","-II.9_tax","hr_240310",""
"","","","","","","","","","","100","tax","invoice","+III.9_tax","hr_140310",""
"","","","","","","","","","","100","base","refund","+II.9_base||+III.9_base","",""
"","","","","","","","","","","-100","tax","refund","-II.9_tax","hr_240310",""
"","","","","","","","","","","100","tax","refund","+III.9_tax","hr_140310",""
"VAT_P_G_IN_EU_5","23","Purchase goods 5% in EU","Purchase goods 5% in EU","5% EU G","False","5.0","percent","purchase","tax_group_pdv_5","100","base","invoice","+II.5_base||+III.5_base","","Kupujte robu 5% u EU"
"","","","","","","","","","","-100","tax","invoice","-II.5_tax","hr_240200",""
"","","","","","","","","","","100","tax","invoice","+III.5_tax","hr_140200",""
"","","","","","","","","","","100","base","refund","+II.5_base||+III.5_base","",""
"","","","","","","","","","","-100","tax","refund","-II.5_tax","hr_240200",""
"","","","","","","","","","","100","tax","refund","+III.5_tax","hr_140200",""
"VAT_P_S_IN_EU_5","24","Purchase service 5% in EU","Purchase service 5% in EU","5% EU S","False","5.0","percent","purchase","tax_group_pdv_5","100","base","invoice","+II.8_base||+III.8_base","","Usluga kupnje 5% u EU"
"","","","","","","","","","","-100","tax","invoice","-II.8_tax","hr_240300",""
"","","","","","","","","","","100","tax","invoice","+III.8_tax","hr_140300",""
"","","","","","","","","","","100","base","refund","+II.8_base||+III.8_base","",""
"","","","","","","","","","","-100","tax","refund","-II.8_tax","hr_240300",""
"","","","","","","","","","","100","tax","refund","+III.8_tax","hr_140300",""
"VAT_P_NOT_IN_EU_25","25","Purchase 25% not in EU","Purchase 25% not in EU","25% EX","False","25.0","percent","purchase","tax_group_pdv_25","100","base","invoice","+II.15_base||+III.14_base","","Kupite 25% izvan EU"
"","","","","","","","","","","-100","tax","invoice","-II.15_tax","hr_240500",""
"","","","","","","","","","","100","tax","invoice","+III.14_tax","hr_140500",""
"","","","","","","","","","","100","base","refund","+II.15_base||+III.14_base","",""
"","","","","","","","","","","-100","tax","refund","-II.15_tax","hr_240500",""
"","","","","","","","","","","100","tax","refund","+III.14_tax","hr_140500",""
"VAT_P_NOT_IN_EU_13","26","Purchase 13% not in EU","Purchase 13% not in EU","13% EX","False","13.0","percent","purchase","tax_group_pdv_13","100","base","invoice","+II.15_base||+III.14_base","","Kupite 13% izvan EU"
"","","","","","","","","","","-100","tax","invoice","-II.15_tax","hr_240500",""
"","","","","","","","","","","100","tax","invoice","+III.14_tax","hr_140500",""
"","","","","","","","","","","100","base","refund","+II.15_base||+III.14_base","",""
"","","","","","","","","","","-100","tax","refund","-II.15_tax","hr_240500",""
"","","","","","","","","","","100","tax","refund","+III.14_tax","hr_140500",""
"VAT_P_NOT_IN_EU_5","27","Purchase 5% not in EU","Purchase 5% not in EU","5% EX","False","5.0","percent","purchase","tax_group_pdv_13","100","base","invoice","+II.15_base||+III.14_base","","Kupite 5% izvan UE"
"","","","","","","","","","","-100","tax","invoice","-II.15_tax","hr_240500",""
"","","","","","","","","","","100","tax","invoice","+III.14_tax","hr_140500",""
"","","","","","","","","","","100","base","refund","+II.15_base||+III.14_base","",""
"","","","","","","","","","","-100","tax","refund","-II.15_tax","hr_240500",""
"","","","","","","","","","","100","tax","refund","+III.14_tax","hr_140500",""
"VAT_P_reverse_charge","28","Domestic reverse charge","Domestic reverse charge","25% R C","False","25.0","percent","purchase","tax_group_pdv_25","100","base","invoice","+II.4_base||+III.4_base","","Domaći obrnuti teret"
"","","","","","","","","","","-100","tax","invoice","-II.4_tax","hr_240100",""
"","","","","","","","","","","100","tax","invoice","+III.4_tax","hr_140100",""
"","","","","","","","","","","100","base","refund","+II.4_base||+III.4_base","",""
"","","","","","","","","","","-100","tax","refund","-II.4_tax","hr_240100",""
"","","","","","","","","","","100","tax","refund","+III.4_tax","hr_140100",""
"VAT_P_O","29","0% Domestic supplies","0% Domestic supplies","0%","False","0.0","percent","purchase","tax_group_pdv_0","100","base","invoice","+I.8_base","","0% Domaće zalihe"
"","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","100","base","refund","+I.8_base","",""
"","","","","","","","","","","100","tax","refund","","",""

```

## File: data\template\account.tax.group-hr.csv

```csv
"id","name","country_id","name@hr"
"tax_group_pdv_0","PDV 0%","base.hr","PDV 0%"
"tax_group_pdv_5","PDV 5%","base.hr","PDV 5%"
"tax_group_pdv_13","PDV 13%","base.hr","PDV 13%"
"tax_group_pdv_25","PDV 25%","base.hr","PDV 25%"

```

## File: models\template_hr.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('hr')
    def _get_hr_template_data(self):
        return {
            'code_digits': '6',
            'use_storno_accounting': True,
            'property_account_receivable_id': 'hr_120000',
            'property_account_payable_id': 'hr_220000',
            'property_account_expense_categ_id': 'hr_400000',
            'property_account_income_categ_id': 'hr_750000',
        }

    @template('hr', 'res.company')
    def _get_hr_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.hr',
                'bank_account_code_prefix': '100',
                'cash_account_code_prefix': '102',
                'transfer_account_code_prefix': '1009',
                'account_default_pos_receivable_account_id': 'hr_120100',
                'income_currency_exchange_account_id': 'hr_772000',
                'expense_currency_exchange_account_id': 'hr_475000',
                'account_sale_tax_id': 'VAT_S_IN_ROC_25',
                'account_purchase_tax_id': 'VAT_P_IN_ROC_25',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_hr

```

