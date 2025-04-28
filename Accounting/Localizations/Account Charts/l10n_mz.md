# Odoo Module: l10n_mz

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
    'name': 'Mozambique - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'description': """
Mozambican Accounting localization
    """,
    'version': '1.0',
    'icon': '/account/static/description/l10n.png',
    'countries': ['mz'],
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'base',
        'account',
    ],
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
    <record id="l10n_mz_tax_report" model="account.report">
        <field name="name">Tax report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.mz"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_mz_tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_mz_tax_report_frame_5" model="account.report.line">
                <field name="name">Frame 5 - CALCULATION OF THE TAX REGARDING THE PERIOD TO WHICH THE DECLARATION REFERS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_mz_tax_report_line_1" model="account.report.line">
                        <field name="name">1. Transfer of goods and/or rendering of services carried out by the taxable person and respective tax assessed</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_1" model="account.report.line">
                                <field name="name">Field 1 - Transfer of goods and/or rendering of services carried out by the taxable person</field>
                                <field name="code">MZ_TR_F_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_2" model="account.report.line">
                                <field name="name">Field 2 - Tax assessed</field>
                                <field name="code">MZ_TR_F_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_3" model="account.report.line">
                                <field name="name">Field 3 -  Operations of paragraph 1, subparagraph b, of Article 18 of the VAT law</field>
                                <field name="code">MZ_TR_F_3</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_4" model="account.report.line">
                                <field name="name">Field 4 - Operations that do not confer the right to deduction</field>
                                <field name="code">MZ_TR_F_4</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">4</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_2" model="account.report.line">
                        <field name="name">Deductible tax relating to transfers of goods and provision of services made to the declarant taxable person</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_5" model="account.report.line">
                                <field name="name">Field 5 - Deductible tax on tangible and intangible assets</field>
                                <field name="code">MZ_TR_F_5</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_6" model="account.report.line">
                                <field name="name">Field 6 - Deductible tax on Inventories</field>
                                <field name="code">MZ_TR_F_6</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_7" model="account.report.line">
                                <field name="name">Field 7 - Deductible tax on Other goods and services</field>
                                <field name="code">MZ_TR_F_7</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">7</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_3" model="account.report.line">
                        <field name="name">3. Deductible tax incurred on imports of goods carried out by the taxable person (Field 08)</field>
                        <field name="code">MZ_TR_F_8</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_4" model="account.report.line">
                        <field name="name">4. Monthly or annual adjustments, with the exception of those communicated by the Tax Administration</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_9" model="account.report.line">
                                <field name="name">Field 9 - Adjustments in favor of taxable person</field>
                                <field name="code">MZ_TR_F_9</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">9</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_10" model="account.report.line">
                                <field name="name">Field 10 - Adjustments in favor of the state</field>
                                <field name="code">MZ_TR_F_10</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_sum" model="account.report.line">
                        <field name="name">Settlement Amount</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_11" model="account.report.line">
                                <field name="name">Field 11 - Tax base Total</field>
                                <field name="aggregation_formula">MZ_TR_F_1.balance + MZ_TR_F_3.balance + MZ_TR_F_4.balance</field>
                            </record>
                            <record id="l10n_mz_tax_report_field_12" model="account.report.line">
                                <field name="name">Field 12 - Total tax in favor of the taxable person</field>
                                <field name="code">MZ_TR_F_12</field>
                                <field name="aggregation_formula">MZ_TR_F_5.balance + MZ_TR_F_6.balance + MZ_TR_F_7.balance + MZ_TR_F_8.balance + MZ_TR_F_9.balance</field>
                            </record>
                            <record id="l10n_mz_tax_report_field_13" model="account.report.line">
                                <field name="name">Field 13 - Total tax in favor of the state</field>
                                <field name="code">MZ_TR_F_13</field>
                                <field name="aggregation_formula">MZ_TR_F_2.balance + MZ_TR_F_10.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_final" model="account.report.line">
                        <field name="name">Value before the use of the excess to be reported and other credits relating to previous periods</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_14" model="account.report.line">
                                <field name="name">Field 14 - If the value entered in field 13 is greater than that in field 12, enter the difference in field 14 (13-12)</field>
                                <field name="code">MZ_TR_F_14</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_14_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">MZ_TR_F_13.balance - MZ_TR_F_12.balance</field>
                                        <field name="subformula">if_above(MZN(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_15" model="account.report.line">
                                <field name="name">Field 15 - If the value entered in field 12 is greater than that in field 13, enter the difference in field 15 (12-13)</field>
                                <field name="code">MZ_TR_F_15</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_15_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">MZ_TR_F_12.balance - MZ_TR_F_13.balance</field>
                                        <field name="subformula">if_above(MZN(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_credits" model="account.report.line">
                        <field name="name"> Use of credits from previous periods. Important: values ​​can only be entered in fields 16 and 17 if this declaration is presented within the legal deadline.</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_16" model="account.report.line">
                                <field name="name">Field 16 - Excess to be reported from the previous period</field>
                                <field name="code">MZ_TR_F_16</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_16_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_17" model="account.report.line">
                                <field name="name">Field 17 - Credits communicated by the services</field>
                                <field name="code">MZ_TR_F_17</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_17_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_mz_tax_report_frame_6" model="account.report.line">
                <field name="name">Frame 6 - Tax to be delivered to the state (This table should only be completed if field 14 of table 05 has been completed)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_mz_tax_report_field_18" model="account.report.line">
                        <field name="name">Field 18 - Payable VAT</field>
                        <field name="code">MZ_TR_F_18</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_18_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">MZ_TR_F_14.balance - MZ_TR_F_16.balance - MZ_TR_F_17.balance</field>
                                <field name="subformula">if_above(MZN(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_19" model="account.report.line">
                        <field name="name">Field 19 - Late payment fine</field>
                        <field name="code">MZ_TR_F_19</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_19_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=0</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_20" model="account.report.line">
                        <field name="name">Field 20 - Total payable</field>
                        <field name="code">MZ_TR_F_20</field>
                        <field name="aggregation_formula">MZ_TR_F_18.balance + MZ_TR_F_19.balance</field>
                    </record>
                </field>
            </record>
            <record id="l10n_mz_tax_report_frame_7" model="account.report.line">
                <field name="name">Frame 7 - Tax to be recovered by taxable person</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_mz_tax_report_field_21" model="account.report.line">
                        <field name="name">Field 21 - Tax credit</field>
                        <field name="code">MZ_TR_F_21</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_21_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">MZ_TR_F_15.balance + MZ_TR_F_16.balance + MZ_TR_F_17.balance - MZ_TR_F_14.balance</field>
                                <field name="subformula">if_above(MZN(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_22" model="account.report.line">
                        <field name="name">Field 22 - Report for the following period (If this declaration is submitted within the deadline)</field>
                        <field name="code">MZ_TR_F_22</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_22_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=0</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_23" model="account.report.line">
                        <field name="name">Field 23 - Refund request (If this declaration is submitted after the deadline)</field>
                        <field name="code">MZ_TR_F_23</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_23_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=0</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-mz.csv

```csv
"id","name","code","account_type","reconcile"
"l10n_mz_account_211","Purchased Goods","211","asset_current","False"
"l10n_mz_account_2121","Purchased Raw materials","2121","asset_current","False"
"l10n_mz_account_2122","Purchased Ancillary materials","2122","asset_current","False"
"l10n_mz_account_21231","Purchased Fuel and lubricants","21231","asset_current","False"
"l10n_mz_account_21232","Purchased Packaging","21232","asset_current","False"
"l10n_mz_account_21233","Purchased Spare parts","21233","asset_current","False"
"l10n_mz_account_21239","Other Purchased materials","21239","asset_current","False"
"l10n_mz_account_217","Returned purchases","217","asset_current","False"
"l10n_mz_account_218","Discounts and rebates on purchases","218","asset_current","False"
"l10n_mz_account_222","Goods in transit","222","asset_current","False"
"l10n_mz_account_223","Goods held by third parties","223","asset_current","False"
"l10n_mz_account_231","Finished and intermediate goods","231","asset_current","False"
"l10n_mz_account_232","Finished goods held by third parties","232","asset_current","False"
"l10n_mz_account_241","By-products","241","asset_current","False"
"l10n_mz_account_242","Waste and scrap","242","asset_current","False"
"l10n_mz_account_25","Work in progress","25","asset_current","False"
"l10n_mz_account_261","Raw materials","261","asset_current","False"
"l10n_mz_account_262","Ancillary materials","262","asset_current","False"
"l10n_mz_account_2631","Fuel and lubricants","2631","asset_current","False"
"l10n_mz_account_2632","Packaging","2632","asset_current","False"
"l10n_mz_account_2633","Spare parts","2633","asset_current","False"
"l10n_mz_account_2639","Other materials","2639","asset_current","False"
"l10n_mz_account_264","Raw materials and other supplies in transit","264","asset_current","False"
"l10n_mz_account_2711","Livestock for production","2711","asset_non_current","False"
"l10n_mz_account_2712","Plants for production","2712","asset_non_current","False"
"l10n_mz_account_2721","Consumable livestock","2721","asset_current","False"
"l10n_mz_account_2722","Consumable Plants","2722","asset_current","False"
"l10n_mz_account_282","Inventory adjustments - Goods","282","asset_current","False"
"l10n_mz_account_283","Inventory adjustments - Finished and intermediate goods","283","asset_current","False"
"l10n_mz_account_284","Inventory adjustments - By-products, waste and scrap","284","asset_current","False"
"l10n_mz_account_285","Inventory adjustments - Work in progress","285","asset_current","False"
"l10n_mz_account_286","Inventory adjustments - Raw materials and other supplies","286","asset_current","False"
"l10n_mz_account_287","Inventory adjustments - Biological assets","287","asset_current","False"
"l10n_mz_account_292","Net realisable value adjustments - Goods","292","asset_current","False"
"l10n_mz_account_293","Net realisable value adjustments - Finished and intermediate goods","293","asset_current","False"
"l10n_mz_account_294","Net realisable value adjustments - By-products, waste and scrap","294","asset_current","False"
"l10n_mz_account_295","Net realisable value adjustments - Work in progress","295","asset_current","False"
"l10n_mz_account_296","Net realisable value adjustments - Raw materials and other supplies","296","asset_current","False"
"l10n_mz_account_297","Net realisable value adjustments - Biological assets","297","asset_current","False"
"l10n_mz_account_311","Investments in subsidiaries","311","asset_non_current","False"
"l10n_mz_account_312","Investments in associates","312","asset_non_current","False"
"l10n_mz_account_313","Other financial investments","313","asset_non_current","False"
"l10n_mz_account_3211","Industrial buildings","3211","asset_fixed","False"
"l10n_mz_account_3212","Administrative and commercial offices","3212","asset_fixed","False"
"l10n_mz_account_3213","Buildings for housing and other social purposes","3213","asset_fixed","False"
"l10n_mz_account_3216","Transport infrastructures and similar constructions","3216","asset_fixed","False"
"l10n_mz_account_322","Equipment","322","asset_fixed","False"
"l10n_mz_account_323","Furniture and fixtures","323","asset_fixed","False"
"l10n_mz_account_324","Vehicles","324","asset_fixed","False"
"l10n_mz_account_325","Returnable containers","325","asset_fixed","False"
"l10n_mz_account_326","Tools and utensils","326","asset_fixed","False"
"l10n_mz_account_329","Other tangible assets","329","asset_fixed","False"
"l10n_mz_account_331","Development costs","331","asset_non_current","False"
"l10n_mz_account_332","Intellectual property and other rights","332","asset_non_current","False"
"l10n_mz_account_333","Goodwill","333","asset_non_current","False"
"l10n_mz_account_334","Set-up or expansion costs","334","asset_non_current","False"
"l10n_mz_account_342","Tangible Assets under construction","342","asset_non_current","False"
"l10n_mz_account_343","Intangible Assets under construction","343","asset_non_current","False"
"l10n_mz_account_36","Investment property","36","asset_non_current","False"
"l10n_mz_account_382","Accumulated depreciation and amortisation - Tangible assets","382","asset_non_current","False"
"l10n_mz_account_383","Accumulated depreciation and amortisation - Intangible assets","383","asset_non_current","False"
"l10n_mz_account_386","Accumulated depreciation and amortisation - Investment property","386","asset_non_current","False"
"l10n_mz_account_39","Financial investments adjustments","39","asset_non_current","False"
"l10n_mz_account_411","Trade receivables - current account","411","asset_receivable","True"
"l10n_mz_account_412","Securities receivable","412","asset_receivable","True"
"l10n_mz_account_413","Point of sale receivable","413","asset_receivable","True"
"l10n_mz_account_418","Doubtful debts","418","asset_receivable","True"
"l10n_mz_account_419","Advances from clients","419","asset_current","False"
"l10n_mz_account_421","Trade payables – current account","421","liability_payable","True"
"l10n_mz_account_422","Securities payable","422","liability_payable","True"
"l10n_mz_account_429","Advances to suppliers","429","liability_current","False"
"l10n_mz_account_4311","Bank loans - Short term","4311","liability_current","False"
"l10n_mz_account_4312","Bank loans - Medium and long term","4312","liability_non_current","True"
"l10n_mz_account_4411","Income tax - Tax estimate","4411","liability_current","True"
"l10n_mz_account_4412","Income tax - Progress payments","4412","liability_current","False"
"l10n_mz_account_4413","Income tax - Special progress payments","4413","liability_current","False"
"l10n_mz_account_4421","Withholding tax - Income from employment","4421","liability_current","False"
"l10n_mz_account_4422","Withholding tax - Professional income","4422","liability_current","False"
"l10n_mz_account_4423","Withholding tax - Capital returns","4423","liability_current","False"
"l10n_mz_account_4424","Withholding tax - Property income","4424","liability_current","False"
"l10n_mz_account_4425","Withholding tax - Other income","4425","liability_current","False"
"l10n_mz_account_44311","Input VAT - Inventories","44311","liability_current","False"
"l10n_mz_account_44312","Input VAT - Tangible and intangible assets","44312","liability_current","False"
"l10n_mz_account_44313","Input VAT - Other goods and services","44313","liability_current","False"
"l10n_mz_account_44321","Deductible VAT - Inventories","44321","liability_current","False"
"l10n_mz_account_44322","Deductible VAT - Tangible and intangible assets","44322","liability_current","False"
"l10n_mz_account_44323","Deductible VAT - Other goods and services","44323","liability_current","False"
"l10n_mz_account_44331","Assessed VAT - General transactions","44331","liability_current","False"
"l10n_mz_account_44332","Assessed VAT - Self consumption and gifts","44332","liability_current","False"
"l10n_mz_account_44333","Assessed VAT - Special transactions","44333","liability_current","False"
"l10n_mz_account_44341","Monthly VAT adjustments in favour of taxable person","44341","asset_current","False"
"l10n_mz_account_44342","Monthly VAT adjustments in favour of State","44342","liability_current","False"
"l10n_mz_account_44343","Annual VAT adjustments by calculation of final pro rata","44343","liability_current","False"
"l10n_mz_account_4435","VAT assessment","4435","liability_current","False"
"l10n_mz_account_4436","VAT assessed by Tax Authorities","4436","liability_current","False"
"l10n_mz_account_4437","VAT payable","4437","liability_current","False"
"l10n_mz_account_4438","VAT recoverable","4438","asset_current","False"
"l10n_mz_account_4439","VAT requested refunds","4439","liability_current","False"
"l10n_mz_account_4441","Stamp duty","4441","liability_current","False"
"l10n_mz_account_4442","MunicipaI taxes","4442","liability_current","False"
"l10n_mz_account_445","Tax adjustments, contributions and other levies","445","liability_current","False"
"l10n_mz_account_449","INSS contributions","449","liability_current","False"
"l10n_mz_account_4511","Advances to corporate bodies","4511","asset_receivable","True"
"l10n_mz_account_4512","Advances to employees","4512","asset_receivable","True"
"l10n_mz_account_4518","Other receivable transactions with corporate bodies","4518","asset_receivable","True"
"l10n_mz_account_4519","Other receivable transactions with employees","4519","asset_receivable","True"
"l10n_mz_account_4521","State and other public entities","4521","asset_receivable","True"
"l10n_mz_account_4522","Private entities","4522","asset_receivable","True"
"l10n_mz_account_4529","Other entities","4529","asset_receivable","True"
"l10n_mz_account_4541","Loans receivable","4541","asset_receivable","True"
"l10n_mz_account_4542","Advances on profits","4542","asset_receivable","True"
"l10n_mz_account_4543","Distributed profits and losses","4543","asset_receivable","True"
"l10n_mz_account_4544","Available profits","4544","asset_receivable","True"
"l10n_mz_account_4549","Other receivable transactions","4549","asset_receivable","True"
"l10n_mz_account_4551","Grants receivable - State and other public entities","4551","asset_receivable","True"
"l10n_mz_account_4552","Grants receivable - Private entities","4552","asset_receivable","True"
"l10n_mz_account_459","Other debtors","459","asset_receivable","True"
"l10n_mz_account_4611","Capital expenditure creditors – Current account","4611","liability_payable","True"
"l10n_mz_account_4612","Capital expenditure creditors - Payable securities","4612","liability_payable","True"
"l10n_mz_account_4613","Capital expenditure creditors - Advances","4613","liability_payable","True"
"l10n_mz_account_4614","Capital expenditure creditors - Finance lease","4614","liability_payable","True"
"l10n_mz_account_4619","Capital expenditure creditors - Other","4619","liability_payable","True"
"l10n_mz_account_4621","Remuneration payable to corporate bodies","4621","liability_payable","True"
"l10n_mz_account_4622","Remuneration payable to employees","4622","liability_payable","True"
"l10n_mz_account_4628","Other transactions with corporate bodies","4628","liability_payable","True"
"l10n_mz_account_4629","Other transactions with employees","4629","liability_payable","True"
"l10n_mz_account_463","Trade unions","463","liability_payable","True"
"l10n_mz_account_466","Consultants, advisors and intermediaries","466","liability_payable","True"
"l10n_mz_account_4671","Borrowings from partners, shareholders or owners","4671","liability_payable","True"
"l10n_mz_account_4673","Distributed profits","4673","liability_payable","True"
"l10n_mz_account_4674","Available profits","4674","liability_payable","True"
"l10n_mz_account_469","Other creditors","469","liability_payable","True"
"l10n_mz_account_471","Accounts receivable adjustments - Trade receivables","471","asset_receivable","True"
"l10n_mz_account_472","Accounts receivable adjustments - Other receivables","472","asset_receivable","True"
"l10n_mz_account_481","Provisions - Outstanding legal matters","481","liability_non_current","True"
"l10n_mz_account_482","Provisions - Accidents at work and occupational diseases","482","liability_non_current","True"
"l10n_mz_account_483","Provisions - Taxes","483","liability_current","True"
"l10n_mz_account_484","Provisions - Business restructuring","484","liability_non_current","True"
"l10n_mz_account_485","Provisions - Onerous contracts","485","liability_non_current","True"
"l10n_mz_account_486","Provisions - Warranty obligations","486","liability_non_current","True"
"l10n_mz_account_487","Provisions - Losses on construction contracts","487","liability_current","True"
"l10n_mz_account_489","Provisions - Other","489","liability_non_current","True"
"l10n_mz_account_4911","Interest payable","4911","liability_payable","True"
"l10n_mz_account_4912","Remuneration payable","4912","liability_payable","True"
"l10n_mz_account_4919","Other accrued expenses","4919","liability_payable","True"
"l10n_mz_account_4923","Revenue from construction contracts","4923","liability_payable","True"
"l10n_mz_account_4924","Investment grants","4924","liability_payable","True"
"l10n_mz_account_4929","Other deferred income","4929","liability_payable","True"
"l10n_mz_account_4931","Interest receivable","4931","asset_receivable","True"
"l10n_mz_account_4933","Revenue from construction contracts","4933","asset_receivable","True"
"l10n_mz_account_4939","Other accrued income","4939","asset_receivable","True"
"l10n_mz_account_494","Deferred expenses","494","asset_receivable","True"
"l10n_mz_account_51","Share capital","51","equity","False"
"l10n_mz_account_521","Treasury shares - Nominal value","521","equity","False"
"l10n_mz_account_522","Treasury shares - Discounts and premiums","522","equity","False"
"l10n_mz_account_53","Supplementary capital","53","equity","False"
"l10n_mz_account_54","Share premium","54","equity","False"
"l10n_mz_account_551","Legal reserves","551","equity","False"
"l10n_mz_account_552","Statutory reserves","552","equity","False"
"l10n_mz_account_553","Free reserves","553","equity","False"
"l10n_mz_account_561","Legal revaluations","561","equity","False"
"l10n_mz_account_562","Other surplus","562","equity","False"
"l10n_mz_account_58","Other changes in equity","58","equity","False"
"l10n_mz_account_6112","Cost of Goods","6112","expense_direct_cost","False"
"l10n_mz_account_61161","Cost of Raw materials","61161","expense_direct_cost","False"
"l10n_mz_account_61162","Cost of Ancillary materials","61162","expense_direct_cost","False"
"l10n_mz_account_61163","Cost of Other materials","61163","expense_direct_cost","False"
"l10n_mz_account_6117","Cost of Biological assets","6117","expense_direct_cost","False"
"l10n_mz_account_6121","Change in production - Finished and intermediate goods","6121","expense_direct_cost","False"
"l10n_mz_account_6122","Change in production - By-products, waste and scrap","6122","expense_direct_cost","False"
"l10n_mz_account_6123","Change in production - Work in progress","6123","expense_direct_cost","False"
"l10n_mz_account_621","Remuneration of corporate bodies","621","expense","False"
"l10n_mz_account_622","Remuneration of employees","622","expense","False"
"l10n_mz_account_623","Charges on remuneration","623","expense","False"
"l10n_mz_account_6251","Allowances - Taxable","6251","expense","False"
"l10n_mz_account_6252","Allowances - Non taxable","6252","expense","False"
"l10n_mz_account_6261","Indemnities - Insurable risk","6261","expense","False"
"l10n_mz_account_6262","Indemnities - Other","6262","expense","False"
"l10n_mz_account_627","Insurance covering accidents at work and occupational diseases","627","expense","False"
"l10n_mz_account_628","Expenses of social nature","628","expense","False"
"l10n_mz_account_629","Other staff expenses","629","expense","False"
"l10n_mz_account_631","Subcontracts","631","expense","False"
"l10n_mz_account_63211","Water","63211","expense","False"
"l10n_mz_account_63212","Electricity","63212","expense","False"
"l10n_mz_account_63213","Fuel","63213","expense","False"
"l10n_mz_account_63214","Fast wear and tear tools","63214","expense","False"
"l10n_mz_account_63215","Maintenance and repair material","63215","expense","False"
"l10n_mz_account_63216","Stationary","63216","expense","False"
"l10n_mz_account_63217","Technical books and documentation","63217","expense","False"
"l10n_mz_account_63218","Gifts","63218","expense","False"
"l10n_mz_account_63221","Maintenance and repair","63221","expense","False"
"l10n_mz_account_63222","Freight services","63222","expense","False"
"l10n_mz_account_63223","Staff transport","63223","expense","False"
"l10n_mz_account_63224","Communications","63224","expense","False"
"l10n_mz_account_63225","Fees","63225","expense","False"
"l10n_mz_account_63226","Commissions to intermediaries","63226","expense","False"
"l10n_mz_account_632271","Advertising – Campaigns","632271","expense","False"
"l10n_mz_account_632272","Advertising - Other","632272","expense","False"
"l10n_mz_account_632281","Travel and accommodation – In business","632281","expense","False"
"l10n_mz_account_632282","Travel and accommodation - Other","632282","expense","False"
"l10n_mz_account_63229","Entertainment expenses","63229","expense","False"
"l10n_mz_account_63231","Litigation and notary expenses","63231","expense","False"
"l10n_mz_account_632321","Hire and rental charges - Finance lease","632321","expense","False"
"l10n_mz_account_632331","Life, personal accidents and disease insurance","632331","expense","False"
"l10n_mz_account_63234","Royalties","63234","expense","False"
"l10n_mz_account_63235","Cleaning, hygiene and comfort expenses","63235","expense","False"
"l10n_mz_account_63236","Surveillance and security","63236","expense","False"
"l10n_mz_account_63237","Specialised services","63237","expense","False"
"l10n_mz_account_63299","Other supplies of goods and services","63299","expense","False"
"l10n_mz_account_641","Adjustments from inventories to net realisable value","641","expense","False"
"l10n_mz_account_642","Adjustments from Financial investments","642","expense","False"
"l10n_mz_account_643","Adjustments from Investment property","643","expense","False"
"l10n_mz_account_6441","Receivables – adjustments within tax limits","6441","expense","False"
"l10n_mz_account_6442","Receivables – adjustments beyond tax limits","6442","expense","False"
"l10n_mz_account_651","Depreciation and amortisation for the period - Tangible assets","651","expense_depreciation","False"
"l10n_mz_account_652","Depreciation and amortisation for the period - Intangible assets","652","expense_depreciation","False"
"l10n_mz_account_653","Depreciation and amortisation for the period - Investment property","653","expense_depreciation","False"
"l10n_mz_account_661","Provisions for the period - Outstanding legal matters","661","expense","False"
"l10n_mz_account_662","Provisions for the period - Accidents at work and occupational diseases","662","expense","False"
"l10n_mz_account_663","Provisions for the period - Taxes","663","expense","False"
"l10n_mz_account_664","Provisions for the period - Business restructuring","664","expense","False"
"l10n_mz_account_665","Provisions for the period - Onerous contracts","665","expense","False"
"l10n_mz_account_666","Provisions for the period - Warranty obligations","666","expense","False"
"l10n_mz_account_667","Provisions for the period - Losses on construction contracts","667","expense","False"
"l10n_mz_account_669","Provisions for the period - Other","669","expense","False"
"l10n_mz_account_681","Research expenses","681","expense","False"
"l10n_mz_account_6821","Custom duties","6821","expense","False"
"l10n_mz_account_6822","Value added tax","6822","expense","False"
"l10n_mz_account_6823","Stamp duty","6823","expense","False"
"l10n_mz_account_6824","Taxes on vehicles","6824","expense","False"
"l10n_mz_account_6825","Municipal taxes","6825","expense","False"
"l10n_mz_account_6831","Disposals","6831","expense","False"
"l10n_mz_account_6832","Retirements","6832","expense","False"
"l10n_mz_account_6833","claims on capital investments","6833","expense","False"
"l10n_mz_account_6841","claims on inventories and biological assets","6841","expense","False"
"l10n_mz_account_6842","Breaks","6842","expense","False"
"l10n_mz_account_6849","Other Losses in inventories and biological assets","6849","expense","False"
"l10n_mz_account_6891","Contributions","6891","expense","False"
"l10n_mz_account_6892","Confidential expenses","6892","expense","False"
"l10n_mz_account_6893","Gifts and inventory samples","6893","expense","False"
"l10n_mz_account_6894","Social responsibility programs","6894","expense","False"
"l10n_mz_account_68951","Donations to the State","68951","expense","False"
"l10n_mz_account_68952","Donations - Other patronage","68952","expense","False"
"l10n_mz_account_6896","Fines and penalties","6896","expense","False"
"l10n_mz_account_6899","Other operating expenses","6899","expense","False"
"l10n_mz_account_6911","Bank loans expenses","6911","expense","False"
"l10n_mz_account_6913","Expenses on Loans from partners, shareholders or owners","6913","expense","False"
"l10n_mz_account_6914","Expenses on Other borrowing","6914","expense","False"
"l10n_mz_account_6915","Securities discount","6915","expense","False"
"l10n_mz_account_69161","Interest of default payments","69161","expense","False"
"l10n_mz_account_69162","Compensatory interest","69162","expense","False"
"l10n_mz_account_6919","Other interest","6919","expense","False"
"l10n_mz_account_6941","Foreign exchange losses - Realised","6941","expense","False"
"l10n_mz_account_6942","Foreign exchange losses - Unrealised","6942","expense","False"
"l10n_mz_account_695","Cash discounts granted","695","expense","False"
"l10n_mz_account_6981","Bank charges","6981","expense","False"
"l10n_mz_account_6989","Other finance costs and losses","6989","expense","False"
"l10n_mz_account_711","Sales - Goods","711","income","False"
"l10n_mz_account_712","Sales - Finished and intermediate goods","712","income","False"
"l10n_mz_account_713","Sales - By-products, waste and scrap","713","income","False"
"l10n_mz_account_714","Sales - Biological assets","714","income","False"
"l10n_mz_account_715","Sales - VAT from sales with tax included","715","income_other","False"
"l10n_mz_account_716","Sales - Return of goods sold","716","income","False"
"l10n_mz_account_717","Sales - Discounts and rebates","717","income_other","False"
"l10n_mz_account_721","Services rendered","721","income_other","False"
"l10n_mz_account_722","VAT from services rendered with tax included","722","income_other","False"
"l10n_mz_account_726","Discounts and rebates Services","726","income_other","False"
"l10n_mz_account_731","Work performed by the entity and capitalised - financial investments","731","income","False"
"l10n_mz_account_732","Work performed by the entity and capitalised - Tangible assets","732","income","False"
"l10n_mz_account_733","Work performed by the entity and capitalised - Intangible assets","733","income","False"
"l10n_mz_account_734","Work performed by the entity and capitalised - Assets under construction","734","income","False"
"l10n_mz_account_7411","Reversals for the period from adjustments of inventories to net realisable value","7411","income_other","False"
"l10n_mz_account_7412","Reversals for the period from adjustments of Financial investments","7412","income_other","False"
"l10n_mz_account_7413","Reversals for the period from adjustments of Tangible assets","7413","income_other","False"
"l10n_mz_account_7414","Reversals for the period from adjustments of Receivables","7414","income_other","False"
"l10n_mz_account_7421","Reversals from depreciation - Tangible assets","7421","income_other","False"
"l10n_mz_account_7422","Reversals from depreciation - Intangible assets","7422","income_other","False"
"l10n_mz_account_7423","Reversals from depreciation - Investment property","7423","income_other","False"
"l10n_mz_account_7431","Reversals from provisions - Outstanding legal matters","7431","income_other","False"
"l10n_mz_account_7432","Reversals from provisions - Accidents at work and occupational diseases","7432","income_other","False"
"l10n_mz_account_7433","Reversals from provisions - Taxes","7433","income_other","False"
"l10n_mz_account_7434","Reversals from provisions - Business restructuring","7434","income_other","False"
"l10n_mz_account_7435","Reversals from provisions - Onerous contracts","7435","income_other","False"
"l10n_mz_account_7436","Reversals from provisions - Warranty obligations","7436","income_other","False"
"l10n_mz_account_7437","Reversals from provisions - Losses on construction contracts","7437","income_other","False"
"l10n_mz_account_7439","Reversals from provisions - Other provisions","7439","income_other","False"
"l10n_mz_account_75","Supplementary income","75","income_other","False"
"l10n_mz_account_7611","Grants for investments from State and other public entities","7611","income_other","False"
"l10n_mz_account_7619","Grants for investments from other entities","7619","income_other","False"
"l10n_mz_account_7621","Operating grants from State and other public entities","7621","income_other","False"
"l10n_mz_account_7629","Operating grants from other entities","7629","income_other","False"
"l10n_mz_account_7631","Profit on disposals from capital investments","7631","income_other","False"
"l10n_mz_account_7632","Profit on claims from capital investments","7632","income_other","False"
"l10n_mz_account_7641","Profit on claims in inventories and biological assets","7641","income_other","False"
"l10n_mz_account_7642","Profit on scrap","7642","income_other","False"
"l10n_mz_account_7649","Other gains in inventories and biological assets","7649","income_other","False"
"l10n_mz_account_7691","Tax refund","7691","income_other","False"
"l10n_mz_account_7692","Benefits from contractual penalties","7692","income_other","False"
"l10n_mz_account_7693","Excess of tax estimate","7693","income_other","False"
"l10n_mz_account_7699","Other income","7699","income_other","False"
"l10n_mz_account_7811","Interest received - Bank deposits","7811","income","False"
"l10n_mz_account_7812","Interest received - Loans","7812","income","False"
"l10n_mz_account_7814","Other cash investments","7814","income","False"
"l10n_mz_account_7819","Other interest","7819","income","False"
"l10n_mz_account_782","Income from investment property","782","income","False"
"l10n_mz_account_783","Income from financial investments","783","income","False"
"l10n_mz_account_7841","Foreign exchange gains - Realised","7841","income","False"
"l10n_mz_account_7842","Foreign exchange gains - Unrealised","7842","income","False"
"l10n_mz_account_785","Discounts on cash payments","785","income","False"
"l10n_mz_account_789","Other financial income and gains","789","income","False"
"l10n_mz_account_85","Income tax","85","expense","False"

```

## File: data\template\account.fiscal.position-mz.csv

```csv
"id","name","auto_apply","country_id","sequence","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_dom","Domestic regime","1","base.mz","","",""
"fiscal_position_import_export","Import/Export","1","","1","vat_sale_16","vat_export"
"","","","","","vat_purch_16_inventories","vat_exempt_import"
"","","","","","vat_purch_16_fixed","vat_exempt_import"
"","","","","","vat_purch_16_other","vat_exempt_import"
"","","","","","vat_sale_5","vat_export"
"","","","","","vat_purchase_5","vat_exempt_import"

```

## File: data\template\account.group-mz.csv

```csv
"id","code_prefix_start","code_prefix_end","name"
"l10n_mz_account_group_1","1","","Financial resources"
"l10n_mz_account_group_2","2","","Inventories and biological assets"
"l10n_mz_account_group_21","21","","Purchased Inventories"
"l10n_mz_account_group_212","212","","Purchased Raw materials and other supplies"
"l10n_mz_account_group_2123","2123","","Other Purchased materials"
"l10n_mz_account_group_22","22","","Goods"
"l10n_mz_account_group_23","23","","Finished and intermediate goods"
"l10n_mz_account_group_24","24","","By-products, waste and scrap"
"l10n_mz_account_group_26","26","","Raw materials and other supplies"
"l10n_mz_account_group_263","263","","Other materials"
"l10n_mz_account_group_27","27","","Biological assets"
"l10n_mz_account_group_271","271","","Biological assets for production"
"l10n_mz_account_group_272","272","","Consumable biological assets"
"l10n_mz_account_group_28","28","","Inventory adjustments"
"l10n_mz_account_group_29","29","","Net realisable value adjustments"
"l10n_mz_account_group_3","3","","Capital investments"
"l10n_mz_account_group_31","31","","Financial investments"
"l10n_mz_account_group_32","32","","Tangible assets"
"l10n_mz_account_group_321","321","","Buildings"
"l10n_mz_account_group_33","33","","Intangible assets"
"l10n_mz_account_group_34","34","","Assets under construction"
"l10n_mz_account_group_38","38","","Accumulated depreciation and amortisation"
"l10n_mz_account_group_4","4","","Accounts receivable, accounts payable, accruals, and deferrals"
"l10n_mz_account_group_41","41","","Trade receivables"
"l10n_mz_account_group_42","42","","Trade payables"
"l10n_mz_account_group_43","43","","Borrowings"
"l10n_mz_account_group_431","431","","Bank loans"
"l10n_mz_account_group_44","44","","State"
"l10n_mz_account_group_441","441","","Income tax"
"l10n_mz_account_group_442","442","","Withholding tax"
"l10n_mz_account_group_443","443","","Value added tax"
"l10n_mz_account_group_4431","4431","","Input VAT"
"l10n_mz_account_group_4432","4432","","Deductible VAT"
"l10n_mz_account_group_4433","4433","","Assessed VAT"
"l10n_mz_account_group_4434","4434","","VAT adjustments"
"l10n_mz_account_group_444","444","","Remaining taxes"
"l10n_mz_account_group_45","45","","Other receivables"
"l10n_mz_account_group_451","451","","Employees receivables"
"l10n_mz_account_group_452","452","","Subscribers of Capital"
"l10n_mz_account_group_454","454","","Debtors – partners, shareholders or owners"
"l10n_mz_account_group_455","455","","Grants receivable"
"l10n_mz_account_group_46","46","","Other payables"
"l10n_mz_account_group_461","461","","Capital expenditure creditors"
"l10n_mz_account_group_462","462","","Employees payables"
"l10n_mz_account_group_467","467","","Creditors – partners, shareholders or owners"
"l10n_mz_account_group_47","47","","Accounts receivable adjustments"
"l10n_mz_account_group_48","48","","Provisions"
"l10n_mz_account_group_49","49","","Accruals and deferrals"
"l10n_mz_account_group_491","491","","Accrued expenses"
"l10n_mz_account_group_492","492","","Deferred income"
"l10n_mz_account_group_493","493","","Accrued income"
"l10n_mz_account_group_5","5","","Equity"
"l10n_mz_account_group_52","52","","Treasury shares"
"l10n_mz_account_group_55","55","","Reserves"
"l10n_mz_account_group_56","56","","Surplus on revaluation of tangible and intangible assets"
"l10n_mz_account_group_6","6","","Expenses and losses"
"l10n_mz_account_group_61","61","","Cost of inventories"
"l10n_mz_account_group_611","611","","Cost of inventories sold or consumed"
"l10n_mz_account_group_6116","6116","","Raw materials and other supplies"
"l10n_mz_account_group_612","612","","Change in production"
"l10n_mz_account_group_62","62","","Staff expenses"
"l10n_mz_account_group_625","625","","Allowances"
"l10n_mz_account_group_626","626","","Indemnities"
"l10n_mz_account_group_63","63","","Purchased supplies and services"
"l10n_mz_account_group_632","632","","Supplies of goods and services"
"l10n_mz_account_group_63227","63227","","Advertising"
"l10n_mz_account_group_63228","63228","","Travel and accommodation"
"l10n_mz_account_group_63232","63232","","Hire and rental charges"
"l10n_mz_account_group_63233","63233","","Insurance"
"l10n_mz_account_group_64","64","","Adjustments for the period"
"l10n_mz_account_group_644","644","","Receivables"
"l10n_mz_account_group_65","65","","Depreciation and amortisation for the period"
"l10n_mz_account_group_66","66","","Provisions for the period"
"l10n_mz_account_group_68","68","","Other operating expenses and losses"
"l10n_mz_account_group_682","682","","Taxes and levies"
"l10n_mz_account_group_683","683","","Losses in capital investments"
"l10n_mz_account_group_684","684","","Losses in inventories and biological assets"
"l10n_mz_account_group_689","689","","Other operating expenses"
"l10n_mz_account_group_6895","6895","","Donations"
"l10n_mz_account_group_69","69","","Finance costs and losses"
"l10n_mz_account_group_691","691","","Interest expenses"
"l10n_mz_account_group_6916","6916","","Interest of default payments and compensatory"
"l10n_mz_account_group_694","694","","Foreign exchange losses"
"l10n_mz_account_group_698","698","","Other finance costs and losses"
"l10n_mz_account_group_7","7","","Revenue and gains"
"l10n_mz_account_group_71","71","","Sales"
"l10n_mz_account_group_72","72","","Services rendered"
"l10n_mz_account_group_73","73","","Work performed by the entity and capitalised"
"l10n_mz_account_group_74","74","","Reversals for the period"
"l10n_mz_account_group_741","741","","Reversals From adjustments"
"l10n_mz_account_group_742","742","","Reversals From depreciation and amortisation"
"l10n_mz_account_group_743","743","","Reversals From provisions"
"l10n_mz_account_group_76","76","","Other operating income and gains"
"l10n_mz_account_group_761","761","","Grants for investments"
"l10n_mz_account_group_762","762","","Operating grants"
"l10n_mz_account_group_763","763","","Gains from capital investments"
"l10n_mz_account_group_764","764","","Gains in inventories and biological assets"
"l10n_mz_account_group_769","769","","Other income non-related to value added"
"l10n_mz_account_group_78","78","","Finance income and gains"
"l10n_mz_account_group_781","781","","Interest received"
"l10n_mz_account_group_784","784","","Foreign exchange gains"

```

## File: data\template\account.tax-mz.csv

```csv
"id","name","description","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id"
"vat_sale_16","16%","16%","16.0","percent","sale","tax_group_vat_16","100","base","invoice","+1",""
"","","","","","","","100","tax","invoice","+2","l10n_mz_account_44331"
"","","","","","","","100","base","refund","-1",""
"","","","","","","","100","tax","refund","-2","l10n_mz_account_44331"
"vat_sale_5","5% S","5%","5.0","percent","sale","tax_group_vat_5","100","base","invoice","+1",""
"","","","","","","","100","tax","invoice","+2","l10n_mz_account_44331"
"","","","","","","","100","base","refund","-1",""
"","","","","","","","100","tax","refund","-2","l10n_mz_account_44331"
"vat_export","0% EX","0%","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+3",""
"","","","","","","","100","tax","invoice","",""
"","","","","","","","100","base","refund","-3",""
"","","","","","","","100","tax","refund","",""
"vat_exempt_sale","0%","0%","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+4",""
"","","","","","","","100","tax","invoice","",""
"","","","","","","","100","base","refund","-4",""
"","","","","","","","100","tax","refund","",""
"vat_purch_16_fixed","16% G Fixed Assets","16%","16.0","percent","purchase","tax_group_vat_16","100","base","invoice","",""
"","","","","","","","100","tax","invoice","+5","l10n_mz_account_44322"
"","","","","","","","100","base","refund","",""
"","","","","","","","100","tax","refund","-5","l10n_mz_account_44322"
"vat_purch_16_inventories","16% G inventories","16%","16.0","percent","purchase","tax_group_vat_16","100","base","invoice","",""
"","","","","","","","100","tax","invoice","+6","l10n_mz_account_44321"
"","","","","","","","100","base","refund","",""
"","","","","","","","100","tax","refund","-6","l10n_mz_account_44321"
"vat_purch_16_other","16% GS","16%","16.0","percent","purchase","tax_group_vat_16","100","base","invoice","",""
"","","","","","","","100","tax","invoice","+7","l10n_mz_account_44323"
"","","","","","","","100","base","refund","",""
"","","","","","","","100","tax","refund","-7","l10n_mz_account_44323"
"vat_import","16% Import","16%","16.0","percent","purchase","tax_group_vat_16","100","base","invoice","",""
"","","","","","","","100","tax","invoice","+8","l10n_mz_account_44323"
"","","","","","","","100","base","refund","",""
"","","","","","","","100","tax","refund","-8","l10n_mz_account_44323"
"vat_purchase_5","5% S","5%","5.0","percent","purchase","tax_group_vat_5","100","base","invoice","",""
"","","","","","","","100","tax","invoice","+7","l10n_mz_account_44323"
"","","","","","","","100","base","refund","",""
"","","","","","","","100","tax","refund","-7","l10n_mz_account_44323"
"vat_exempt_import","0% Import","0%","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","",""
"","","","","","","","100","tax","invoice","",""
"","","","","","","","100","base","refund","",""
"","","","","","","","100","tax","refund","",""
"vat_exempt_purchase","0%","0%","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","",""
"","","","","","","","100","tax","invoice","",""
"","","","","","","","100","base","refund","",""
"","","","","","","","100","tax","refund","",""

```

## File: data\template\account.tax.group-mz.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"tax_group_vat_0","VAT 0%","base.mz","l10n_mz_account_4438","l10n_mz_account_4437"
"tax_group_vat_5","VAT 5%","base.mz","l10n_mz_account_4438","l10n_mz_account_4437"
"tax_group_vat_16","VAT 16%","base.mz","l10n_mz_account_4438","l10n_mz_account_4437"

```

## File: models\template_mz.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('mz')
    def _get_mz_template_data(self):
        return {
            'code_digits': '7',
            'property_account_receivable_id': 'l10n_mz_account_411',
            'property_account_payable_id': 'l10n_mz_account_421',
            'property_account_expense_categ_id': 'l10n_mz_account_61161',
            'property_account_income_categ_id': 'l10n_mz_account_711',
        }

    @template('mz', 'res.company')
    def _get_mz_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.mz',
                'bank_account_code_prefix': '12',
                'cash_account_code_prefix': '11',
                'transfer_account_code_prefix': '456',
                'account_default_pos_receivable_account_id': 'l10n_mz_account_413',
                'income_currency_exchange_account_id': 'l10n_mz_account_7841',
                'expense_currency_exchange_account_id': 'l10n_mz_account_6941',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_mz_account_695',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_mz_account_785',
                'account_sale_tax_id': 'vat_sale_16',
                'account_purchase_tax_id': 'vat_purch_16_inventories',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_mz

```

