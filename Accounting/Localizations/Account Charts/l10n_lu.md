# Odoo Module: l10n_lu

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

def _post_init_hook(env):
    _preserve_tag_on_taxes(env)

def _preserve_tag_on_taxes(env):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(env, 'l10n_lu')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Luxembourg - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/luxembourg.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['lu'],
    'version': '2.2',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Luxembourg.
======================================================================

    * the Luxembourg Official Chart of Accounts (law of June 2009 + 2015 chart and Taxes),
    * the Tax Code Chart for Luxembourg
    * the main taxes used in Luxembourg
    * default fiscal position for local, intracom, extracom

Notes:
    * the 2015 chart of taxes is implemented to a large extent,
      see the first sheet of tax.xls for details of coverage
    * to update the chart of tax template, update tax.xls and run tax2csv.py
""",
    'author': 'Odoo S.A., ADN, ACSONE SA/NV',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
        'data/account.account.tag.csv',
        'data/l10n_lu_chart_data.xml',
        'data/tax_report/section_1.xml',
        'data/tax_report/section_2.xml',
        'data/tax_report/sections_34.xml',
        'data/tax_report/tax_report.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_post_init_hook',
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
"id","name","applicability","country_id/id"
"account_tag_appendix_188","188 - Appendix A - Expenses for other work carried out by third parties","accounts","base.lu"
"account_tag_appendix_190","190 - Appendix A - Car expenses","accounts","base.lu"
"account_tag_appendix_239","239 - Appendix A - Gross salaries","accounts","base.lu"
"account_tag_appendix_244","244 - Appendix A - Gross wages","accounts","base.lu"
"account_tag_appendix_247","247 - Appendix A - Occasional salaries","accounts","base.lu"
"account_tag_appendix_250","250 - Appendix A - Compulsory social security contributions (employer's share)","accounts","base.lu"
"account_tag_appendix_253","253 - Appendix A - Accident insurance","accounts","base.lu"
"account_tag_appendix_260","260 - Appendix A - Staff travel and representation expenses","accounts","base.lu"
"account_tag_appendix_269","269 - Appendix A - Accounting and bookkeeping fees","accounts","base.lu"
"account_tag_appendix_283","283 - Appendix A - Employer's travel and representation expenses","accounts","base.lu"
"account_tag_appendix_285","285 - Appendix A - Electricity","accounts","base.lu"
"account_tag_appendix_289","289 - Appendix A - Gas","accounts","base.lu"
"account_tag_appendix_293","293 - Appendix A - Employer's travel and representation expenses","accounts","base.lu"
"account_tag_appendix_301","301 - Appendix A - Telecommunications","accounts","base.lu"
"account_tag_appendix_305","305 - Appendix A - Renting/leasing of immovable property with application of VAT","accounts","base.lu"
"account_tag_appendix_307","307 - Appendix A - Renting/leasing of immovable property with no application of VAT","accounts","base.lu"
"account_tag_appendix_310","310 - Appendix A - Renting/leasing of permanently installed equipment and machinery","accounts","base.lu"
"account_tag_appendix_316","316 - Appendix A - Property tax","accounts","base.lu"
"account_tag_appendix_324","324 - Appendix A - Business tax","accounts","base.lu"
"account_tag_appendix_326","326 - Appendix A - Interest paid for long-term debts","accounts","base.lu"
"account_tag_appendix_327","327 - Appendix A - Interest paid for short-term debts","accounts","base.lu"
"account_tag_appendix_328","328 - Appendix A - Other financial costs","accounts","base.lu"
"account_tag_appendix_330","330 - Appendix A - Stock and business equipment insurance","accounts","base.lu"
"account_tag_appendix_331","331 - Appendix A - Public and professional third party liability insurance","accounts","base.lu"
"account_tag_appendix_332","332 - Appendix A - Office expenses","accounts","base.lu"
"account_tag_appendix_336","336 - Appendix A - Fees and subscriptions paid to professional associations and learned societies","accounts","base.lu"
"account_tag_appendix_337","337 - Appendix A - Papers and periodicals for business purposes","accounts","base.lu"
"account_tag_appendix_343","343 - Appendix A - Shipping and transport expenses","accounts","base.lu"
"account_tag_appendix_345","345 - Appendix A - Work clothes","accounts","base.lu"
"account_tag_appendix_347","347 - Appendix A - Advertising and publicity","accounts","base.lu"
"account_tag_appendix_349","349 - Appendix A - Packaging","accounts","base.lu"
"account_tag_appendix_351","351 - Appendix A - Repair and maintenance of equipment and machinery","accounts","base.lu"
"account_tag_appendix_353","353 - Appendix A - Other repairs","accounts","base.lu"
"account_tag_appendix_355","355 - Appendix A - New acquisitions (tools and equipment) if their cost can be fully allocated to the year of acquisition or creation","accounts","base.lu"
"account_tag_appendix_358","358 - Appendix A - Custom (value)","accounts","base.lu"
"account_tag_appendix_361","361 - Appendix A - Total 'Appendix to Operational expenditures'","accounts","base.lu"

```

## File: data\l10n_lu_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <menuitem id="account_reports_lu_statements_menu" name="Luxembourg" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>

        </odoo>

```

## File: data\tax_report\sections_34.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="l10n_lu_tax_report_sections_3_4" model="account.report">
        <field name="name">Sections III, IV</field>
        <field name="sequence">3</field>
        <field name="country_id" ref="base.lu"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_sections_3_4_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_3_assessment_deducible_tax" model="account.report.line">
                <field name="name">III. ASSESSMENT OF DEDUCTIBLE TAX (input tax)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_3a_total_input_tax" model="account.report.line">
                        <field name="name">093 - Total input tax</field>
                        <field name="code">LUTAX_093</field>
                        <field name="aggregation_formula">LUTAX_090.balance + LUTAX_092.balance + LUTAX_228.balance + LUTAX_458.balance + LUTAX_459.balance + LUTAX_460.balance + LUTAX_461.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_3a_1_invoiced_by_other_taxable_person" model="account.report.line">
                                <field name="name">458 - Invoiced by other taxable persons for goods or services supplied</field>
                                <field name="code">LUTAX_458</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3a_1_invoiced_by_other_taxable_person_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">458</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3a_2_due_respect_intra_comm_goods" model="account.report.line">
                                <field name="name">459 - Due in respect of intra-Community acquisitions of goods</field>
                                <field name="code">LUTAX_459</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3a_2_due_respect_intra_comm_goods_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">459</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3a_3_due_paid_respect_importation_goods" model="account.report.line">
                                <field name="name">460 - Due or paid in respect of importation of goods</field>
                                <field name="code">LUTAX_460</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3a_3_due_paid_respect_importation_goods_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">460</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3a_4_due_respect_application_goods" model="account.report.line">
                                <field name="name">090 - Due in respect of the application of goods for business purposes</field>
                                <field name="code">LUTAX_090</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3a_4_due_respect_application_goods_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">090</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3a_5_due_under_reverse_charge" model="account.report.line">
                                <field name="name">461 - Due under the reverse charge (see points II.E and F)</field>
                                <field name="code">LUTAX_461</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3a_5_due_under_reverse_charge_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">461</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3a_6_paid_joint_several_guarantee" model="account.report.line">
                                <field name="name">092 - Paid as joint and several guarantee</field>
                                <field name="code">LUTAX_092</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3a_6_paid_joint_several_guarantee_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">092</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3a_7_adjusted_tax_special_arrangement" model="account.report.line">
                                <field name="name">228 - Adjusted tax - special arrangement for tax suspension</field>
                                <field name="code">LUTAX_228</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3a_7_adjusted_tax_special_arrangement_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">228</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_3b_total_input_tax_nd" model="account.report.line">
                        <field name="name">097 - Total input tax non-deductible</field>
                        <field name="code">LUTAX_097</field>
                        <field name="aggregation_formula">LUTAX_094.balance + LUTAX_095.balance + LUTAX_096.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_3b1_rel_trans" model="account.report.line">
                                <field name="name">094 - relating to transactions which are exempt pursuant to articles 44 and 56quater</field>
                                <field name="code">LUTAX_094</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3b1_rel_trans_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">094</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3b2_ded_prop" model="account.report.line">
                                <field name="name">095 - where the deductible proportion determined in accordance to article 50 is applied</field>
                                <field name="code">LUTAX_095</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3b2_ded_prop_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">095</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_3b2_input_tax_margin" model="account.report.line">
                                <field name="name">096 - Non recoverable input tax in accordance with Art. 56ter-1(7) and 56ter-2(7) (when applying the margin scheme)</field>
                                <field name="code">LUTAX_096</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_3b2_input_tax_margin_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">096</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_3c_total_input_tax_deductible" model="account.report.line">
                        <field name="name">102 - Total input tax deductible</field>
                        <field name="code">LUTAX_102</field>
                        <field name="aggregation_formula">LUTAX_093.balance - LUTAX_097.balance</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_4_tax_tobe_paid_or_reclaimed" model="account.report.line">
                <field name="name">IV. TAX TO BE PAID OR TO BE RECLAIMED</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_4a_total_tax_due" model="account.report.line">
                        <field name="name">103 - Total tax due</field>
                        <field name="code">LUTAX_103</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_4a_total_tax_due_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">LUTAX_076.balance</field>
                                <field name="subformula">cross_report</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_4a_total_input_tax_deductible" model="account.report.line">
                        <field name="name">104 - Total input tax deductible</field>
                        <field name="code">LUTAX_104</field>
                        <field name="aggregation_formula">LUTAX_102.balance</field>
                    </record>
                    <record id="account_tax_report_line_4c_exceeding_amount" model="account.report.line">
                        <field name="name">105 - Exceeding amount</field>
                        <field name="code">LUTAX_105</field>
                        <field name="aggregation_formula">LUTAX_103.balance - LUTAX_102.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\section_1.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="l10n_lu_tax_report_section_1" model="account.report">
        <field name="name">Section I</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.lu"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_section_1_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_lu_tax_report_assessment_turnover" model="account.report.line">
                <field name="name">I. ASSESSMENT OF TAXABLE TURNOVER</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_1a_overall_turnover" model="account.report.line">
                        <field name="name">012 - Overall turnover</field>
                        <field name="code">LUTAX_012</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">LUTAX_454.balance + LUTAX_455.balance + LUTAX_456.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_1a_total_sale" model="account.report.line">
                                <field name="name">454 - Total Sales / Receipts</field>
                                <field name="code">LUTAX_454</field>
                                <field name="aggregation_formula">LUTAX_471.balance + LUTAX_472.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_1a_telecom_service" model="account.report.line">
                                        <field name="name">471 - Telecommunications services, radio and television broadcasting services...</field>
                                        <field name="code">LUTAX_471</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_1a_telecom_service_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">471</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_1a_other_sales" model="account.report.line">
                                        <field name="name">472 - Other sales / receipts</field>
                                        <field name="code">LUTAX_472</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_1a_other_sales_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">LUTAX_021.balance + LUTAX_022.balance - LUTAX_456.balance - LUTAX_455.balance - LUTAX_471.balance</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1a_app_goods_non_bus" model="account.report.line">
                                <field name="name">455 - Application of goods for non-business use and for business purposes</field>
                                <field name="code">LUTAX_455</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1a_app_goods_non_bus_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">455</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1a_non_bus_gs" model="account.report.line">
                                <field name="name">456 - Non-business use of goods and supply of services free of charge</field>
                                <field name="code">LUTAX_456</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1a_non_bus_gs_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">456</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_1b_exemptions_deductible_amounts" model="account.report.line">
                        <field name="name">021 - Exemptions and deductible amounts</field>
                        <field name="code">LUTAX_021</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">LUTAX_014.balance + LUTAX_457.balance + LUTAX_015.balance + LUTAX_016.balance + LUTAX_017.balance + LUTAX_018.balance + LUTAX_423.balance + LUTAX_424.balance + LUTAX_226.balance + LUTAX_019.balance + LUTAX_481.balance + LUTAX_482.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_1b_1_intra_community_goods_pi_vat" model="account.report.line">
                                <field name="name">457 - Intra-Community supply of goods to persons identified for VAT purposes in another Member State (MS)</field>
                                <field name="code">LUTAX_457</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_1_intra_community_goods_pi_vat_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">457</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_2_export" model="account.report.line">
                                <field name="name">014 - Exports</field>
                                <field name="code">LUTAX_014</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_2_export_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">014</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_3_other_exemptions_art_43" model="account.report.line">
                                <field name="name">015 - Other exemptions</field>
                                <field name="code">LUTAX_015</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_3_other_exemptions_art_43_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">015</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater" model="account.report.line">
                                <field name="name">016 - Other exemptions</field>
                                <field name="code">LUTAX_016</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">016</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_5_manufactured_tobacco_vat_collected" model="account.report.line">
                                <field name="name">017 - Manufactured tobacco whose VAT was collected at the source or at the exit of the tax...</field>
                                <field name="code">LUTAX_017</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_5_manufactured_tobacco_vat_collected_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">017</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_8_supplies_carried_out_domestic" model="account.report.line">
                                <field name="name">481 - Supplies carried out within the scope of the domestic SME scheme of article 57bis (7)</field>
                                <field name="code">LUTAX_481</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_8_supplies_carried_out_domestic_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_9_supplies_carried_out_cross_border" model="account.report.line">
                                <field name="name">482 - Supplies carried out within the scope of the cross-border SME scheme of article 57quater </field>
                                <field name="code">LUTAX_482</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_9_supplies_carried_out_cross_border_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_6_a_subsequent_to_intra_community" model="account.report.line">
                                <field name="name">018 - Supply, subsequent to intra-Community acquisitions of goods, in the context of triangular transactions, when the customer identified,...</field>
                                <field name="code">LUTAX_018</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_6_a_subsequent_to_intra_community_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">018</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_6_b1_non_exempt_customer_vat" model="account.report.line">
                                <field name="name">423 - not exempt in the MS where the customer is liable for payment of VAT</field>
                                <field name="code">LUTAX_423</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_6_b1_non_exempt_customer_vat_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">423</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_6_b2_exempt_ms_customer" model="account.report.line">
                                <field name="name">424 - exempt in the MS where the customer is identified</field>
                                <field name="code">LUTAX_424</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_6_b2_exempt_ms_customer_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">424</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_6_c_supplies_scope_special_arrangement" model="account.report.line">
                                <field name="name">226 - Supplies carried out within the scope of the special arrangement of art. 56sexies</field>
                                <field name="code">LUTAX_226</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_6_c_supplies_scope_special_arrangement_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">226</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_6_d_supplies_other_referred" model="account.report.line">
                                <field name="name">019 - Other supplies carried out (for which the place of supply is) abroad</field>
                                <field name="code">LUTAX_019</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_6_d_supplies_other_referred_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">019</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_1b_7_inland_supplies_for_customer" model="account.report.line">
                                <field name="name">419 - Inland supplies for which the customer is liable for the payment of VAT</field>
                                <field name="code">LUTAX_419</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_1b_7_inland_supplies_for_customer_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">419</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_1c_taxable_turnover" model="account.report.line">
                        <field name="name">022 - Taxable turnover</field>
                        <field name="code">LUTAX_022</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_1c_taxable_turnover_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">LUTAX_037.balance</field>
                                <field name="subformula">cross_report</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\section_2.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="l10n_lu_tax_report_section_2" model="account.report">
        <field name="name">Section II</field>
        <field name="sequence">2</field>
        <field name="country_id" ref="base.lu"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_section_2_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_lu_tax_report_assessment_tax_due" model="account.report.line">
                <field name="name">II. ASSESSMENT OF TAX DUE</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_2a_breakdown_taxable_turnover_base" model="account.report.line">
                        <field name="name">037 - Breakdown of taxable turnover – base</field>
                        <field name="code">LUTAX_037</field>
                        <field name="aggregation_formula">LUTAX_031.balance + LUTAX_033.balance + LUTAX_701.balance + LUTAX_703.balance + LUTAX_705.balance + LUTAX_901.balance + LUTAX_903.balance + LUTAX_905.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2a_base_17" model="account.report.line">
                                <field name="name">701 - base 17%</field>
                                <field name="code">LUTAX_701</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">701</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_base_16" model="account.report.line">
                                <field name="name">901 - base 16%</field>
                                <field name="code">LUTAX_901</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">901</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_base_14" model="account.report.line">
                                <field name="name">703 - base 14%</field>
                                <field name="code">LUTAX_703</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">703</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_base_13" model="account.report.line">
                                <field name="name">903 - base 13%</field>
                                <field name="code">LUTAX_903</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">903</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_base_8" model="account.report.line">
                                <field name="name">705 - base 8%</field>
                                <field name="code">LUTAX_705</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">705</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_base_7" model="account.report.line">
                                <field name="name">905 - base 7%</field>
                                <field name="code">LUTAX_905</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">905</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_base_3" model="account.report.line">
                                <field name="name">031 - base 3%</field>
                                <field name="code">LUTAX_031</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">031</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_base_0" model="account.report.line">
                                <field name="name">033 - base 0%</field>
                                <field name="code">LUTAX_033</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_base_0_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">033</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2a_breakdown_taxable_turnover_tax" model="account.report.line">
                        <field name="name">046 - Breakdown of taxable turnover – tax</field>
                        <field name="code">LUTAX_046</field>
                        <field name="aggregation_formula">LUTAX_040.balance + LUTAX_702.balance + LUTAX_704.balance + LUTAX_706.balance + LUTAX_902.balance + LUTAX_904.balance + LUTAX_906.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2a_tax_17" model="account.report.line">
                                <field name="name">702 - tax 17%</field>
                                <field name="code">LUTAX_702</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_tax_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">702</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_tax_16" model="account.report.line">
                                <field name="name">902 - tax 16%</field>
                                <field name="code">LUTAX_902</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_tax_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">902</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_tax_14" model="account.report.line">
                                <field name="name">704 - tax 14%</field>
                                <field name="code">LUTAX_704</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_tax_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">704</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_tax_13" model="account.report.line">
                                <field name="name">904 - tax 13%</field>
                                <field name="code">LUTAX_904</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_tax_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">904</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_tax_8" model="account.report.line">
                                <field name="name">706 - tax 8%</field>
                                <field name="code">LUTAX_706</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_tax_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">706</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_tax_7" model="account.report.line">
                                <field name="name">906 - tax 7%</field>
                                <field name="code">LUTAX_906</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_tax_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">906</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2a_tax_3" model="account.report.line">
                                <field name="name">040 - tax 3%</field>
                                <field name="code">LUTAX_040</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2a_tax_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">040</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2b_intra_community_acqui_of_goods_base" model="account.report.line">
                        <field name="name">051 - Intra-Community acquisitions of goods – base</field>
                        <field name="code">LUTAX_051</field>
                        <field name="aggregation_formula">LUTAX_049.balance + LUTAX_194.balance + LUTAX_711.balance + LUTAX_713.balance + LUTAX_715.balance + LUTAX_719.balance + LUTAX_911.balance + LUTAX_913.balance + LUTAX_915.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2b_base_17" model="account.report.line">
                                <field name="name">711 - base 17%</field>
                                <field name="code">LUTAX_711</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">711</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_base_16" model="account.report.line">
                                <field name="name">911 - base 16%</field>
                                <field name="code">LUTAX_911</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">911</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_base_14" model="account.report.line">
                                <field name="name">713 - base 14%</field>
                                <field name="code">LUTAX_713</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">713</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_base_13" model="account.report.line">
                                <field name="name">913 - base 13%</field>
                                <field name="code">LUTAX_913</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">913</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_base_8" model="account.report.line">
                                <field name="name">715 - base 8%</field>
                                <field name="code">LUTAX_715</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">715</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_base_7" model="account.report.line">
                                <field name="name">915 - base 7%</field>
                                <field name="code">LUTAX_915</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">915</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_base_3" model="account.report.line">
                                <field name="name">049 - base 3%</field>
                                <field name="code">LUTAX_049</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">049</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_base_exempt" model="account.report.line">
                                <field name="name">194 - base exempt</field>
                                <field name="code">LUTAX_194</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_base_exempt_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">194</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_manufactured_tobacco" model="account.report.line">
                                <field name="name">719 - of manufactured tobacco (VAT is collected at the exit of the tax warehouse with excise duties)</field>
                                <field name="code">LUTAX_719</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_manufactured_tobacco_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">719</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2b_intra_community_acquisitions_goods_tax" model="account.report.line">
                        <field name="name">056 - Intra-Community acquisitions of goods – tax</field>
                        <field name="code">LUTAX_056</field>
                        <field name="aggregation_formula">LUTAX_054.balance + LUTAX_712.balance + LUTAX_714.balance + LUTAX_716.balance + LUTAX_912.balance + LUTAX_914.balance + LUTAX_916.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2b_tax_17" model="account.report.line">
                                <field name="name">712 - tax 17%</field>
                                <field name="code">LUTAX_712</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_tax_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">712</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_tax_16" model="account.report.line">
                                <field name="name">912 - tax 16%</field>
                                <field name="code">LUTAX_912</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_tax_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">912</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_tax_14" model="account.report.line">
                                <field name="name">714 - tax 14%</field>
                                <field name="code">LUTAX_714</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_tax_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">714</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_tax_13" model="account.report.line">
                                <field name="name">914 - tax 13%</field>
                                <field name="code">LUTAX_914</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_tax_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">914</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_tax_8" model="account.report.line">
                                <field name="name">716 - tax 8%</field>
                                <field name="code">LUTAX_716</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_tax_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">716</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_tax_7" model="account.report.line">
                                <field name="name">916 - tax 7%</field>
                                <field name="code">LUTAX_916</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_tax_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">916</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2b_tax_3" model="account.report.line">
                                <field name="name">054 - tax 3%</field>
                                <field name="code">LUTAX_054</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2b_tax_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">054</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2c_acquisitions_triangular_transactions_base" model="account.report.line">
                        <field name="name">152 - Acquisitions, in the context of triangular transactions – base</field>
                        <field name="code">LUTAX_152</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_2c_acquisitions_triangular_transactions_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">152</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2d_importation_of_goods_base" model="account.report.line">
                        <field name="name">065 - Importation of goods – base</field>
                        <field name="code">LUTAX_065</field>
                        <field name="aggregation_formula">LUTAX_196.balance + LUTAX_721.balance + LUTAX_723.balance + LUTAX_725.balance + LUTAX_731.balance + LUTAX_733.balance + LUTAX_735.balance + LUTAX_059.balance + LUTAX_063.balance + LUTAX_195.balance + LUTAX_729.balance + LUTAX_921.balance + LUTAX_923.balance + LUTAX_925.balance + LUTAX_931.balance + LUTAX_933.balance + LUTAX_935.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2d_1_base_17" model="account.report.line">
                                <field name="name">721 - for business purposes: base 17%</field>
                                <field name="code">LUTAX_721</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">721</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_base_16" model="account.report.line">
                                <field name="name">921 - for business purposes: base 16%</field>
                                <field name="code">LUTAX_921</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">921</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_base_14" model="account.report.line">
                                <field name="name">723 - for business purposes: base 14%</field>
                                <field name="code">LUTAX_723</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">723</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_base_13" model="account.report.line">
                                <field name="name">923 - for business purposes: base 13%</field>
                                <field name="code">LUTAX_923</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">923</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_base_8" model="account.report.line">
                                <field name="name">725 - for business purposes: base 8%</field>
                                <field name="code">LUTAX_725</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">725</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_base_7" model="account.report.line">
                                <field name="name">925 - for business purposes: base 7%</field>
                                <field name="code">LUTAX_925</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">925</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_base_3" model="account.report.line">
                                <field name="name">059 - for business purposes: base 3%</field>
                                <field name="code">LUTAX_059</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">059</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_base_exempt" model="account.report.line">
                                <field name="name">195 - for business purposes: base exempt</field>
                                <field name="code">LUTAX_195</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_base_exempt_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">195</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_manufactured_tobacco" model="account.report.line">
                                <field name="name">729 - of manufactured tobacco (VAT is collected at the exit of the tax warehouse with excise duties)</field>
                                <field name="code">LUTAX_729</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_manufactured_tobacco_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">729</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_17" model="account.report.line">
                                <field name="name">731 - for non-business purposes: base 17%</field>
                                <field name="code">LUTAX_731</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">731</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_16" model="account.report.line">
                                <field name="name">931 - for non-business purposes: base 16%</field>
                                <field name="code">LUTAX_931</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">931</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_14" model="account.report.line">
                                <field name="name">733 - for non-business purposes: base 14%</field>
                                <field name="code">LUTAX_733</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">733</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_13" model="account.report.line">
                                <field name="name">933 - for non-business purposes: base 13%</field>
                                <field name="code">LUTAX_933</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">933</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_8" model="account.report.line">
                                <field name="name">735 - for non-business purposes: base 8%</field>
                                <field name="code">LUTAX_735</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">735</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_7" model="account.report.line">
                                <field name="name">935 - for non-business purposes: base 7%</field>
                                <field name="code">LUTAX_935</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">935</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_3" model="account.report.line">
                                <field name="name">063 - for non-business purposes: base 3%</field>
                                <field name="code">LUTAX_063</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">063</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_base_exempt" model="account.report.line">
                                <field name="name">196 - for non-business purposes: base exempt</field>
                                <field name="code">LUTAX_196</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_base_exempt_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">196</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2d_importation_of_goods_tax" model="account.report.line">
                        <field name="name">407 - Importation of goods – tax</field>
                        <field name="code">LUTAX_407</field>
                        <field name="aggregation_formula">LUTAX_724.balance + LUTAX_726.balance + LUTAX_732.balance + LUTAX_734.balance + LUTAX_736.balance + LUTAX_068.balance + LUTAX_073.balance + LUTAX_722.balance + LUTAX_922.balance + LUTAX_924.balance + LUTAX_926.balance + LUTAX_932.balance + LUTAX_934.balance + LUTAX_936.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2d_1_tax_17" model="account.report.line">
                                <field name="name">722 - for business purposes: tax 17%</field>
                                <field name="code">LUTAX_722</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_tax_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">722</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_tax_16" model="account.report.line">
                                <field name="name">922 - for business purposes: tax 16%</field>
                                <field name="code">LUTAX_922</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_tax_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">922</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_tax_14" model="account.report.line">
                                <field name="name">724 - for business purposes: tax 14%</field>
                                <field name="code">LUTAX_724</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_tax_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">724</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_tax_13" model="account.report.line">
                                <field name="name">924 - for business purposes: tax 13%</field>
                                <field name="code">LUTAX_924</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_tax_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">924</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_tax_8" model="account.report.line">
                                <field name="name">726 - for business purposes: tax 8%</field>
                                <field name="code">LUTAX_726</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_tax_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">726</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_tax_7" model="account.report.line">
                                <field name="name">926 - for business purposes: tax 7%</field>
                                <field name="code">LUTAX_926</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_tax_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">926</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_1_tax_3" model="account.report.line">
                                <field name="name">068 - for business purposes: tax 3%</field>
                                <field name="code">LUTAX_068</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_1_tax_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">068</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_tax_17" model="account.report.line">
                                <field name="name">732 - for non-business purposes: tax 17%</field>
                                <field name="code">LUTAX_732</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_tax_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">732</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_tax_16" model="account.report.line">
                                <field name="name">932 - for non-business purposes: tax 16%</field>
                                <field name="code">LUTAX_932</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_tax_16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">932</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_tax_14" model="account.report.line">
                                <field name="name">734 - for non-business purposes: tax 14%</field>
                                <field name="code">LUTAX_734</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_tax_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">734</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_tax_13" model="account.report.line">
                                <field name="name">934 - for non-business purposes: tax 13%</field>
                                <field name="code">LUTAX_934</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_tax_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">934</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_tax_8" model="account.report.line">
                                <field name="name">736 - for non-business purposes: tax 8%</field>
                                <field name="code">LUTAX_736</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_tax_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">736</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_tax_7" model="account.report.line">
                                <field name="name">936 - for non-business purposes: tax 7%</field>
                                <field name="code">LUTAX_936</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_tax_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">936</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2d_2_tax_3" model="account.report.line">
                                <field name="name">073 - for non-business purposes: tax 3%</field>
                                <field name="code">LUTAX_073</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2d_2_tax_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">073</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2e_supply_of_service_for_customer" model="account.report.line">
                        <field name="name">409 - Supply of services for which the customer is liable for the payment of VAT – base</field>
                        <field name="code">LUTAX_409</field>
                        <field name="aggregation_formula">LUTAX_436.balance + LUTAX_435.balance + LUTAX_463.balance + LUTAX_765.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2e_1_base" model="account.report.line">
                                <field name="name">436 - base</field>
                                <field name="code">LUTAX_436</field>
                                <field name="aggregation_formula">LUTAX_431.balance + LUTAX_741.balance + LUTAX_743.balance + LUTAX_745.balance + LUTAX_941.balance + LUTAX_943.balance + LUTAX_945.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_2e_1_a_base_17" model="account.report.line">
                                        <field name="name">741 - not exempt within the territory: base 17%</field>
                                        <field name="code">LUTAX_741</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_base_17_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">741</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_base_16" model="account.report.line">
                                        <field name="name">941 - not exempt within the territory: base 16%</field>
                                        <field name="code">LUTAX_941</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_base_16_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">941</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_base_14" model="account.report.line">
                                        <field name="name">743 - not exempt within the territory: base 14%</field>
                                        <field name="code">LUTAX_743</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_base_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">743</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_base_13" model="account.report.line">
                                        <field name="name">943 - not exempt within the territory: base 13%</field>
                                        <field name="code">LUTAX_943</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_base_13_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">943</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_base_8" model="account.report.line">
                                        <field name="name">745 - not exempt within the territory: base 8%</field>
                                        <field name="code">LUTAX_745</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_base_8_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">745</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_base_7" model="account.report.line">
                                        <field name="name">945 - not exempt within the territory: base 7%</field>
                                        <field name="code">LUTAX_945</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_base_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">945</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_base_3" model="account.report.line">
                                        <field name="name">431 - not exempt within the territory: base 3%</field>
                                        <field name="code">LUTAX_431</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_base_3_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">431</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2e_1_b_exempt" model="account.report.line">
                                <field name="name">435 - exempt within the territory: exempt</field>
                                <field name="code">LUTAX_435</field>
                                <field name="hide_if_zero" eval="1"/>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2e_1_b_exempt_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">435</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2e_2_base" model="account.report.line">
                                <field name="name">463 - base</field>
                                <field name="code">LUTAX_463</field>
                                <field name="aggregation_formula">LUTAX_441.balance + LUTAX_445.balance + LUTAX_751.balance + LUTAX_753.balance + LUTAX_755.balance + LUTAX_951.balance + LUTAX_953.balance + LUTAX_955.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_2e_2_base_17" model="account.report.line">
                                        <field name="name">751 - not established or residing within the Community: base 17%</field>
                                        <field name="code">LUTAX_751</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_base_17_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">751</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_base_16" model="account.report.line">
                                        <field name="name">951 - not established or residing within the Community: base 16%</field>
                                        <field name="code">LUTAX_951</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_base_16_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">951</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_base_14" model="account.report.line">
                                        <field name="name">753 - not established or residing within the Community: base 14%</field>
                                        <field name="code">LUTAX_753</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_base_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">753</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_base_13" model="account.report.line">
                                        <field name="name">953 - not established or residing within the Community: base 13%</field>
                                        <field name="code">LUTAX_953</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_base_13_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">953</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_base_8" model="account.report.line">
                                        <field name="name">755 - not established or residing within the Community: base 8%</field>
                                        <field name="code">LUTAX_755</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_base_8_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">755</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_base_7" model="account.report.line">
                                        <field name="name">955 - not established or residing within the Community: base 7%</field>
                                        <field name="code">LUTAX_955</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_base_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">955</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_base_3" model="account.report.line">
                                        <field name="name">441 - not established or residing within the Community: base 3%</field>
                                        <field name="code">LUTAX_441</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_base_3_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">441</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_exempt" model="account.report.line">
                                        <field name="name">445 - not established or residing within the Community: exempt</field>
                                        <field name="code">LUTAX_445</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_exempt_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">445</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2e_3_base" model="account.report.line">
                                <field name="name">765 - base</field>
                                <field name="code">LUTAX_765</field>
                                <field name="aggregation_formula">LUTAX_761.balance + LUTAX_961.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_2e_3_base_17" model="account.report.line">
                                        <field name="name">761 - suppliers established within the territory: base 17%</field>
                                        <field name="code">LUTAX_761</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_3_base_17_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">761</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_3_base_16" model="account.report.line">
                                        <field name="name">961 - suppliers established within the territory: base 16%</field>
                                        <field name="code">LUTAX_961</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_3_base_16_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">961</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax" model="account.report.line">
                        <field name="name">410 - Supply of services for which the customer is liable for the payment of VAT – tax</field>
                        <field name="code">LUTAX_410</field>
                        <field name="aggregation_formula">LUTAX_462.balance + LUTAX_464.balance + LUTAX_766.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2e_1_a_tax" model="account.report.line">
                                <field name="name">462 - tax</field>
                                <field name="code">LUTAX_462</field>
                                <field name="aggregation_formula">LUTAX_432.balance + LUTAX_742.balance + LUTAX_744.balance + LUTAX_746.balance + LUTAX_942.balance + LUTAX_944.balance + LUTAX_946.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_2e_1_a_tax_17" model="account.report.line">
                                        <field name="name">742 - not exempt within the territory: tax 17%</field>
                                        <field name="code">LUTAX_742</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_tax_17_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">742</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_tax_16" model="account.report.line">
                                        <field name="name">942 - not exempt within the territory: tax 16%</field>
                                        <field name="code">LUTAX_942</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_tax_16_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">942</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_tax_14" model="account.report.line">
                                        <field name="name">744 - not exempt within the territory: tax 14%</field>
                                        <field name="code">LUTAX_744</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_tax_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">744</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_tax_13" model="account.report.line">
                                        <field name="name">944 - not exempt within the territory: tax 13%</field>
                                        <field name="code">LUTAX_944</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_tax_13_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">944</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_tax_8" model="account.report.line">
                                        <field name="name">746 - not exempt within the territory: tax 8%</field>
                                        <field name="code">LUTAX_746</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_tax_8_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">746</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_tax_7" model="account.report.line">
                                        <field name="name">946 - not exempt within the territory: tax 7%</field>
                                        <field name="code">LUTAX_946</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_tax_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">946</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_1_a_tax_3" model="account.report.line">
                                        <field name="name">432 - not exempt within the territory: tax 3%</field>
                                        <field name="code">LUTAX_432</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_1_a_tax_3_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">432</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2e_2_tax" model="account.report.line">
                                <field name="name">464 - tax</field>
                                <field name="code">LUTAX_464</field>
                                <field name="aggregation_formula">LUTAX_442.balance + LUTAX_752.balance + LUTAX_754.balance + LUTAX_756.balance + LUTAX_952.balance + LUTAX_954.balance + LUTAX_956.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_2e_2_tax_17" model="account.report.line">
                                        <field name="name">752 - not established or residing within the Community: tax 17%</field>
                                        <field name="code">LUTAX_752</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_tax_17_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">752</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_tax_16" model="account.report.line">
                                        <field name="name">952 - not established or residing within the Community: tax 16%</field>
                                        <field name="code">LUTAX_952</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_tax_16_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">952</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_tax_14" model="account.report.line">
                                        <field name="name">754 - not established or residing within the Community: tax 14%</field>
                                        <field name="code">LUTAX_754</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_tax_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">754</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_tax_13" model="account.report.line">
                                        <field name="name">954 - not established or residing within the Community: tax 13%</field>
                                        <field name="code">LUTAX_954</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_tax_13_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">954</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_tax_8" model="account.report.line">
                                        <field name="name">756 - not established or residing within the Community: tax 8%</field>
                                        <field name="code">LUTAX_756</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_tax_8_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">756</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_tax_7" model="account.report.line">
                                        <field name="name">956 - not established or residing within the Community: tax 7%</field>
                                        <field name="code">LUTAX_956</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_tax_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">956</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_2_tax_3" model="account.report.line">
                                        <field name="name">442 - not established or residing within the Community: tax 3%</field>
                                        <field name="code">LUTAX_442</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_2_tax_3_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">442</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2e_3_tax" model="account.report.line">
                                <field name="name">766 - tax</field>
                                <field name="code">LUTAX_766</field>
                                <field name="aggregation_formula">LUTAX_762.balance + LUTAX_962.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_2e_3_tax_17" model="account.report.line">
                                        <field name="name">762 - suppliers established within the territory: tax 17%</field>
                                        <field name="code">LUTAX_762</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_3_tax_17_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">762</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_2e_3_tax_16" model="account.report.line">
                                        <field name="name">962 - suppliers established within the territory: tax 16%</field>
                                        <field name="code">LUTAX_962</field>
                                        <field name="hide_if_zero" eval="1"/>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_2e_3_tax_16_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">962</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2f_supply_goods_base" model="account.report.line">
                        <field name="name">767 - Supply of goods for which the purchaser is liable for the payment of VAT - base</field>
                        <field name="code">LUTAX_767</field>
                        <field name="aggregation_formula">LUTAX_769.balance + LUTAX_763.balance + LUTAX_963.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2f_supply_goods_base_17" model="account.report.line">
                                <field name="name">769 - base 17%</field>
                                <field name="code">LUTAX_769</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2f_supply_goods_base_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">769</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2f_supply_goods_base_8" model="account.report.line">
                                <field name="name">763 - base 8%</field>
                                <field name="code">LUTAX_763</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2f_supply_goods_base_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">763</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2f_supply_goods_base_7" model="account.report.line">
                                <field name="name">963 - base 7%</field>
                                <field name="code">LUTAX_963</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2f_supply_goods_base_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">963</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2f_supply_goods_tax" model="account.report.line">
                        <field name="name">768 - Supply of goods for which the purchaser is liable for the payment of VAT - tax</field>
                        <field name="code">LUTAX_768</field>
                        <field name="aggregation_formula">LUTAX_770.balance + LUTAX_764.balance + LUTAX_964.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_2f_supply_goods_tax_17" model="account.report.line">
                                <field name="name">770 - tax 17%</field>
                                <field name="code">LUTAX_770</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2f_supply_goods_tax_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">770</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2f_supply_goods_tax_8" model="account.report.line">
                                <field name="name">764 - tax 8%</field>
                                <field name="code">LUTAX_764</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2f_supply_goods_tax_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">764</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_2f_supply_goods_tax_7" model="account.report.line">
                                <field name="name">964 - tax 7%</field>
                                <field name="code">LUTAX_964</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_2f_supply_goods_tax_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">964</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2g_special_arrangement" model="account.report.line">
                        <field name="name">227 - Special arrangement for tax suspension: adjustment</field>
                        <field name="code">LUTAX_227</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_2g_special_arrangement_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">227</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_2h_total_tax_due" model="account.report.line">
                        <field name="name">076 - Total tax due</field>
                        <field name="code">LUTAX_076</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_2h_total_tax_due_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">LUTAX_046.balance + LUTAX_056.balance + LUTAX_407.balance + LUTAX_410.balance + LUTAX_768.balance + LUTAX_227.balance</field>
                                <field name="subformula">cross_report</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report\tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.lu"/>
        <field name="availability_condition">country</field>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="filter_multi_company">tax_units</field>
        <field name="filter_hierarchy">never</field>
        <field name="use_sections" eval="True"/>
        <field name="section_report_ids" eval="[Command.set([ref('l10n_lu.l10n_lu_tax_report_section_1'),
                                                            ref('l10n_lu.l10n_lu_tax_report_section_2'),
                                                            ref('l10n_lu.l10n_lu_tax_report_sections_3_4')])]"/>
    </record>
</odoo>

```

## File: data\template\account.account-lu.csv

```csv
"id","code","name","account_type","tag_ids","reconcile","name@de","name@fr"
"lu_2011_account_101","101","Subscribed capital","equity","","False","Gezeichnetes Kapital","Capital souscrit"
"lu_2011_account_102","102","Subscribed capital not called","asset_fixed","","False","Gezeichnetes nicht eingefordertes Kapital","Capital souscrit non appelé"
"lu_2011_account_103","103","Subscribed capital called but unpaid","asset_fixed","","False","Gezeichnetes eingefordertes und nicht eingezahltes Kapital","Capital souscrit appelé et non versé"
"lu_2020_account_104","104","Capital of individual companies, corporate partnerships and similar","equity","","False","Kapital von Einzelkaufleuten, Personenhandelsgesellschaften und ähnlichen","Capital des entreprises individuelles, des sociétés de personnes et assimilées"
"lu_2011_account_105","105","Endowment of branches","equity","","False","Dotationskapital von Niederlassungen","Dotation des succursales"
"lu_2011_account_10611","10611","Cash withdrawals (daily life)","equity","","False","Barentnahmen (Lebenshaltungskosten)","Prélèvements en numéraire (train de vie)"
"lu_2011_account_10612","10612","Withdrawals of merchandise, finished products and services (at cost)","equity","","False","Sachentnahme von Waren, von fertigen Erzeugnissen und Dienstleistungen (zu Einstandspreisen)","Prélèvements en nature de marchandises, de produits finis et services (au prix de revient)"
"lu_2011_account_10613","10613","Private share of medical services expenses","equity","","False","Privater Anteil an den Krankheitskosten","Part personnelle des frais de maladie"
"lu_2011_account_106141","106141","Life insurance","equity","","False","Lebensversicherung","Vie"
"lu_2011_account_106142","106142","Accident insurance","equity","","False","Unfallversicherung","Accident"
"lu_2011_account_106143","106143","Fire insurance","equity","","False","Brandschutzversicherung","Incendie"
"lu_2011_account_106144","106144","Third-party insurance","equity","","False","Haftpflichtversicherung","Assurance responsabilité civile"
"lu_2011_account_106145","106145","Full coverage insurance","equity","","False","Mehrgefahrenversicherung","Multirisques"
"lu_2011_account_106148","106148","Other private insurance premiums","equity","","False","Sonstige Beiträge für private Versicherungen","Autres primes d’assurances privées"
"lu_2011_account_106151","106151","Social Security","equity","","False","Sozialversicherungen (Pflegeversicherung)","Assurances sociales (assurance dépendance)"
"lu_2011_account_106152","106152","Child benefit office","equity","","False","Familienzulagen","Allocations familiales"
"lu_2011_account_106153","106153","Health insurance funds","equity","","False","Krankenversicherungen","Cotisations pour mutuelles"
"lu_2011_account_106154","106154","Death and other health insurance funds","equity","","False","Sterbekasse oder sonstige Krankenzusatzversicherungen","Caisse de décès, médico-chirurgicale, Prestaplus"
"lu_2011_account_106158","106158","Other contributions","equity","","False","Sonstige Beiträge","Autres cotisations"
"lu_2011_account_106161","106161","Wages","equity","","False","Löhne","Salaires"
"lu_2011_account_106162","106162","Rent","equity","","False","Miete","Loyer"
"lu_2011_account_106163","106163","Heating, gas, electricity","equity","","False","Heizung, Gas, Strom","Chauffage, gaz, électricité"
"lu_2011_account_106164","106164","Water","equity","","False","Wasser","Eau"
"lu_2011_account_106165","106165","Telephone","equity","","False","Telefon","Téléphone"
"lu_2011_account_106166","106166","Car","equity","","False","Fahrzeug","Voiture"
"lu_2011_account_106168","106168","Other in kind withdrawals","equity","","False","Sonstige Sachentnahmen","Autres prélèvements en nature"
"lu_2011_account_106171","106171","Private furniture","equity","","False","Privates Mobiliar","Mobilier privé"
"lu_2011_account_106172","106172","Private car","equity","","False","Privates Fahrzeug","Voiture privée"
"lu_2011_account_106173","106173","Private held securities","equity","","False","Private Wertpapiere","Titres privés"
"lu_2011_account_106174","106174","Private buildings","equity","","False","Private Gebäude","Immeubles privés"
"lu_2011_account_106178","106178","Other acquisitions","equity","","False","Sonstiger Erwerb","Autres acquisitions"
"lu_2011_account_106181","106181","Income tax paid","equity","","False","Gezahlte Einkommenssteuer","Impôt sur le revenu payé (IRPP)"
"lu_2011_account_106183","106183","Municipal business tax - payment in arrears","equity","","False","Gewerbesteuer - gezahlte Rückstände","Impôt commercial communal (ICC) - arriérés payés"
"lu_2011_account_106188","106188","Other taxes","equity","","False","Sonstige Steuern","Autres impôts"
"lu_2011_account_106191","106191","Repairs to private buildings","equity","","False","Reparaturen an privaten Gebäuden","Réparations aux immeubles privés"
"lu_2011_account_106192","106192","Deposits on private financial accounts","equity","","False","Anlagen auf finanziellen Privatkonten","Placements sur comptes financiers privés"
"lu_2011_account_106193","106193","Refund of private debts","equity","","False","Rückzahlungen privater Schulden","Remboursements de dettes privées"
"lu_2011_account_106194","106194","Gifts and allowance to children","equity","","False","Spenden und Zuwendungen an Kinder","Dons et dotations aux enfants"
"lu_2011_account_106195","106195","Inheritance taxes and mutation tax due to death","equity","","False","Erbschaftssteuern und Wechselsteuer durch Sterbefall","Droits de succession et droits de mutation par décès"
"lu_2011_account_106198","106198","Other special private withdrawals","equity","","False","Sonstige besondere Privatentnahmen","Autres prélèvements privés particuliers"
"lu_2011_account_10621","10621","Inheritance or donation","equity","","False","Erbschaft oder Schenkung","Héritage ou donation"
"lu_2011_account_10622","10622","Personal holdings","equity","","False","Private Guthaben","Avoirs privés"
"lu_2011_account_10623","10623","Private loans","equity","","False","Private Ausleihungen","Emprunts privés"
"lu_2011_account_106241","106241","Private furniture","equity","","False","Privates Mobiliar","Mobilier privé"
"lu_2011_account_106242","106242","Private car","equity","","False","Privates Fahrzeug","Voiture privée"
"lu_2011_account_106243","106243","Private shares / bonds","equity","","False","Private Wertpapiere / Aktien","Titres privés"
"lu_2011_account_106244","106244","Private buildings","equity","","False","Private Gebäude","Immeubles privés"
"lu_2011_account_106248","106248","Other disposals","equity","","False","Sonstige Übertragungen","Autres cessions"
"lu_2011_account_10625","10625","Received rents","equity","","False","Mieteinnahmen","Loyers encaissés"
"lu_2011_account_10626","10626","Received wages or pensions","equity","","False","Eingenommene Löhne oder Renten","Salaires ou rentes touchés"
"lu_2011_account_10627","10627","Received child benefit","equity","","False","Erhaltene Familienzulagen","Allocations familiales reçues"
"lu_2011_account_106281","106281","Income tax","equity","","False","Einkommenssteuer","Impôt sur le revenu (IRPP)"
"lu_2011_account_106284","106284","Municipal business tax (MBT)","equity","","False","Gewerbesteuer","Impôt commercial communal (ICC)"
"lu_2011_account_106288","106288","Other tax refunds","equity","","False","Sonstige Steuerrückzahlungen","Autres remboursements d’impôts"
"lu_2011_account_10629","10629","Business share in private expenses","equity","","False","Gewerblicher Anteil an privaten Ausgaben","Quote-part professionnelle de frais privés"
"lu_2011_account_111","111","Share premium","equity","","False","Ausgabeagio","Primes d'émission"
"lu_2011_account_112","112","Merger premium","equity","","False","Agio bei Verschmelzungen","Primes de fusion"
"lu_2011_account_113","113","Contribution premium","equity","","False","Agio bei Einlagen","Primes d'apport"
"lu_2011_account_114","114","Premiums on conversion of bonds into shares","equity","","False","Agio bei der Umwandlung von Anleihen in Aktien","Primes de conversion d'obligations en actions"
"lu_2011_account_115","115","Capital contribution without issue of shares","equity","","False","Kapitaleinlagen ohne Ausgabe von Anteilen","Apport en capitaux propres non rémunéré par des  titres"
"lu_2011_account_122","122","Reserves in application of the equity method","equity","","False","Rücklagen durch Anwendung der Kapitalanteilsmethode","Réserves de mise en équivalence"
"lu_2011_account_123","123","Temporarily not taxable currency translation adjustments","equity","","False","Rücklagen aus der Währungsumrechnung (unversteuert)","Plus-values sur écarts de conversion immunisées"
"lu_2011_account_128","128","Other revaluation reserves","equity","","False","Sonstige Neubewertungsrücklagen","Autres réserves de réévaluation"
"lu_2011_account_131","131","Legal reserve","equity","","False","Gesetzliche Rücklage","Réserve légale"
"lu_2011_account_132","132","Reserves for own shares or own corporate units","equity","","False","Rücklagen für eigene Aktien oder eigene Anteile","Réserve pour actions propres ou parts propres"
"lu_2011_account_133","133","Reserves provided for by the articles of association","equity","","False","Satzungsmäßige Rücklagen","Réserves statutaires"
"lu_2011_account_1381","1381","Other reserves available for distribution","equity","","False","Sonstige freie Rücklagen","Autres réserves disponibles"
"lu_2020_account_13821","13821","Reserve for net wealth tax (NWT)","equity","","False","Vermögensteuerrücklage","Réserve pour l'impôt sur la fortune (IF)"
"lu_2020_account_13822","13822","Reserves in application of fair value","equity","","False","Rücklagen durch Anwendung des beizulegenden Zeitwerts (Fair Value)","Réserves en application de la juste valeur"
"lu_2020_account_138231","138231","Temporarily not taxable capital gains to reinvest","equity","","False","Sonderposten mit Rücklageanteil, zur Wiederanlage bestimmt","Plus-values immunisées à réinvestir"
"lu_2020_account_138232","138232","Temporarily not taxable capital gains reinvested","equity","","False","Sonderposten mit Rücklageanteil, wiederangelegt","Plus-values immunisées réinvesties"
"lu_2020_account_13828","13828","Reserves not available for distribution not mentioned above","equity","","False","Gebundene Rücklagen welche nicht oben aufgelistet wurden","Réserves non disponibles non visées ci-dessus"
"lu_2020_account_1411","1411","Results brought forward in the process of assignment","equity","","False","Ergebnisvorträge in Zuweisung","Résultats reportés en instance d'affectation"
"lu_2020_account_1412","1412","Results brought forward (assigned)","equity","","False","Ergebnisvorträge (zugewiesen)","Résultats reportés (affectés)"
"lu_2011_account_142","142","Result for the financial year","equity_unaffected","","False","Ergebnis des Geschäftsjahres","Résultat de l'exercice"
"lu_2011_account_15","15","Interim dividends","equity","","False","Vorabdividenden","Acomptes sur dividendes"
"lu_2020_account_1611","1611","Development costs","equity","","False","Entwicklungskosten","Frais de développement"
"lu_2020_account_16121","16121","Acquired against payment (except Goodwill)","equity","","False","Entgeltlich erworben (Firmenwert nicht inbegriffen)","Acquis à titre onéreux"
"lu_2020_account_16122","16122","Created by the undertaking itself","equity","","False","Vom Unternehmen selbst erstellt","Créés par l'entreprise elle-même"
"lu_2020_account_1613","1613","Goodwill acquired for consideration","equity","","False","Geschäfts- oder Firmenwert, soweit er entgeltlich erworben wurde","Fonds de commerce, dans la mesure où il a été acquis à titre onéreux"
"lu_2020_account_1621","1621","Subsidies on land, fitting-outs and buildings","equity","","False","Investitionszuschüsse auf Grundstücke, Erschließungen und Bauten","Subventions sur terrains, aménagements et constructions"
"lu_2020_account_1622","1622","Subsidies on plant and machinery","equity","","False","Investitionszuschüsse auf Ttechnische Anlagen und Maschinen","Subventions sur installations techniques et machines"
"lu_2020_account_1623","1623","Subsidies on other fixtures, fittings, tools and equipment (including rolling stock)","equity","","False","Investitionszuschüsse auf sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Subventions sur autres installations, outillage et mobilier (y compris matériel roulant)"
"lu_2011_account_168","168","Other capital investment subsidies","equity","","False","Sonstige Kapitalsubventionen","Autres subventions d'investissement en capital"
"lu_2011_account_181","181","Provisions for pensions and similar obligations","liability_non_current","","False","Rückstellungen für Pensionen und ähnliche Verpflichtungen","Provisions pour pensions et obligations similaires"
"lu_2020_account_182","182","Provisions for taxation","liability_non_current","","False","Steuerrückstellungen","Provisions pour impôts"
"lu_2011_account_183","183","Deferred tax provisions","liability_non_current","","False","Rückstellungen für latente Steuern","Provisions pour impôts différés"
"lu_2011_account_1881","1881","Operating provisions","liability_non_current","","False","Betriebliche Rückstellungen","Provisions d'exploitation"
"lu_2011_account_1882","1882","Financial provisions","liability_non_current","","False","Finanzielle Rückstellungen","Provisions financières"
"lu_2020_account_1921","1921","Due and payable within one year","liability_current","","False","Mit einer Restlaufzeit von bis zu einem Jahr","Dont la durée résiduelle est inférieure ou égale à un an"
"lu_2020_account_1922","1922","Due and payable after more than one year","liability_non_current","","False","Mit einer Restlaufzeit von mehr als einem Jahr","Dont la durée résiduelle est supérieure à un an"
"lu_2020_account_1931","1931","Due and payable within one year","liability_current","","False","Mit einer Restlaufzeit von bis zu einem Jahr","Dont la durée résiduelle est inférieure ou égale à un an"
"lu_2020_account_1932","1932","Due and payable after more than one year","liability_non_current","","False","Mit einer Restlaufzeit von mehr als einem Jahr","Dont la durée résiduelle est supérieure à un an"
"lu_2020_account_1941","1941","Due and payable within one year","liability_current","","False","Mit einer Restlaufzeit von bis zu einem Jahr","Dont la durée résiduelle est inférieure ou égale à un an"
"lu_2020_account_1942","1942","Due and payable after more than one year","liability_non_current","","False","Mit einer Restlaufzeit von mehr als einem Jahr","Dont la durée résiduelle est supérieure à un an"
"lu_2011_account_201","201","Set-up and start-up costs","asset_fixed","","False","Gründungs- und Einrichtungskosten","Frais de constitution et de premier établissement"
"lu_2011_account_203","203","Expenses for increases in capital and for various operations (merger, demerger, change of legal form)","asset_fixed","","False","Kosten für Kapitalerhöhung und für verschiedene umwandlugsrechtliche Vorgänge (Verschmelzungen, Spaltungen, Formwechsel)","Frais d'augmentation de capital et d'opérations diverses (fusions, scissions, transformations)"
"lu_2011_account_204","204","Loan issuances expenses","asset_fixed","","False","Emissionskosten von Anleihen","Frais d'émission d'emprunts"
"lu_2011_account_208","208","Other similar expenses","asset_fixed","","False","Sonstige ähnliche Kosten","Autres frais assimilés"
"lu_2011_account_211","211","Development costs","asset_fixed","","False","Entwicklungskosten","Frais de développement"
"lu_2011_account_21211","21211","Concessions","asset_fixed","","False","Konzessionen",""
"lu_2011_account_21212","21212","Patents","asset_fixed","","False","Patente","Brevets"
"lu_2011_account_21213","21213","Software licences","asset_fixed","","False","Software- und Softwarepaketlizenzen","Licences informatiques"
"lu_2011_account_21214","21214","Trademarks and franchises","asset_fixed","","False","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"lu_2011_account_212151","212151","Copyrights and reproduction rights","asset_fixed","","False","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"lu_2011_account_212152","212152","Greenhouse gas and similar emission quotas","asset_fixed","","False","Kontigente für Treibhausgasemissionen und ähnliche","Quotas d'émission de gaz à effet de serre et assimilés"
"lu_2011_account_212158","212158","Other similar rights and assets acquired for consideration","asset_fixed","","False","Sonstige entgeltlich erworbene Rechte und Werte","Autres droits et valeurs similaires acquis à titre onéreux"
"lu_2011_account_21221","21221","Concessions","asset_fixed","","False","Konzessionen",""
"lu_2011_account_21222","21222","Patents","asset_fixed","","False","Patente","Brevets"
"lu_2011_account_21223","21223","Software licences","asset_fixed","","False","Software- und Softwarepaketlizenzen","Licences informatiques"
"lu_2011_account_21224","21224","Trademarks and franchises","asset_fixed","","False","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"lu_2011_account_212251","212251","Copyrights and reproduction rights","asset_fixed","","False","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"lu_2011_account_212258","212258","Other similar rights and assets created by the undertaking itself","asset_fixed","","False","Sonstige vergleichbare vom Unternehmen selbst erstellte Rechte und Werte","Autres droits et valeurs similaires créés par l'entreprise elle-même"
"lu_2011_account_213","213","Goodwill acquired for consideration","asset_fixed","","False","Geschäfts- oder Firmenwert, soweit er entgeltlich erworben wurde","Fonds de commerce, dans la mesure où il a été acquis à titre onéreux"
"lu_2020_account_214","214","Down payments and intangible fixed assets under development","asset_fixed","","False","Geleistete Anzahlungen und immaterielle Vermögensgegenstände in Entwicklung","Acomptes versés et immobilisations incorporelles en cours"
"lu_2020_account_221111","221111","Developed land","asset_fixed","","False","Bebaute Grundstücke","Terrains bâtis"
"lu_2020_account_221112","221112","Property rights and similar","asset_fixed","","False","Immobilien- / Eigentumsrechte auf Sachanlagen und ähnliche","Droits immobiliers et assimilés"
"lu_2020_account_221118","221118","Other land","asset_fixed","","False","Sonstige Grundstücke","Autres terrains"
"lu_2011_account_22112","22112","Land in foreign countries","asset_fixed","","False","Grundstücke im Ausland","Terrains à l'étranger"
"lu_2011_account_22121","22121","Fixtures and fitting-outs of land in Luxembourg","asset_fixed","","False","Erschließung von Grundstücken in Luxembourg","Agencements et aménagements de terrains au Luxembourg"
"lu_2011_account_22122","22122","Fixtures and fitting-outs of land in foreign countries","asset_fixed","","False","Erschließung von Grundstücken im Ausland","Agencements et aménagements de terrains à l'étranger"
"lu_2020_account_221311","221311","Residential buildings","asset_fixed","","False","Wohngebäude","Constructions / Bâtiments résidentiels"
"lu_2020_account_221312","221312","Non-residential buildings","asset_fixed","","False","Zweckbauten","Constructions / Bâtiments non résidentiels"
"lu_2020_account_221313","221313","Mixed-use buildings","asset_fixed","","False","Mischnutzungsbauten","Constructions / Bâtiments à usage mixte"
"lu_2020_account_221318","221318","Other buildings","asset_fixed","","False","Sonstige Bauten / Gebäude","Autres constructions / bâtiments"
"lu_2011_account_22132","22132","Buildings in foreign countries","asset_fixed","","False","Bauten / Gebäude im Ausland","Constructions / Bâtiments à l'étranger"
"lu_2020_account_22141","22141","Fixtures and fitting-outs of buildings in Luxembourg","asset_fixed","","False","Einrichtung von Bauten / Gebäuden in Luxemburg","Agencements et aménagements de constructions / bâtiments au Luxembourg"
"lu_2020_account_22142","22142","Fixtures and fitting-outs of buildings in foreign countries","asset_fixed","","False","Einrichtung von Bauten / Gebäuden im Ausland","Agencements et aménagements de constructions / bâtiments à l'étranger"
"lu_2020_account_22151","22151","Investment properties in Luxembourg","asset_fixed","","False","Anlageimmobilien in Luxembourg","Immeubles de placement au Luxembourg"
"lu_2020_account_22152","22152","Investment properties in foreign countries","asset_fixed","","False","Anlageimmobilien im Ausland","Immeubles de placement à l'étranger"
"lu_2011_account_2221","2221","Plant","asset_fixed","","False","Technische Anlagen","Installations techniques"
"lu_2011_account_2222","2222","Machinery","asset_fixed","","False","Maschinen","Machines"
"lu_2011_account_2231","2231","Transportation and handling equipment","asset_fixed","","False","Transport- und Wartungsmittel","Equipement de transport et de manutention"
"lu_2011_account_2232","2232","Motor vehicles","asset_fixed","","False","Transportfahrzeuge","Véhicules de transport"
"lu_2011_account_2233","2233","Tools","asset_fixed","","False","Werkzeuge","Outillage"
"lu_2011_account_2234","2234","Furniture","asset_fixed","","False","Mobiliar","Mobilier"
"lu_2011_account_2235","2235","Computer equipment","asset_fixed","","False","IT-Ausstattung","Matériel informatique"
"lu_2011_account_2236","2236","Livestock","asset_fixed","","False","Viehbestand","Cheptel"
"lu_2011_account_2237","2237","Returnable packaging","asset_fixed","","False","Wiederverwendbare Verpackungen","Emballages récupérables"
"lu_2011_account_2238","2238","Other fixtures","asset_fixed","","False","Sonstige Anlagen","Autres installations"
"lu_2011_account_22411","22411","Land, fitting-outs and buildings in Luxembourg","asset_fixed","","False","Grundstücke, Erschließungen, Einrichtungen und Bauten in Luxemburg","Terrains, aménagements et constructions au Luxembourg"
"lu_2011_account_22412","22412","Land, fitting-outs and buildings in foreign countries","asset_fixed","","False","Grundstücke, Erschließungen, Einrichtungen und Bauten im Ausland","Terrains, aménagements et constructions à l'étranger"
"lu_2011_account_2242","2242","Plant and machinery","asset_fixed","","False","Technische Anlagen und Maschinen","Installations techniques et machines"
"lu_2011_account_2243","2243","Other fixtures and fittings, tools and equipment (including rolling stock)","asset_fixed","","False","Sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Autres installations, outillage et mobilier (y compris matériel roulant)"
"lu_2011_account_231","231","Shares in affiliated undertakings","asset_fixed","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2011_account_232","232","Amounts owed by affiliated undertakings","asset_fixed","","False","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"lu_2011_account_233","233","Participating interests","asset_fixed","","False","Anteile","Participations"
"lu_2011_account_234","234","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","asset_fixed","","False","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_235111","235111","Listed shares","asset_fixed","","False","Notierte Aktien","Actions cotées"
"lu_2020_account_235112","235112","Unlisted shares","asset_fixed","","False","Nicht notierte Aktien","Actions non cotées"
"lu_2011_account_23518","23518","Other securities held as fixed assets (equity right)","asset_fixed","","False","Sonstige Wertpapiere des Anlagevermögens (Eigentumsrecht)","Autres titres immobilisés (droit de propriété)"
"lu_2011_account_23521","23521","Debentures","asset_fixed","","False","Anleihen","Obligations"
"lu_2011_account_23528","23528","Other securities held as fixed assets (creditor's right)","asset_fixed","","False","Sonstige Wertpapiere des Anlagevermögens (Forderungsrecht)","Autres titres immobilisés (droit de créance)"
"lu_2020_account_2353","2353","Shares of collective investment funds","asset_fixed","","False","OPC Anteile","Parts d'OPC"
"lu_2011_account_2358","2358","Other securities held as fixed assets","asset_fixed","","False","Sonstige Wertpapiere des Anlagevermögens","Autres titres ayant le caractère d'immobilisations"
"lu_2020_account_2361","2361","Loans","asset_fixed","","False","Ausleihungen","Prêts"
"lu_2020_account_2362","2362","Deposits and guarantees paid","asset_fixed","","False","Geleistete Hinterlegungen und Kautionen","Dépôts et cautionnements versés"
"lu_2011_account_2363","2363","Long-term receivables","asset_fixed","","False","Forderungen","Créances immobilisées"
"lu_2011_account_301","301","Inventories of raw materials","asset_current","","False","Vorräte an Rohstoffen","Stocks de matières premières"
"lu_2020_account_303","303","Inventories of consumable materials and supplies","asset_current","","False","Vorräte an Hilfs-und Betriebsstoffen","Stocks de matières et fournitures consommables"
"lu_2020_account_304","304","Inventories of packaging","asset_current","","False","Vorräte an Verpackungen","Stocks d'emballages"
"lu_2011_account_311","311","Inventories of work in progress","asset_current","","False","Vorräte an unfertigen Erzeugnissen","Stocks de produits en cours de fabrication"
"lu_2011_account_312","312","Contracts in progress - goods","asset_current","","False","In Arbeit befindlichen Aufträge - Waren","Commandes en cours – produits"
"lu_2011_account_313","313","Contracts in progress - services","asset_current","","False","In Arbeit befindlichen Aufträge - Dienstleistungen","Commandes en cours – prestations de services"
"lu_2011_account_314","314","Buildings under construction","asset_current","","False","Im Bau befindliche Bauten / Gebäude","Immeubles en construction"
"lu_2011_account_321","321","Inventories of finished goods","asset_current","","False","Vorräte an fertigen Erzeugnissen","Stocks de produits finis"
"lu_2011_account_322","322","Inventories of semi-finished goods","asset_current","","False","Vorräte an Zwischenprodukten","Stocks de produits intermédiaires"
"lu_2020_account_323","323","Inventories of residual goods (waste, rejected and recuperable material)","asset_current","","False","Vorräte an Restprodukten","Stocks de produits résiduels (déchets, rebuts, matières de récupération)"
"lu_2020_account_361","361","Inventories of merchandise","asset_current","","False","Vorräte an Waren","Stocks de marchandises"
"lu_2020_account_3621","3621","Inventories of land for resale in Luxembourg","asset_current","","False","Vorräte an, zum Verkauf bestimmten, Grundstücken in Luxemburg","Stocks de terrains au Luxembourg"
"lu_2020_account_3622","3622","Inventories of land for resale in foreign countries","asset_current","","False","Vorräte an, zum Verkauf bestimmten, Grundstücken im Ausland","Stocks de terrains à l'étranger"
"lu_2020_account_3631","3631","Inventories of buildings for resale in Luxembourg","asset_current","","False","Vorräte an, zum Verkauf bestimmten, Bauten / Gebäuden in Luxemburg","Stocks d'immeubles au Luxembourg"
"lu_2020_account_3632","3632","Inventories of buildings for resale in foreign countries","asset_current","","False","Vorräte an, zum Verkauf bestimmten, Bauten / Gebäuden im Ausland","Stocks d'immeubles à l'étranger"
"lu_2020_account_37","37","Down payments on account on inventories","asset_current","","False","Geleistete Anzahlungen auf Vorräte","Acomptes versés sur stocks"
"lu_2011_account_4011","4011","Customers","asset_receivable","","True","Kunden","Clients"
"lu_2011_account_40111","40111","Customers (PoS)","asset_receivable","","True","Kunden (POS)","Clients (POS)"
"lu_2011_account_4012","4012","Customers - Receivable bills of exchange","asset_current","","True","Kunden - Einzutreibende Wechsel (Besitzwechsel)","Clients – Effets à recevoir"
"lu_2011_account_4013","4013","Doubtful or disputed customers","asset_current","","True","Zweifelhafte oder strittige Kunden","Clients douteux ou litigieux"
"lu_2011_account_4014","4014","Customers - Unbilled sales","asset_current","","True","Noch auszustellende Rechnungen","Clients – Factures à établir"
"lu_2011_account_4015","4015","Customers with a credit balance","liability_current","","True","Kunden - Kreditorische Debitoren","Clients créditeurs"
"lu_2011_account_4019","4019","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2011_account_4021","4021","Customers","asset_receivable","","True","Kunden","Clients"
"lu_2011_account_4025","4025","Customers with creditor balance","asset_current","","True","Kreditorische Debitoren","Clients créditeurs"
"lu_2011_account_4029","4029","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2011_account_41111","41111","Trade receivables","asset_current","","False","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"lu_2011_account_41112","41112","Loans and advances","asset_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_41118","41118","Other receivables","asset_current","","False","Sonstige Forderungen","Autres créances"
"lu_2011_account_41119","41119","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2011_account_41121","41121","Trade receivables","asset_current","","False","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"lu_2011_account_41122","41122","Loans and advances","asset_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_41128","41128","Other receivables","asset_current","","False","Sonstige Forderungen","Autres créances"
"lu_2011_account_41129","41129","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2011_account_41211","41211","Trade receivables","asset_current","","False","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"lu_2011_account_41212","41212","Loans and advances","asset_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_41218","41218","Other receivables","asset_current","","False","Sonstige Forderungen","Autres créances"
"lu_2011_account_41219","41219","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2011_account_41221","41221","Trade receivables","asset_current","","False","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"lu_2011_account_41222","41222","Loans and advances","asset_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_41228","41228","Other receivables","asset_current","","False","Sonstige Forderungen","Autres créances"
"lu_2011_account_41229","41229","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2011_account_42111","42111","Advances and down payments","asset_current","","False","Vorschüsse und geleistete Anzahlungen","Avances et acomptes"
"lu_2011_account_42119","42119","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2020_account_4212","4212","Amounts owed by partners and shareholders (others than from affiliated undertakings)","asset_current","","False","Forderungen gegen Gesellschafter und Aktionäre (andere als von verbundenen Unternehmen)","Créances sur associés ou actionnaires (autres qu'entreprises liées)"
"lu_2011_account_42131","42131","Investment subsidies","asset_current","","False","Investitionszuschüsse","Subventions d'investissement"
"lu_2011_account_42132","42132","Operating subsidies","asset_current","","False","Betriebszuschüsse","Subventions d'exploitation"
"lu_2011_account_42138","42138","Other subsidies","asset_current","","False","Sonstige Zuschüsse","Autres subventions"
"lu_2020_account_42141","42141","Corporate income tax","asset_current","","False","Körperschaftssteuer","Impôt sur le revenu des collectivités (IRC)"
"lu_2020_account_42142","42142","Municipal business tax","asset_current","","False","Gewerbesteuer","Impôt commercial communal (ICC)"
"lu_2020_account_42143","42143","Net wealth tax","asset_current","","False","Vermögensteuer","Impôt sur la fortune (IF)"
"lu_2020_account_42144","42144","Withholding tax on wages and salaries","asset_current","","False","Einbehaltene Steuer auf Gehälter und Löhne","Retenue d'impôt sur traitements et salaires (RTS)"
"lu_2020_account_42145","42145","Withholding tax on financial investment income","asset_current","","False","Einbehaltene Kapitalertragsteuer","Retenue d'impôt sur revenus de capitaux mobiliers"
"lu_2020_account_42146","42146","Withholding tax on director's fees","asset_current","","False","Einbehaltene Steuer auf Tantiemen","Retenue d'impôt sur les tantièmes"
"lu_2020_account_42148","42148","ACD - Other amounts receivable","asset_current","","False","ACD - Sonstige Verbindlichkeiten","ACD – Autres créances"
"lu_2011_account_4215","4215","Customs and Excise Authority (ADA)","asset_current","","False","Zoll- und Verbrauchssteuerverwaltung (ADA)","Administration des Douanes et Accises (ADA)"
"lu_2020_account_421611","421611","VAT paid and recoverable","asset_current","","False","Vorsteuer","TVA en amont"
"lu_2011_account_421612","421612","VAT receivable","asset_current","","False","Zu erhaltende MwSt","TVA à recevoir"
"lu_2011_account_421613","421613","VAT down payments made","asset_current","","False","Geleistete Anzahlungen auf MwSt","TVA acomptes versés"
"lu_2011_account_421618","421618","VAT - Other receivables","asset_current","","False","MwSt - Sonstige Forderungen","TVA – Autres créances"
"lu_2011_account_421621","421621","Registration duties","asset_current","","False","Registriergebühren","Droits d'enregistrement"
"lu_2011_account_421622","421622","Subscription tax","asset_current","","False","Abgeltungssteuer","Taxe d'abonnement"
"lu_2011_account_421628","421628","Other indirect taxes","asset_current","","False","Sonstige indirekte Steuern","Autres impôts indirects"
"lu_2011_account_42168","42168","Other receivables","asset_current","","False","Sonstige Forderungen","Autres créances"
"lu_2011_account_42171","42171","Social Security office (CCSS)","asset_current","","False","Sozialversicherungszentrum (CCSS)","Centre Commun de Sécurité Sociale (CCSS)"
"lu_2011_account_42172","42172","Foreign social security offices","asset_current","","False","Ausländische Sozialversicherungen","Organismes étrangers de sécurité sociale"
"lu_2011_account_42178","42178","Other social bodies","asset_current","","False","Sonstige Einrichtungen der sozialen Sicherheit","Autres organismes sociaux"
"lu_2011_account_421811","421811","Foreign VAT","asset_current","","False","Ausländische MwSt","TVA étrangères"
"lu_2011_account_421818","421818","Other foreign taxes","asset_current","","False","Sonstige ausländische Steuern","Autres impôts étrangers"
"lu_2020_account_42187","42187","Derivative financial instruments","asset_current","","False","Derivative Finanzinstrumente","Instruments financiers dérivés"
"lu_2011_account_42188","42188","Other miscellaneous receivables","asset_current","","False","Andere sonstige Forderungen","Autres créances diverses"
"lu_2011_account_42189","42189","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2020_account_4221","4221","Staff - advances and down payments","asset_current","","False","Personal - Vorschüsse und geleistete Anzahlungen","Personnel – Avances et acomptes"
"lu_2020_account_4222","4222","Amounts owed by partners and shareholders (others than from affiliated undertakings)","asset_current","","False","Forderungen gegen Gesellschafter und Aktionäre (andere als von verbundenen Unternehmen)","Créances sur associés ou actionnaires (autres qu'entreprises liées)"
"lu_2011_account_42231","42231","Investment subsidies","asset_current","","False","Investitionszuschüsse","Subventions d'investissement"
"lu_2011_account_42232","42232","Operating subsidies","asset_current","","False","Betriebszuschüsse","Subventions d'exploitation"
"lu_2011_account_42238","42238","Other subsidies","asset_current","","False","Sonstige Zuschüsse","Autres subventions"
"lu_2020_account_42287","42287","Derivative financial instruments","asset_current","","False","Derivative Finanzinstrumente","Instruments financiers dérivés"
"lu_2011_account_42288","42288","Other miscellaneous receivables","asset_current","","False","Andere sonstige Forderungen","Autres créances diverses"
"lu_2011_account_42289","42289","Value adjustments","asset_current","","False","Wertberichtigungen","Corrections de valeur"
"lu_2011_account_431","431","Down payments received within one year","liability_current","","False","Erhaltene Anzahlungen mit einer Restlaufzeit von bis zu einem Jahr","Acomptes reçus dont la durée résiduelle est inférieure ou égale à un an"
"lu_2011_account_432","432","Down payments received after more than one year","liability_current","","False","Erhaltene Anzahlungen mit einer Restlaufzeit von mehr als einem Jahr","Acomptes reçus dont la durée résiduelle est supérieure à un an"
"lu_2011_account_44111","44111","Suppliers","liability_payable","","True","Lieferanten","Fournisseurs"
"lu_2011_account_44112","44112","Suppliers - invoices not yet received","liability_current","","True","Lieferanten - Noch nicht erhaltene Rechnungen","Fournisseurs – Factures non parvenues"
"lu_2020_account_44113","44113","Suppliers with a debit balance","liability_current","","True","Lieferanten - Debitorische Kreditoren","Fournisseurs débiteurs"
"lu_2011_account_44121","44121","Suppliers","liability_current","","True","Lieferanten","Fournisseurs"
"lu_2020_account_44123","44123","Suppliers with a debit balance","liability_current","","True","Lieferanten - Debitorische Kreditoren","Fournisseurs débiteurs"
"lu_2011_account_4421","4421","Bills of exchange payable within one year","liability_current","","False","Durch Handelswechsel entstandene Verbindlichkeiten (Schuldwechsel) mit einer Restlaufzeit von bis zu einem Jahr","Effets à payer dont la durée résiduelle est inférieure ou égale à un an"
"lu_2011_account_4422","4422","Bills of exchange payable after more than one year","liability_current","","False","Durch Handelswechsel entstandene Verbindlichkeiten (Schuldwechsel) mit einer Restlaufzeit von mehr als einem Jahr","Effets à payer dont la durée résiduelle est supérieure à un an"
"lu_2011_account_45111","45111","Purchases and services","liability_current","","False","Käufe und Dienstleistungen","Achats et prestations de services"
"lu_2011_account_45112","45112","Loans and advances","liability_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_45118","45118","Other payables","liability_current","","False","Sonstige Verbindlichkeiten","Autres dettes"
"lu_2011_account_45121","45121","Purchases and services","liability_current","","False","Käufe und Dienstleistungen","Achats et prestations de services"
"lu_2011_account_45122","45122","Loans and advances","liability_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_45128","45128","Other payables","liability_current","","False","Sonstige Verbindlichkeiten","Autres dettes"
"lu_2011_account_45211","45211","Purchases and services","liability_current","","False","Käufe und Dienstleistungen","Achats et prestations de services"
"lu_2011_account_45212","45212","Loans and advances","liability_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_45218","45218","Other payables","liability_current","","False","Sonstige Verbindlichkeiten","Autres dettes"
"lu_2011_account_45221","45221","Purchases and services","liability_current","","False","Käufe und Dienstleistungen","Achats et prestations de services"
"lu_2011_account_45222","45222","Loans and advances","liability_current","","False","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"lu_2011_account_45228","45228","Other payables","liability_current","","False","Sonstige Verbindlichkeiten","Autres dettes"
"lu_2020_account_4611","4611","Municipal authorities","liability_current","","True","Gemeindeverwaltungen","Administrations communales"
"lu_2011_account_461211","461211","Corporate income tax - Tax accrual","liability_current","","False","Körperschaftssteuer - Ermittelte Steuerschuld","IRC – charge fiscale estimée"
"lu_2011_account_461212","461212","CIT - Tax payable","liability_current","","False","Körperschaftssteuer - Zu zahlende Steuerschuld","IRC – dette fiscale à payer"
"lu_2011_account_461221","461221","MBT - Tax accrual","liability_current","","False","Gewerbesteuer - Ermittelte Steuerschuld","ICC – charge fiscale estimée"
"lu_2011_account_461222","461222","MBT - Tax payable","liability_current","","False","Gewerbesteuer - Zu zahlende Steuerschuld","ICC – dette fiscale à payer"
"lu_2011_account_461231","461231","NWT - Tax accrual","liability_current","","False","Vermögensteuer - Ermittelte Steuerschuld","IF – charge fiscale estimée"
"lu_2011_account_461232","461232","NWT - Tax payable","liability_current","","False","Vermögensteuer - Zu zahlende Steuerschuld","IF – dette fiscale à payer"
"lu_2011_account_46124","46124","Withholding tax on wages and salaries","liability_current","","False","Einbehaltene Steuer auf Gehälter und Löhne","Retenue d'impôt sur traitements et salaires (RTS)"
"lu_2011_account_46125","46125","Withholding tax on financial investment income","liability_current","","False","Einbehaltene Kapitalertragsteuer","Retenue d'impôt sur revenus de capitaux mobiliers"
"lu_2011_account_46126","46126","Withholding tax on director's fees","liability_current","","False","Einbehaltene Steuer auf Tantiemen","Retenue d'impôt sur les tantièmes"
"lu_2011_account_46128","46128","ACD - Other amounts payable","liability_current","","False","ACD - Sonstige Verbindlichkeiten","ACD – Autres dettes"
"lu_2020_account_4613","4613","Customs and Excise Authority (ADA)","liability_current","","True","Zoll- und Verbrauchssteuerverwaltung (ADA)","Administration des Douanes et Accises (ADA)"
"lu_2020_account_461411","461411","VAT received","liability_current","","True","Umsatzsteuer","TVA en aval"
"lu_2011_account_461412","461412","VAT payable","liability_current","","False","Zu zahlende MwSt","TVA à payer"
"lu_2011_account_461413","461413","VAT down payments received","liability_current","","False","Erhaltene Anzahlungen auf MwSt","TVA acomptes reçus"
"lu_2011_account_461418","461418","VAT - Other payables","liability_current","","False","MwSt - Sonstige Verbindlichkeiten","TVA – Autres dettes"
"lu_2011_account_461421","461421","Registration duties","liability_current","","False","Registriergebühren","Droits d'enregistrement"
"lu_2011_account_461422","461422","Subscription tax","liability_current","","False","Abgeltungssteuer","Taxe d'abonnement"
"lu_2011_account_461428","461428","Other indirect taxes","liability_current","","False","Sonstige indirekte Steuern","Autres impôts indirects"
"lu_2020_account_46148","46148","AED - Other debts","liability_current","","False","Steuerverwaltung - Indirekte Steuern (AED) - Sonstige Verbindlichkeiten","AED - Autres dettes"
"lu_2020_account_46151","46151","Foreign VAT","liability_current","","False","Ausländische MwSt","TVA étrangères"
"lu_2020_account_46158","46158","Other foreign taxes","liability_current","","False","Sonstige ausländische Steuern","Autres impôts étrangers"
"lu_2011_account_4621","4621","Social Security office (CCSS)","liability_current","","False","Sozialversicherungszentrum (CCSS)","Centre Commun de Sécurité Sociale (CCSS)"
"lu_2011_account_4622","4622","Foreign Social Security offices","liability_current","","False","Ausländische Sozialversicherungen","Organismes étrangers de sécurité sociale"
"lu_2011_account_4628","4628","Other social bodies","liability_current","","False","Sonstige Einrichtungen der sozialen Sicherheit","Autres organismes sociaux"
"lu_2020_account_4711","4711","Received deposits and guarantees","liability_current","","True","Erhaltene Hinterlegungen und Kautionen","Dépôts et cautionnements reçus"
"lu_2020_account_4712","4712","Amounts payable to partners and shareholders (others than from affiliated undertakings)","liability_current","","True","Verbindlichkeiten gegenüber Gesellschaftern und Aktionären (andere als gegnüber verbundenen Unternehmen)","Dettes envers associés et actionnaires (autres qu'entreprises liées)"
"lu_2011_account_4713","4713","Amounts payable to directors, managers, statutory auditors and similar","liability_current","","False","Verbindlichkeiten gegenüber Verwaltern, Geschäftsführern, Kommissaren und ähnlichen","Dettes envers administrateurs, gérants, commissaires et organes assimilés"
"lu_2020_account_4714","4714","Amounts payable to staff","liability_current","","True","Verbindlichkeiten gegenüber dem Personal","Dettes envers le personnel"
"lu_2011_account_4715","4715","State - Greenhous gas and similar emission quotas to be returned or acquired","liability_current","","False","Staat - zurückzugebende oder zu erwerbende Kontigente für Treibhausgasemissionen und ähnliche","Etat – Quotas d'émission de gaz à effet de serre et assimilés à restituer ou à acquérir"
"lu_2020_account_47161","47161","Other loans","liability_current","","True","Sonstige Ausleihungen","Autres emprunts"
"lu_2020_account_47162","47162","Lease debts","liability_current","","True","Verbindlichkeiten aus Finanzierungsleasing","Dettes de leasing"
"lu_2020_account_47163","47163","Life annuities","liability_current","","True","Leibrenten","Rentes viagères"
"lu_2020_account_47164","47164","Other similar debts","liability_current","","True","Sonstige vergleichbare Verbindlichkeiten","Autres dettes assimilées"
"lu_2020_account_4717","4717","Derivative financial instruments","liability_current","","True","Derivative Finanzinstrumente","Instruments financiers dérivés"
"lu_2011_account_4718","4718","Other miscellaneous debts","liability_current","","False","Andere sonstige Verbindlichkeiten","Autres dettes diverses"
"lu_2020_account_4721","4721","Received deposits and guarantees","liability_current","","True","Erhaltene Hinterlegungen und Kautionen","Dépôts et cautionnements reçus"
"lu_2020_account_4722","4722","Amounts payable to partners and shareholders (others than from affiliated undertakings)","liability_current","","True","Verbindlichkeiten gegenüber Gesellschaftern und Aktionären (andere als gegnüber verbundenen Unternehmen)","Dettes envers associés et actionnaires (autres qu'entreprises liées)"
"lu_2011_account_4723","4723","Amounts payable to directors, managers, statutory auditors and similar","liability_current","","False","Verbindlichkeiten gegenüber Verwaltern, Geschäftsführern, Kommissaren und ähnlichen","Dettes envers administrateurs, gérants, commissaires et organes assimilés"
"lu_2020_account_4724","4724","Amounts payable to staff","liability_current","","True","Verbindlichkeiten gegenüber dem Personal","Dettes envers le personnel"
"lu_2020_account_47261","47261","Other loans","liability_current","","True","Sonstige Ausleihungen","Autres emprunts"
"lu_2020_account_47262","47262","Lease debts","liability_current","","True","Verbindlichkeiten aus Finanzierungsleasing","Dettes de leasing"
"lu_2020_account_47263","47263","Life annuities","liability_current","","True","Leibrenten","Rentes viagères"
"lu_2020_account_47264","47264","Other similar debts","liability_current","","True","Sonstige vergleichbare Verbindlichkeiten","Autres dettes assimilées"
"lu_2020_account_4727","4727","Derivative financial instruments","liability_current","","True","Derivative Finanzinstrumente","Instruments financiers dérivés"
"lu_2011_account_4728","4728","Other miscellaneous debts","liability_current","","False","Andere sonstige Verbindlichkeiten","Autres dettes diverses"
"lu_2011_account_481","481","Deferred charges (on one or more financial years)","asset_prepayments","","False","Abgegrenzte Aufwendungen (über ein oder mehrere Geschäfstjahre)","Charges à reporter (sur un ou plusieurs exercices)"
"lu_2011_account_482","482","Deferred income (on one or more financial years)","liability_current","","False","Abgegrenzte Erträge (über ein oder mehrere Geschäfstjahre)","Produits à reporter (sur un ou plusieurs exercices)"
"lu_2011_account_483","483","State - Greenhouse gas and similar emission quotas received","liability_current","","False","Staat - zugeteilte Kontigente für Treibhausgasemissionen und ähnliche","Etat - Quotas d'émission de gaz à effet de serre et assimilés alloués"
"lu_2011_account_484","484","Transitory or suspense accounts - Assets","asset_current","","False","Transitorische Konten - Aktiva","Comptes transitoires ou d'attente – Actif"
"lu_2011_account_485","485","Transitory or suspense accounts - Liabilities","liability_current","","False","Transitorische Konten - Passiva","Comptes transitoires ou d'attente – Passif"
"lu_2011_account_486","486","Linking accounts (branches) - Assets","asset_current","","False","Verbindungskonten (Niederlassungen) - Aktiva","Comptes de liaison (succursales) – Actif"
"lu_2011_account_487","487","Linking accounts (branches) - Liabilities","liability_current","","False","Verbindungskonten (Niederlassungen) - Passiva","Comptes de liaison (succursales) – Passif"
"lu_2011_account_501","501","Shares in affiliated undertakings","asset_current","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2011_account_502","502","Own shares or own corporate units","asset_current","","False","Eigene Aktien oder eigene Anteile","Actions propres ou parts propres"
"lu_2011_account_503","503","Shares in undertakings with which the undertaking is linked by virtue of participating interests","asset_current","","False","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_5081","5081","Shares - listed securities","asset_current","","False","Aktien - Notierte Wertpapiere","Actions – Titres cotés"
"lu_2011_account_5082","5082","Shares - unlisted securities","asset_current","","False","Aktien - Nicht notierte Wertpapiere","Actions – Titres non cotés"
"lu_2011_account_5083","5083","Debenture loans and other notes issued and repurchased by the company","asset_current","","False","Eigene Anleihen und sonstige eigene Schuldtitel","Obligations et autres titres de créance émis par l'entreprise et rachetés par elle"
"lu_2011_account_5084","5084","Listed debenture loans","asset_current","","False","Anleihen - Notierte Wertpapiere","Obligations – Titres cotés"
"lu_2011_account_5085","5085","Unlisted debenture loans","asset_current","","False","Anleihen - Nicht notierte Wertpapiere","Obligations – Titres non cotés"
"lu_2011_account_5088","5088","Other miscellaneous transferable securities","asset_current","","False","Andere sonstige Wertpapiere","Autres valeurs mobilières diverses"
"lu_2020_account_5131","5131","Banks and CCP : available balance","asset_cash","","False","Kreditinstitute und CCP : Guthaben","Banques et CCP : avoirs"
"lu_2020_account_5132","5132","Banks and CCP : overdraft","asset_cash","","False","Kreditinstitute und CCP : Überziehungen","Banques et CCP : découverts"
"lu_2020_account_516","516","Cash in hand","asset_cash","","False","Kassenbestand","Caisse"
"lu_2020_account_5171","5171","Internal transfers : debit balance","asset_cash","","False","Interne Transferkonten : Sollsaldo","Virements internes : solde débiteur"
"lu_2020_account_5172","5172","Internal transfers : credit balance","asset_cash","","False","Interne Transferkonten : Guthaben","Virements internes : solde créditeur"
"lu_2011_account_518","518","Other cash amounts","asset_current","","False","Sonstige Guthaben","Autres avoirs"
"lu_2020_account_601","601","Purchases of raw materials","expense","l10n_lu.account_tag_appendix_361","False","Einkäufe von Rohstoffen","Achats de matières premières"
"lu_2011_account_60311","60311","Solid fuels","expense","l10n_lu.account_tag_appendix_361","False","Feste Brennstoffe","Combustibles solides"
"lu_2011_account_60312","60312","Liquid fuels","expense","l10n_lu.account_tag_appendix_361","False","Flüssige Brennstoffe","Combustibles liquides"
"lu_2011_account_60313","60313","Gas","expense","l10n_lu.account_tag_appendix_289","False","Gas","Gaz"
"lu_2020_account_60314","60314","Water and sewage","expense","l10n_lu.account_tag_appendix_289","False","Wasser und Abwasser","Eau et eaux usées"
"lu_2020_account_60315","60315","Electricity","expense","l10n_lu.account_tag_appendix_289","False","Strom","Electricité"
"lu_2011_account_6032","6032","Maintenance supplies","expense","l10n_lu.account_tag_appendix_361","False","Pflegemittel","Produits d'entretien"
"lu_2011_account_6033","6033","Workshop, factory and store supplies and small equipment","expense","l10n_lu.account_tag_appendix_361","False","Werkstatt-, Fabrik- und Ladenausstattung","Fournitures et petit équipement d'atelier, d'usine et de magasin"
"lu_2011_account_6034","6034","Work clothes","expense","l10n_lu.account_tag_appendix_361","False","Berufsbekleidung","Vêtements professionnels"
"lu_2011_account_6035","6035","Office and administrative supplies","expense","l10n_lu.account_tag_appendix_332","False","Büroausstattung","Fournitures administratives et de bureau"
"lu_2011_account_6036","6036","Motor fuels","expense","l10n_lu.account_tag_appendix_190","False","Treibstoffe","Carburants"
"lu_2011_account_6037","6037","Lubricants","expense","l10n_lu.account_tag_appendix_361","False","Schmiermittel","Lubrifiants"
"lu_2011_account_6038","6038","Other consumable supplies","expense","l10n_lu.account_tag_appendix_361","False","Sonstige Betriebsstoffe","Autres fournitures consommables"
"lu_2020_account_604","604","Purchases of packaging","expense","l10n_lu.account_tag_appendix_349","False","Einkäufe von Verpackungen","Achats d'emballages"
"lu_2011_account_6061","6061","Purchases of merchandise","expense","l10n_lu.account_tag_appendix_361","False","Einkäufe von Waren","Achats de marchandises"
"lu_2011_account_6062","6062","Purchases of land for resale","expense","l10n_lu.account_tag_appendix_361","False","Einkäufe von zum Verkauf bestimmten Grundstücken","Achats de terrains destinés à la revente"
"lu_2011_account_6063","6063","Purchases of buildings for resale","expense","l10n_lu.account_tag_appendix_361","False","Einkäufe von zum Verkauf bestimmten Bauten/Gebäuden","Achats d'immeubles destinés à la revente"
"lu_2011_account_6071","6071","Changes in inventory of raw materials","expense","l10n_lu.account_tag_appendix_361","False","Bestandsveränderungen an Rohstoffen","Variation des stocks de matières premières"
"lu_2011_account_6073","6073","Changes in inventory of consumable materials and supplies","expense","l10n_lu.account_tag_appendix_361","False","Bestandsveränderungen an Hilfs- und Betriebsstoffen","Variation des stocks de matières et fournitures consommables"
"lu_2011_account_6074","6074","Changes in inventory of packaging","expense","l10n_lu.account_tag_appendix_361","False","Bestandsveränderungen an Verpackungen","Variation des stocks d'emballages"
"lu_2020_account_60761","60761","Merchandise","expense","l10n_lu.account_tag_appendix_361","False","Waren","Marchandises"
"lu_2020_account_60762","60762","Land for resale","expense","l10n_lu.account_tag_appendix_361","False","Zum Verkauf bestimme Grundstücke","Terrains destinés à la revente"
"lu_2020_account_60763","60763","Buildings for resale","expense","l10n_lu.account_tag_appendix_361","False","Zum Verkauf bestimmte Bauten/Gebäude","Immeubles destinés à la revente"
"lu_2020_account_60811","60811","Tailoring","expense","l10n_lu.account_tag_appendix_361","False","Lohnverarbeitung","Travail à façon"
"lu_2011_account_60812","60812","Research and development","expense","l10n_lu.account_tag_appendix_361","False","Forschung und Entwicklung","Recherche et développement"
"lu_2011_account_60813","60813","Architects' and engineers' fees","expense","l10n_lu.account_tag_appendix_361","False","Honorare für Architekten und Ingenieure","Frais d'architectes et d'ingénieurs"
"lu_2011_account_60814","60814","Outsourcing included in the production of goods and services","expense","l10n_lu.account_tag_appendix_361","False","Zulieferungen für Werklieferungen und -leistungen","Sous-traitance incorporée aux ouvrages et produits"
"lu_2020_account_6082","6082","Other purchases of material included in the production of goods and services","expense","l10n_lu.account_tag_appendix_361","False","Sonstige Einkäufe von Material, Ausstattungen, Ersatzteilen und Arbeiten für Werklieferungen und -leistungen","Autres achats de matériel incorporés aux ouvrages et produits"
"lu_2020_account_6083","6083","Purchase of greenhouse gas and similar emission quotas","expense","l10n_lu.account_tag_appendix_361","False","Einkäufe von Kontigenten für Treibhausgasemissionen und ähnliche","Achats de quotas d'émission de gaz à effet de serre et assimilés"
"lu_2020_account_6088","6088","Other purchases included in the production of goods and services","expense","l10n_lu.account_tag_appendix_361","False","Sonstige bezogene Gutachten und Dienstleistungen","Autres achats incorporés aux ouvrages et produits"
"lu_2011_account_6091","6091","RDR on purchases of raw materials","expense","l10n_lu.account_tag_appendix_361","False","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Rohstoffe","RRR sur achats de matières premières"
"lu_2011_account_6093","6093","RDR on purchases of consumable materials and supplies","expense","l10n_lu.account_tag_appendix_361","False","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Hilfs- und Betriebsstoffe","RRR sur achats de matières et fournitures consommables"
"lu_2011_account_6094","6094","RDR on purchases of packaging","expense","l10n_lu.account_tag_appendix_361","False","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Verpackungen","RRR sur achats d'emballages"
"lu_2011_account_6096","6096","RDR on purchases of merchandise and other goods for resale","expense","l10n_lu.account_tag_appendix_361","False","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Waren und sonstige zum Verkauf bestimmte Güter/Vermögensgegenstände","RRR sur achats de marchandises et d'autres biens destinés à la revente"
"lu_2011_account_6098","6098","RDR on purchases included in the production of goods and services","expense","l10n_lu.account_tag_appendix_361","False","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Einkäufe für Werklieferungen und -leistungen","RRR sur achats incorporés aux ouvrages et produits"
"lu_2011_account_6099","6099","Unallocated RDR","expense","l10n_lu.account_tag_appendix_361","False","Nicht zugeordnete Rabatte, Preisnachlässe und Rückvergütungen","RRR non affectés"
"lu_2011_account_61111","61111","Land","expense","l10n_lu.account_tag_appendix_361","False","Grundstücke","Terrains"
"lu_2011_account_61112","61112","Buildings","expense","l10n_lu.account_tag_appendix_307","False","Bauten/Gebäude","Constructions / Bâtiments"
"lu_2011_account_61123","61123","Rolling stock","expense","l10n_lu.account_tag_appendix_310","False","Fuhrpark","Matériel roulant"
"lu_2020_account_61128","61128","Other","expense","l10n_lu.account_tag_appendix_310","False","Sonstige","Autres"
"lu_2011_account_6113","6113","Service charges and co-ownership expenses","expense","l10n_lu.account_tag_appendix_361","False","Mietnebenkosten und Miteigentumsgemeinschaftskosten","Charges locatives et de copropriété"
"lu_2020_account_6114","6114","Financial leasing on real property","expense","l10n_lu.account_tag_appendix_361","False","Immobilienfinanzierungsleasing","Leasing financier immobilier"
"lu_2011_account_61153","61153","Rolling stock","expense","l10n_lu.account_tag_appendix_190","False","Fuhrpark","Matériel roulant"
"lu_2020_account_61158","61158","Other","expense","l10n_lu.account_tag_appendix_190","False","Sonstige","Autres"
"lu_2011_account_6121","6121","General subcontracting (not included in the production of goods and services)","expense","l10n_lu.account_tag_appendix_353","False","Allgemeine Zulieferung (nicht für Werklieferungen und -leistungen und sonstige Arbeiten)","Sous-traitance générale (non incorporée aux ouvrages et produits)"
"lu_2011_account_61221","61221","Buildings","expense","l10n_lu.account_tag_appendix_353","False","Bauten/Gebäude","Constructions / Bâtiments"
"lu_2011_account_61223","61223","Rolling stock","expense","l10n_lu.account_tag_appendix_351","False","Fuhrpark","Matériel roulant"
"lu_2020_account_61228","61228","Other","expense","l10n_lu.account_tag_appendix_361","False","Sonstige","Autres"
"lu_2020_account_6131","6131","Commissions and brokerage fees","expense","l10n_lu.account_tag_appendix_328","False","Provisionen und Maklergebühren","Commissions et courtages"
"lu_2011_account_6132","6132","IT services","expense","l10n_lu.account_tag_appendix_328","False","IT - Instandhaltung","Services informatiques"
"lu_2011_account_61332","61332","Loans' issuance expenses","expense","l10n_lu.account_tag_appendix_328","False","Emissionskosten von Anleihen","Frais sur émission d'emprunts"
"lu_2011_account_61333","61333","Bank account charges and bank commissions (included custody fees on securities)","expense","l10n_lu.account_tag_appendix_328","False","Kontogebühren und Bankkommissionen (Verwahrungsrechte von Wertpapieren inbegriffen)","Frais de comptes et commissions bancaires (y compris droits de garde sur titres)"
"lu_2011_account_61334","61334","Charges for electronic means of payment","expense","l10n_lu.account_tag_appendix_328","False","Gebühren auf elektronische Bezahlungsmittel","Frais sur moyens de paiements électroniques"
"lu_2011_account_61336","61336","Factoring services","expense","l10n_lu.account_tag_appendix_328","False","Factoringgebühren","Rémunérations d'affacturage"
"lu_2011_account_61338","61338","Other banking and similar services (except interest and similar expenses)","expense","l10n_lu.account_tag_appendix_328","False","Sonstige Bankdienstleistungen (außer Zinsen und vergleichbare Kosten)","Autres services bancaires et assimilés (hors intérêts et frais assimilés)"
"lu_2011_account_61341","61341","Legal, litigation and similar fees","expense","l10n_lu.account_tag_appendix_361","False","Rechts- und Prozesskosten und ähnliche","Honoraires juridiques, de contentieux et assimilés"
"lu_2011_account_61342","61342","Accounting, tax consulting, auditing and similar fees","expense","l10n_lu.account_tag_appendix_269","False","Buchführungs-, Steuerberatungs- und Prüfungskosten und ähnliche","Honoraires comptables, fiscaux, d'audit et assimilés"
"lu_2011_account_61348","61348","Other professional fees","expense","l10n_lu.account_tag_appendix_361","False","Sonstige Honorare","Autres honoraires"
"lu_2011_account_6135","6135","Notarial and similar fees","expense","l10n_lu.account_tag_appendix_361","False","Notarielle Beurkundungskosten und ähnliche","Frais d'actes notariés et assimilés"
"lu_2011_account_6138","6138","Other remuneration of intermediaries and professional fees","expense","l10n_lu.account_tag_appendix_361","False","Sonstige Vermittlervergütungen und Honorare","Autres rémunérations d'intermédiaires et honoraires"
"lu_2011_account_61411","61411","Buildings","expense","l10n_lu.account_tag_appendix_316","False","Bauten/Gebäude","Constructions / Bâtiments"
"lu_2011_account_61412","61412","Rolling stock","expense","l10n_lu.account_tag_appendix_190","False","Fuhrpark","Matériel roulant"
"lu_2011_account_61418","61418","Other","expense","l10n_lu.account_tag_appendix_316","False","Sonstige","Autres"
"lu_2011_account_6142","6142","Insurance on rented assets","expense","l10n_lu.account_tag_appendix_316","False","Versicherung für gemietete Vermögensgegenstände","Assurance sur biens pris en location"
"lu_2020_account_6143","6143","Transport insurance","expense","l10n_lu.account_tag_appendix_316","False","Transportversicherung","Assurance-transport"
"lu_2011_account_6144","6144","Business risk insurance","expense","l10n_lu.account_tag_appendix_316","False","Versicherung für betriebliche Risiken","Assurance risque d'exploitation"
"lu_2011_account_6145","6145","Customers credit insurance","expense","l10n_lu.account_tag_appendix_316","False","Forderungsausfallversicherung","Assurance insolvabilité clients"
"lu_2011_account_6146","6146","Third-party insurance","expense","l10n_lu.account_tag_appendix_331","False","Haftpflichtversicherung","Assurance responsabilité civile"
"lu_2011_account_6148","6148","Other insurances","expense","l10n_lu.account_tag_appendix_330","False","Sonstige Versicherungen","Autres assurances"
"lu_2011_account_61511","61511","Press advertising","expense","l10n_lu.account_tag_appendix_347","False","Annoncen und Inserate","Annonces et insertions"
"lu_2011_account_61512","61512","Samples","expense","l10n_lu.account_tag_appendix_361","False","Muster","Echantillons"
"lu_2011_account_61513","61513","Fairs and exhibitions","expense","l10n_lu.account_tag_appendix_361","False","Messen und Ausstellungen","Foires et expositions"
"lu_2011_account_61514","61514","Gifts to customers","expense","l10n_lu.account_tag_appendix_332","False","Geschenke für Kunden","Cadeaux à la clientèle"
"lu_2011_account_61515","61515","Catalogues, printed materials and publications","expense","l10n_lu.account_tag_appendix_337","False","Kataloge, Drucksachen und Veröffentlichungen","Catalogues et imprimés et publications"
"lu_2011_account_61516","61516","Donations","expense","l10n_lu.account_tag_appendix_361","False","Spenden","Dons courants"
"lu_2011_account_61517","61517","Sponsorship","expense","l10n_lu.account_tag_appendix_361","False","Sponsoring","Sponsoring"
"lu_2011_account_61518","61518","Other purchases of advertising services","expense","l10n_lu.account_tag_appendix_361","False","Sonstige Werbung","Autres achats de services publicitaires"
"lu_2011_account_615211","615211","Management (respectively owner and partner)","expense","l10n_lu.account_tag_appendix_283","False","Geschäftsleitung (Einzelunternehmer bzw. Gesellschafter)","Direction (respectivement exploitant et associés)"
"lu_2011_account_615212","615212","Staff","expense","l10n_lu.account_tag_appendix_260","False","Personal","Personnel"
"lu_2011_account_61522","61522","Relocation expenses","expense","l10n_lu.account_tag_appendix_361","False","Umzugskosten des Unternehmens","Frais de déménagement de l'entreprise"
"lu_2011_account_61523","61523","Business assignments","expense","l10n_lu.account_tag_appendix_361","False","Dienstreisen","Missions"
"lu_2011_account_61524","61524","Receptions and entertainment costs","expense","l10n_lu.account_tag_appendix_283","False","Bewirtungs- und Repräsentationskosten","Réceptions et frais de représentation"
"lu_2011_account_61531","61531","Postal charges","expense","l10n_lu.account_tag_appendix_332","False","Postgebühren","Frais postaux"
"lu_2011_account_61532","61532","Telecommunication costs","expense","l10n_lu.account_tag_appendix_301","False","Sonstige Telekommunikationskosten","Frais de télécommunication"
"lu_2011_account_6161","6161","Transportation of purchased goods","expense","l10n_lu.account_tag_appendix_343","False","Transporte von Einkäufen","Transports sur achats"
"lu_2011_account_6162","6162","Transportation of sold goods","expense","l10n_lu.account_tag_appendix_343","False","Transporte von Verkäufen","Transports sur ventes"
"lu_2011_account_6165","6165","Collective staff transportation","expense","l10n_lu.account_tag_appendix_343","False","Beförderungen von Personal","Transports collectifs du personnel"
"lu_2011_account_6168","6168","Other transportation","expense","l10n_lu.account_tag_appendix_343","False","Sonstige Transporte","Autres transports"
"lu_2011_account_6171","6171","Temporary staff","expense","l10n_lu.account_tag_appendix_188","False","Aushilfskräfte","Personnel intérimaire"
"lu_2011_account_6172","6172","External staff on secondment","expense","l10n_lu.account_tag_appendix_188","False","Ausgeliehenes Personal","Personnel prêté à l'entreprise"
"lu_2020_account_6181","6181","Documentation","expense","l10n_lu.account_tag_appendix_361","False","Dokumentation",""
"lu_2011_account_6182","6182","Costs of training, symposiums, seminars, conferences","expense","l10n_lu.account_tag_appendix_361","False","Kosten von Weiterbildungen, Kolloquien, Seminaren, Konferenzen","Frais de formation, colloques, séminaires, conférences"
"lu_2011_account_6183","6183","Industrial and non-industrial waste treatment","expense","l10n_lu.account_tag_appendix_361","False","Abfallbeseitigung von Industrieabfällen und sonstigen Abfällen","Elimination des déchets industriels et non industriels"
"lu_2020_account_61841","61841","Solid fuels","expense","l10n_lu.account_tag_appendix_361","False","Feste Brennstoffe","Combustibles solides"
"lu_2020_account_61842","61842","Liquid fuels (oil, motor fuel, etc.)","expense","l10n_lu.account_tag_appendix_361","False","Flüssige Brennsoffe","Combustibles liquides (mazout, carburants, etc.)"
"lu_2020_account_61843","61843","Gas","expense","l10n_lu.account_tag_appendix_361","False","Gas","Gaz"
"lu_2020_account_61844","61844","Water and waste water","expense","l10n_lu.account_tag_appendix_361","False","Wasserversorgung und Abwasserbeseitigung","Eau et eaux usées"
"lu_2020_account_61845","61845","Electricity","expense","l10n_lu.account_tag_appendix_361","False","Strom","Electricité"
"lu_2020_account_61851","61851","Office supplies","expense","l10n_lu.account_tag_appendix_361","False","Büromaterial","Fournitures administratives et de bureau"
"lu_2020_account_61852","61852","Small equipment","expense","l10n_lu.account_tag_appendix_361","False","Kleines Werkzeug","Petit équipement"
"lu_2020_account_61853","61853","Work clothes","expense","l10n_lu.account_tag_appendix_361","False","Berufsbekleidung","Vêtements professionnels"
"lu_2020_account_61854","61854","Maintenance supplies","expense","l10n_lu.account_tag_appendix_361","False","Pflegemittel","Produits d'entretien"
"lu_2020_account_61858","61858","Other","expense","l10n_lu.account_tag_appendix_361","False","Sonstige","Autres"
"lu_2011_account_6186","6186","Surveillance and security charges","expense","l10n_lu.account_tag_appendix_361","False","Wach- und Sicherheitsdienste","Frais de surveillance et de gardiennage"
"lu_2011_account_6187","6187","Contributions to professional associations","expense","l10n_lu.account_tag_appendix_336","False","Beiträge für Berufsverbände","Cotisations aux associations professionnelles"
"lu_2011_account_6188","6188","Other miscellaneous external charges","expense","l10n_lu.account_tag_appendix_361","False","Andere sonstige externe Aufwendungen","Autres charges externes diverses"
"lu_2011_account_619","619","Rebates, discounts and refunds received on other external charges","expense","l10n_lu.account_tag_appendix_239","False","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf sonstige externe Aufwendungen","Rabais, remises et ristournes (RRR) obtenus et non directement déduits des autres charges externes"
"lu_2011_account_62111","62111","Base wages","expense","l10n_lu.account_tag_appendix_239","False","Grundlöhne","Salaires de base"
"lu_2011_account_621121","621121","Sunday","expense","l10n_lu.account_tag_appendix_239","False","Sonntagsarbeit","Dimanche"
"lu_2011_account_621122","621122","Public holidays","expense","l10n_lu.account_tag_appendix_239","False","Feiertagsarbeit","Jours fériés légaux"
"lu_2011_account_621123","621123","Overtime","expense","l10n_lu.account_tag_appendix_239","False","Überstunden","Heures supplémentaires"
"lu_2011_account_621128","621128","Other supplements","expense","l10n_lu.account_tag_appendix_239","False","Sonstige Zuschläge","Autres suppléments"
"lu_2011_account_62114","62114","Incentives, bonuses and commissions","expense","l10n_lu.account_tag_appendix_239","False","Gratifikationen, Prämien und Provisionen","Gratifications, primes et commissions"
"lu_2011_account_62115","62115","Benefits in kind","expense","l10n_lu.account_tag_appendix_239","False","Sachleistungen","Avantages en nature"
"lu_2011_account_62116","62116","Severance pay","expense","l10n_lu.account_tag_appendix_239","False","Abfindungen","Indemnités de licenciement"
"lu_2011_account_62117","62117","Survivor's pay","expense","l10n_lu.account_tag_appendix_239","False","Hinterbliebenenzuschuss","Trimestre de faveur"
"lu_2011_account_6218","6218","Other benefits","expense","l10n_lu.account_tag_appendix_239","False","Sonstige Zulagen","Autres avantages"
"lu_2020_account_6219","6219","Refunds on wages paid","expense","l10n_lu.account_tag_appendix_244","False","Erstattungen von gezahlten Löhnen","Remboursements sur salaires"
"lu_2011_account_6221","6221","Students","expense","l10n_lu.account_tag_appendix_247","False","Studentische Aushilfskräfte","Etudiants"
"lu_2011_account_6222","6222","Casual workers","expense","l10n_lu.account_tag_appendix_247","False","Gelegenheitsarbeitskräfte","Salaires occasionnels"
"lu_2011_account_6228","6228","Other","expense","l10n_lu.account_tag_appendix_247","False","Sonstige","Autres"
"lu_2020_account_6231","6231","Social security on pensions","expense","l10n_lu.account_tag_appendix_250","False","Soziale Aufwendungen für Renten","Charges sociales couvrant les pensions"
"lu_2011_account_6232","6232","Other social security costs (including illness, accidents, a.s.o.)","expense","l10n_lu.account_tag_appendix_253","False","Sonstige soziale Aufwendungen (einschließlich für Krankheits- und Arbeitsunfallversicherung)","Autres charges sociales (y inclus maladie, accident, etc.)"
"lu_2020_account_62411","62411","Premiums for external pensions funds","expense","l10n_lu.account_tag_appendix_361","False","Beiträge für externe Rentenfonds","Primes à des fonds de pensions extérieurs"
"lu_2020_account_62412","62412","Changes to provisions for complementary pensions","expense","l10n_lu.account_tag_appendix_361","False","Zuführung zu Rückstellungen für Zusatzrenten","Variations sur provisions pour pensions complémentaires"
"lu_2020_account_62413","62413","Withholding tax on complementary pensions","expense","l10n_lu.account_tag_appendix_361","False","Einbehaltene Steuer auf Zusatzrenten","Retenue d'impôt sur pension complémentaire"
"lu_2020_account_62414","62414","Insolvency insurance premiums","expense","l10n_lu.account_tag_appendix_361","False","Beiträge zur Insolvenzversicherung","Prime d'assurance insolvabilité"
"lu_2020_account_62415","62415","Complementary pensions paid by the employer","expense","l10n_lu.account_tag_appendix_361","False","Ausgezahlte betriebliche Zusatzrente","Pensions complémentaires versées par l'employeur"
"lu_2020_account_6248","6248","Other staff expenses not mentioned above","expense","l10n_lu.account_tag_appendix_361","False","Sonstige nicht oben aufgeführte Personalaufwendungen","Autres frais de personnel non visés ci-dessus"
"lu_2011_account_6311","6311","AVA on set-up and start-up costs","expense","","False","ZWb von Gründungs- und Errichtungskosten (Rechts- und Beratungskosten)","DCV sur frais de constitution et de premier établissement"
"lu_2011_account_6313","6313","AVA on expenses for capital increases and various operations (mergers, demergers, changes of legal form)","expense","","False","ZWb von Kosten für Kapitalerhöhung und anderen Vorgängen (Verschmelzungen, Spaltungen und Umwandlungen)","DCV sur frais d'augmentation de capital et d'opérations diverses (fusions, scissions, transformations)"
"lu_2011_account_6314","6314","AVA on loan-issuance expenses","expense","","False","ZWb von Emissionskosten von Anleihen","DCV sur frais d'émission d'emprunts"
"lu_2011_account_6318","6318","AVA on other similar expenses","expense","","False","ZWb von sonstigen vergleichbaren Kosten","DCV sur autres frais assimilés"
"lu_2011_account_6321","6321","AVA on development costs","expense","","False","ZWb von Entwicklungskosten","DCV sur frais de développement"
"lu_2011_account_6322","6322","AVA on concessions, patents, licences, trademarks and similar rights and assets","expense","","False","ZWb von Konzessionen, Patenten, Lizenzen, Warenzeichen und vergleichbaren Rechten und Werten","DCV sur concessions, brevets, licences, marques ainsi que droits et valeurs similaires"
"lu_2011_account_6323","6323","AVA on goodwill acquired for consideration","expense","","False","ZWb vom Geschäfts- oder Firmenwert, soweit er entgeltlich erworben wurde","DCV sur fonds de commerce dans la mesure où il a été acquis à titre onéreux"
"lu_2011_account_6324","6324","AVA on down payments and intangible fixed assets under development","expense","","False","ZWb von geleisteten Anzahlungen und immateriellen Vermögensgegenständen in Entwicklung","DCV sur acomptes versés et immobilisations incorporelles en cours"
"lu_2011_account_63311","63311","AVA on land","expense","","False","ZWb von Grundstücken","DCV sur terrains"
"lu_2011_account_63312","63312","AVA on fixtures and fittings-out of land","expense","","False","ZWb von Erschließungen von Grundstücken","DCV sur agencements et aménagements de terrains"
"lu_2011_account_63313","63313","AVA on buildings","expense","","False","ZWb von Bauten / Gebäuden","DCV sur constructions / bâtiments"
"lu_2020_account_63314","63314","AVA on fixtures and fittings-out of buildings","expense","","False","ZWb von Einrichtungen von Bauten/Gebäuden","DCV sur agencements et aménagements de constructions / bâtiments"
"lu_2020_account_63315","63315","FVA on investment properties","expense","","False","AFV von Anlageimmobilien","AJV sur immeubles de placement"
"lu_2011_account_6332","6332","AVA on plant and machinery","expense","","False","ZWb von technischen Anlagen und Maschinen","DCV sur installations techniques et machines"
"lu_2011_account_6333","6333","AVA on other fixtures and fittings, tools and equipment (including rolling stock)","expense","","False","ZWb von anderen Anlagen, Betriebs- und Geschäftsausstattung und Fuhrpark","DCV sur autres installations, outillage et mobilier (y compris matériel roulant)"
"lu_2011_account_6334","6334","AVA on down payments and tangible fixed assets under development","expense","","False","ZWb von geleisteten Anzahlungen und Anlagen im Bau","DCV sur acomptes versés et immobilisations corporelles en cours"
"lu_2011_account_6341","6341","AVA on inventories of raw materials and consumables","expense","","False","ZWb von Roh-, Hilfs- und Betriebsstoffen","DCV sur stocks de matières premières et consommables"
"lu_2011_account_6342","6342","AVA on inventories of work and contracts in progress","expense","","False","ZWb von unfertigen Erzeugnissen und in Arbeit befindlichen Aufträgen","DCV sur stocks de produits en cours de fabrication et commandes en cours"
"lu_2011_account_6343","6343","AVA on inventories of goods","expense","","False","ZWb von Erzeugnissen","DCV sur stocks de produits"
"lu_2011_account_6344","6344","AVA on inventories of merchandise and other goods for resale","expense","","False","ZWb von Waren und zum Verkauf bestimmten Gütern/Vermögensgegenständen","DCV sur stocks de marchandises et d'autres biens destinés à la revente"
"lu_2011_account_6345","6345","AVA on down payments on inventories","expense","","False","ZWb von geleisteten Anzahlungen auf Vorräte","DCV sur acomptes versés sur stocks"
"lu_2011_account_6351","6351","AVA on trade receivables","expense","","False","ZWb von Forderungen aus Lieferungen und Leistungen","DCV sur créances résultant de ventes et prestations de services"
"lu_2011_account_6352","6352","AVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","ZWb von Forderungen gegen verbundene Unternehmen und Unternehmen, mit denen ein Beteiligungsverhältnis besteht","DCV sur créances sur des entreprises liées et sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_6353","6353","AVA on other receivables","expense","","False","ZWb von sonstigen Forderungen","DCV sur autres créances"
"lu_2020_account_6354","6354","FVA on receivables from current assets","expense","","False","AFV von Forderungen des Umlaufvermögens","AJV sur créances de l'actif circulant"
"lu_2011_account_6411","6411","Concessions","expense","","False","Konzessionen",""
"lu_2011_account_6412","6412","Patents","expense","","False","Patente","Brevets"
"lu_2011_account_6413","6413","Software licences","expense","","False","Software- und Softwarepaketlizenzen","Licences informatiques"
"lu_2011_account_6414","6414","Trademarks and franchise","expense","","False","Warenzeichen und Franchising (Verkaufskonzession/Alleinverkaufsrecht)","Marques et franchises"
"lu_2011_account_64151","64151","Copyrights and reproduction rights","expense","","False","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"lu_2011_account_64158","64158","Other similar rights and assets","expense","","False","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"lu_2011_account_642","642","Indemnities, damages and interest","expense","","False","Entschädigungen, Schadensersatz und Zinsen","Indemnités, dommages et intérêts"
"lu_2020_account_6431","6431","Attendance fees","expense","","False","Sitzungsgeld","Jetons de présence"
"lu_2020_account_6432","6432","Director's fees","expense","","False","Tantiemen","Tantièmes"
"lu_2020_account_6438","6438","Other similar remuneration","expense","","False","Sonstige ähnliche Vergütungen","Autres rémunérations assimilées"
"lu_2020_account_64411","64411","Book value of intangible fixed assets disposed of","expense","","False","Buchwert der verkauften immateriellen Anlagewerte","Valeur comptable d'immobilisations incorporelles cédées"
"lu_2020_account_64412","64412","Disposal proceeds of intangible fixed assets","expense","","False","Verkaufserlös von immateriellen Anlagewerten","Produits de cession d'immobilisations incorporelles"
"lu_2020_account_64421","64421","Book value of tangible fixed assets disposed of","expense","","False","Buchwert der verkauften materiellen Anlagewerte","Valeur comptable d'immobilisations corporelles cédées"
"lu_2020_account_64422","64422","Disposal proceeds of tangible fixed assets","expense","","False","Verkaufserlös von materiellen Anlagewerten","Produits de cession d'immobilisations corporelles"
"lu_2011_account_6451","6451","Trade receivables","expense","","False","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"lu_2011_account_6452","6452","Amounts owed by affiliated undertakings","expense","","False","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"lu_2011_account_6453","6453","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_6454","6454","Other receivables","expense","","False","Sonstige Forderungen","Autres créances"
"lu_2011_account_6461","6461","Real property tax","expense","","False","Grundsteuer","Impôt foncier"
"lu_2011_account_6462","6462","Non-refundable VAT","expense","","False","Nicht erstattungsfähige Mehrwertsteuer (MwSt)","TVA non récupérable"
"lu_2020_account_6463","6463","Duties on imported merchandise","expense","","False","Zölle und Einfuhrabgaben","Droits sur les marchandises en provenance de l'étranger"
"lu_2011_account_6464","6464","Excise duties on production and tax on consumption","expense","","False","Verbrauchsabgaben aus der Produktion und Verbrauchssteuer","Droits d'accises à la production et taxe de consommation"
"lu_2011_account_64651","64651","Registration fees","expense","","False","Eintragungsgebühren","Droits d'enregistrement"
"lu_2011_account_64658","64658","Other registration fees, stamp duties and mortgage duties","expense","","False","Sonstige Eintragungs- und Stempelgebühren und Hypothekensteuern","Autres droits d'enregistrement et de timbre, droits d'hypothèques"
"lu_2011_account_6466","6466","Motor-vehicle taxes","expense","l10n_lu.account_tag_appendix_190","False","Kfz-Steuern","Taxes sur les véhicules"
"lu_2011_account_6467","6467","Bar licence tax","expense","l10n_lu.account_tag_appendix_358","False","Getränkesteuer","Taxe de cabaretage"
"lu_2011_account_6468","6468","Other duties and taxes","expense","l10n_lu.account_tag_appendix_358","False","Sonstige Gebühren und Steuern","Autres droits et taxes"
"lu_2011_account_647","647","Allocations to tax-exempt capital gains","expense","","False","Zuführungen zu Sonderposten mit Rücklageanteil","Dotations aux plus-values immunisées"
"lu_2020_account_6481","6481","Fines, sanctions and penalties","expense","","False","Steuer- und strafrechtliche Bußgelder","Amendes, sanctions et pénalités"
"lu_2020_account_6488","6488","Miscellaneous operating charges","expense","","False","Sonstige betriebliche Aufwendungen","Charges d'exploitation diverses"
"lu_2020_account_6491","6491","Allocations to tax provisions","expense","","False","Zuführungen zu Steuerrückstellungen","Dotations aux provisions pour impôts"
"lu_2020_account_6492","6492","Allocations to operating provisions","expense","","False","Zuführungen zu betrieblichen Rückstellungen","Dotations aux provisions d'exploitation"
"lu_2011_account_65111","65111","AVA on shares in affiliated undertakings","expense","","False","ZWb von Anteilen an verbundenen Unternehmen","DCV sur parts dans des entreprises liées"
"lu_2011_account_65112","65112","AVA on amounts owed by affiliated undertakings","expense","","False","ZWb von Forderungen gegen verbundene Unternehmen","DCV sur créances sur des entreprises liées"
"lu_2011_account_65113","65113","AVA on participating interests","expense","","False","ZWb von Anteilen","DCV sur participations"
"lu_2011_account_65114","65114","AVA on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","ZWb von Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","DCV sur créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_65115","65115","AVA on securities held as fixed assets","expense","","False","ZWb von Wertpapieren des Anlagevermögens","DCV sur titres ayant le caractère d'immobilisations"
"lu_2011_account_65116","65116","AVA on loans, deposits and claims held as fixed assets","expense","","False","ZWb von Ausleihungen, geleisteten Hinterlegungen und Forderungen des Anlagevermögens","DCV sur prêts, dépôts et créances immobilisés"
"lu_2011_account_6512","6512","FVA on financial fixed assets","expense","","False","AFV der Finanzanlagen","AJV sur immobilisations financières"
"lu_2020_account_65211","65211","Shares in affiliated undertakings","expense","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2020_account_65212","65212","Amounts owed by affiliated undertakings","expense","","False","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"lu_2020_account_65213","65213","Participating interests","expense","","False","Anteile","Participations"
"lu_2020_account_65214","65214","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_65215","65215","Securities held as fixed assets","expense","","False","Wertpapiere des Anlagevermögens","Titres ayant le caractère d'immobilisations"
"lu_2020_account_65216","65216","Loans, deposits and claims held as fixed assets","expense","","False","Ausleihungen, geleistete Hinterlegungen und Forderungen (Anlagevermögen)","Prêts, dépôts et créances immobilisés"
"lu_2020_account_652211","652211","Book value of shares in affiliated undertakings disposed of","expense","","False","Buchwert der verkauften Beteiligungen an verbundenen Unternehmen","Valeur comptable de parts cédées dans des entreprises liées"
"lu_2020_account_652212","652212","Disposal proceeds of shares in affiliated undertakings","expense","","False","Veräußerungswert der Anteile an verbundenen Unternehmen","Produits de cession de parts dans des entreprises liées"
"lu_2020_account_652221","652221","Book value of amounts owed by affiliated undertakings disposed of","expense","","False","Buchwert der verkauften Forderungen gegen verbundene Unternehmen","Valeur comptable de créances cédées sur des entreprises liées"
"lu_2020_account_652222","652222","Disposal proceeds of amounts owed by affiliated undertakings","expense","","False","Veräußerungswert der Forderungen gegen verbundene Unternehmen","Produits de cession de créances sur des entreprises liées"
"lu_2020_account_652231","652231","Book value of participating interests disposed of","expense","","False","Buchwert der verkauften Anteile","Valeur comptable de participations cédées"
"lu_2020_account_652232","652232","Disposal proceeds of participating interests","expense","","False","Veräußerungswert der Anteile","Produits de cession de participations"
"lu_2020_account_652241","652241","Book value of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests disposed of","expense","","False","Buchwert der verkauften Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Valeur comptable de créances cédées sur participations"
"lu_2020_account_652242","652242","Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","Veräußerungswert der Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Produits de cession de créances sur participations"
"lu_2020_account_652251","652251","Book value of securities held as fixed assets disposed of","expense","","False","Buchwert der verkauften Wertpapiere des Anlagevermögens","Valeur comptable de titres cédés ayant le caractère d'immobilisations"
"lu_2020_account_652252","652252","Disposal proceeds of securities held as fixed assets","expense","","False","Veräußerungswert der Wertpapiere des Anlagevermögens","Produits de cession de titres ayant le caractère d'immobilisations"
"lu_2020_account_652261","652261","Book value of yielded loans, deposits and claims held as fixed assets","expense","","False","Buchwert der verkauften Ausleihungen, geleistete Hinterlegungen und Forderungen des Umlaufvermögens","Valeur comptable de prêts, dépôts et créances immobilisés cédés"
"lu_2020_account_652262","652262","Disposal proceeds of loans, deposits and claims held as fixed assets","expense","","False","Veräußerungswert der Ausleihungen, geleistete Hinterlegungen und Forderungen des Umlaufvermögens","Produits de cession de prêts, dépôts et créances immobilisés"
"lu_2011_account_65311","65311","AVA on shares in affiliated undertakings","expense","","False","ZWb von Anteilen an verbundenen Unternehmen","DCV sur parts dans des entreprises liées"
"lu_2011_account_65312","65312","AVA on own shares or own corporate units","expense","","False","ZWb von eigenen Aktien oder Anteilen","DCV sur actions propres ou parts propres"
"lu_2011_account_65313","65313","AVA on shares in undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","ZWb von Anteilen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","DCV sur parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_65318","65318","AVA on other transferable securities","expense","","False","ZWb von sonstigen Wertpapieren","DCV sur autres valeurs mobilières"
"lu_2011_account_6532","6532","FVA on transferable securities","expense","","False","AFV der Wertpapiere","AJV sur valeurs mobilières"
"lu_2020_account_65411","65411","From affiliated undertakings","expense","","False","Forderungen gegen verbundene Unternehmen","Sur des entreprises liées"
"lu_2020_account_65412","65412","From undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_65413","65413","From other receivables from current assets","expense","","False","Sonstige Forderungen","Sur autres créances de l'actif circulant"
"lu_2020_account_65421","65421","Shares in affiliated undertakings","expense","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2020_account_65422","65422","Own shares or corporate units","expense","","False","Eigene Aktien oder Anteile","Actions propres ou parts propres"
"lu_2020_account_65423","65423","Shares in undertakings with which the undertaking is linked by virtue of participating interests","expense","","False","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_65428","65428","Other transferable securities","expense","","False","Sonstige Wertpapiere","Autres valeurs mobilières"
"lu_2011_account_65511","65511","Interest on debenture loans - affiliated undertakings","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf Anleihen - verbundene Unternehmen","Intérêts sur emprunts obligataires - entreprises liées"
"lu_2011_account_65512","65512","Interest on debenture loans - other","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf Anleihen - sonstige","Intérêts sur emprunts obligataires - autres"
"lu_2011_account_65521","65521","Banking interest on current accounts","expense","l10n_lu.account_tag_appendix_327","False","Kontozinsen","Intérêts sur comptes bancaires"
"lu_2011_account_65522","65522","Banking interest on financing operations","expense","l10n_lu.account_tag_appendix_326","False","Bankzinsen auf Finanzoperationen","Intérêts bancaires sur opérations de financement"
"lu_2020_account_655231","655231","Interest on financial leases - affiliated undertakings","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf Finanzierungsleasings - verbundene Unternehmen","Intérêts sur leasings financiers - entreprises liées"
"lu_2020_account_655232","655232","Interest on financial leases - other","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf Finanzierungsleasings - sonstige","Intérêts sur leasings financiers - autres"
"lu_2011_account_6553","6553","Interest on trade payables","expense","l10n_lu.account_tag_appendix_327","False","Zinsen auf Verbindlichkeiten aus Lieferungen und Leistungen","Intérêts sur dettes commerciales"
"lu_2020_account_65541","65541","Interest payable to affiliated undertakings","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf Verbindlichkeiten gegenüber verbundenen Unternehmen","Intérêts sur des entreprises liées"
"lu_2020_account_65542","65542","Interest payable to undertakings with which the undertaking is linked by virtue of participating interests","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf Verbindlichkeiten gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Intérêts sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_65551","65551","Discounts and charges on bills of exchange - affiliated undertakings","expense","l10n_lu.account_tag_appendix_328","False","Diskonte und Kosten von Handelswechsel - verbundene Unternehmen","Escomptes et frais sur effets de commerce - entreprises liées"
"lu_2020_account_65552","65552","Discounts and charges on bills of exchange - other","expense","l10n_lu.account_tag_appendix_328","False","Diskonte und Kosten von Handelswechsel - sonstige","Escomptes et frais sur effets de commerce - autres"
"lu_2020_account_65561","65561","Granted discounts - affiliated undertakings","expense","l10n_lu.account_tag_appendix_328","False","Gewährte Diskonten - verbundene Unternehmen","Escomptes accordés - entreprises liées"
"lu_2020_account_65562","65562","Granted discounts - other","expense","l10n_lu.account_tag_appendix_328","False","Gewährte Diskonten - sonstige","Escomptes accordés - autres"
"lu_2011_account_657","657","Share in the losses of undertakings accounted for under the equity method","expense","l10n_lu.account_tag_appendix_328","False","Verlustanteile in den gemeinsamen Unternehmen (andere als Kapitalgesellschaften)","Quote-part dans la perte des entreprises mises en équivalence"
"lu_2020_account_65581","65581","Interest payable on other loans and debts - affiliated undertakings","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf sonstige Ausleihungen und Verbindlichkeiten - verbundene Unternehmen","Intérêts sur autres emprunts et dettes - entreprises liées"
"lu_2020_account_65582","65582","Interest payable on other loans and debts - other","expense","l10n_lu.account_tag_appendix_326","False","Zinsen auf sonstige Ausleihungen und Verbindlichkeiten - sonstige","Intérêts sur autres emprunts et dettes - autres"
"lu_2020_account_6561","6561","Foreign currency exchange losses - affiliated undertakings","expense","","False","Wechselkursverluste - verbundene Unternehmen","Pertes de change - entreprises liées"
"lu_2020_account_6562","6562","Foreign currency exchange losses - other","expense","","False","Wechselkursverluste - sonstige","Pertes de change - autres"
"lu_2020_account_6581","6581","Other financial charges - affiliated undertakings","expense","l10n_lu.account_tag_appendix_328","False","Sonstige finanzielle Aufwendungen - verbundene Unternehmen","Autres charges financières - entreprises liées"
"lu_2020_account_6582","6582","Other financial charges - other","expense","l10n_lu.account_tag_appendix_328","False","Sonstige finanzielle Aufwendungen - sonstige","Autres charges financières - autres"
"lu_2020_account_6591","6591","Allocations to financial provisions - affiliated undertakings","expense","l10n_lu.account_tag_appendix_328","False","Zuführungen zu finanziellen Rückstellungen - verbundene Unternehmen","Dotations aux provisions financières - entreprises liées"
"lu_2020_account_6592","6592","Allocations to financial provisions - other","expense","l10n_lu.account_tag_appendix_328","False","Zuführungen zu finanziellen Rückstellungen - sonstige","Dotations aux provisions financières - autres"
"lu_2011_account_6711","6711","CIT - current financial year","expense","","False","KSt - laufendes Geschäftsjahr","IRC - exercice courant"
"lu_2011_account_6712","6712","CIT - previous financial years","expense","","False","KSt - vorhergehende Geschäftsjahre","IRC - exercices antérieurs"
"lu_2011_account_6721","6721","MBT - current financial year","expense","","False","GewSt - laufendes Geschäftsjahr","ICC - exercice courant"
"lu_2011_account_6722","6722","MBT - previous financial years","expense","","False","GewSt - vorhergehende Geschäftsjahre","ICC - exercices antérieurs"
"lu_2011_account_6731","6731","Withholding taxes","expense","","False","Quellensteuern","Retenues d'impôt à la source"
"lu_2011_account_67321","67321","Current financial year","expense","","False","Laufendes Geschäftsjahr","Exercice courant"
"lu_2011_account_67322","67322","Previous financial years","expense","","False","Vorhergehende Geschäftsjahre","Exercices antérieurs"
"lu_2011_account_6733","6733","Taxes levied on non-resident undertakings","expense","","False","Steuern, die durch die nicht gebietsansässige Unternehmen getragen wurden","Impôts supportés par les entreprises non résidentes"
"lu_2011_account_6738","6738","Other foreign income taxes","expense","","False","Sonstige ausländische Steuern auf Einkommen und Erträge","Autres impôts étrangers sur le résultat"
"lu_2020_account_679","679","Allocations to provisions for deferred taxes","expense","","False","Zuführungen zu Rückstellungen für Steuern","Dotations aux provisions pour impôts différés"
"lu_2011_account_6811","6811","NWT - current financial year","expense","","False","VermSt - laufendes Geschäftsjahr","IF - exercice courant"
"lu_2011_account_6812","6812","NWT - previous financial years","expense","","False","VermSt - vorhergehende Geschäftsjahre","IF - exercices antérieurs"
"lu_2011_account_682","682","Subscription tax","expense","","False","Abgeltungssteuer","Taxe d'abonnement"
"lu_2011_account_683","683","Foreign taxes","expense","","False","Ausländische Steuern","Impôts étrangers"
"lu_2011_account_688","688","Other taxes","expense","","False","Sonstige Steuern","Autres impôts"
"lu_2020_account_7021","7021","Sales of finished goods","income","","False","Verkäufe von fertigen Erzeugnissen","Ventes de produits finis"
"lu_2020_account_7022","7022","Sales of semi-finished goods","income","","False","Verkäufe von Zwischenprodukten","Ventes de produits intermédiaires"
"lu_2020_account_7023","7023","Sales of residual products","income","","False","Verkäufe von Restprodukten","Ventes de produits résiduels"
"lu_2020_account_7029","7029","Sales of work in progress","income","","False","Verkäufe von unfertigen Erzeugnissen","Ventes de produits en cours de fabrication"
"lu_2020_account_703001","703001","Sale of Services","income","","False","Verkäufe von Dienstleistungen","Prestations de services"
"lu_2020_account_70311","70311","Concessions","income","","False","Konzessionen",""
"lu_2020_account_70312","70312","Patents","income","","False","Patente","Brevets"
"lu_2020_account_70313","70313","Software licences","income","","False","Software- und Softwarepaketlizenzen","Licences informatiques"
"lu_2020_account_70314","70314","Trademarks and franchises","income","","False","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"lu_2020_account_703151","703151","Copyrights and reproduction rights","income","","False","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"lu_2020_account_703158","703158","Other similar rights and assets","income","","False","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"lu_2020_account_70321","70321","Rental income from real property","income","","False","Mieterträge aus Immobilien","Revenus de location immobilière"
"lu_2020_account_70322","70322","Rental income from movable property","income","","False","Mieterträge aus beweglichem Vermögen","Revenus de location mobilière"
"lu_2020_account_7033","7033","Sales of services not mentioned above","income","","False","Dienstleistungen welche nicht oben erwähnt wurden","Prestations de services non visées ci-dessus"
"lu_2020_account_7039","7039","Sales of services in the course of completion","income","","False","Nicht ausgeführte Dienstleistungen","Prestations de services en cours de réalisation"
"lu_2011_account_704","704","Sales of packaging","income","","False","Verkäufe von Verpackungen","Ventes d'emballages"
"lu_2020_account_705","705","Commissions and brokerage fees","income","","False","Provisionen und Maklergebühren","Commissions et courtages"
"lu_2020_account_7061","7061","Sales of merchandise","income","","False","Verkäufe von Waren","Ventes de marchandises"
"lu_2020_account_7062","7062","Sales of land resale","income","","False","Verkäufe von zum Verkauf bestimmten Grundstücken","Ventes de terrains destinés à la revente"
"lu_2020_account_7063","7063","Sales of buildings for resale","income","","False","Verkäufe von zum Verkauf bestimmten Bauten/Gebäuden","Ventes d'immeubles destinés à la revente"
"lu_2020_account_708","708","Other components of turnover","income","","False","Sonstige Umsatzerlöse","Autres éléments du chiffre d'affaires"
"lu_2011_account_7092","7092","RDR on sales of goods","income","","False","RPR auf Verkäufe von Erzeugnissen","RRR sur ventes de produits"
"lu_2011_account_7093","7093","RDR on sales of services","income","","False","RPR auf Verkäufe von Dienstleistungen","RRR sur prestations de services"
"lu_2011_account_7094","7094","RDR on sales of packages","income","","False","RPR auf Verkäufe von Verpackungen","RRR sur ventes d'emballages"
"lu_2011_account_7095","7095","RDR on commissions and brokerage fees","income","","False","RPR auf Kommissionen und Courtagen","RRR sur commissions et courtages"
"lu_2011_account_7096","7096","RDR on sales of merchandise and other goods for resale","income","","False","RPR auf Verkäufe von Waren und zum Verkauf bestimmten Gütern/Vermögensgegenständen","RRR sur ventes de marchandises et d'autres biens destinés à la revente"
"lu_2011_account_7098","7098","RDR on other components of turnover","income","","False","RPR auf sonstige Umsatzerlöse","RRR sur autres éléments du chiffre d'affaires"
"lu_2020_account_7099","7099","Not allocated rebates, discounts and refunds","income","","False","Nicht zugeordnete RPR","RRR non affectés"
"lu_2011_account_7111","7111","Change in inventories of work in progress","income","","False","Bestandsveränderung von unfertigen Erzeugnissen","Variation des stocks de produits en cours de fabrication"
"lu_2011_account_7112","7112","Change in inventories: contracts in progress - goods","income","","False","Bestandsveränderung von in Arbeit befindlichen Aufträgen - Erzeugnisse","Variation des stocks : commandes en cours – produits"
"lu_2011_account_7113","7113","Change in inventories: contracts in progress - services","income","","False","Bestandsveränderung von in Arbeit befindlichen Aufträgen - Dienstleistungen","Variation des stocks : commandes en cours – prestations de services"
"lu_2011_account_7114","7114","Change in inventories: buildings under construction","income","","False","Bestandsveränderung von im Bau befindlichen Bauten","Variation des stocks : immeubles en construction"
"lu_2011_account_7121","7121","Change in inventories of finished goods","income","","False","Bestandsveränderung von fertigen Erzeugnissen","Variation des stocks de produits finis"
"lu_2011_account_7122","7122","Change in inventories of semi-finished goods","income","","False","Bestandsveränderung von Zwischenprodukten","Variation des stocks de produits intermédiaires"
"lu_2011_account_7123","7123","Change in inventories of residual goods","income","","False","Bestandsveränderung von Restprodukten","Variation des stocks de produits résiduels"
"lu_2011_account_7211","7211","Development costs","income","","False","Entwicklungskosten","Frais de développement"
"lu_2011_account_72121","72121","Concessions","income","","False","Konzessionen",""
"lu_2011_account_72122","72122","Patents","income","","False","Patente","Brevets"
"lu_2011_account_72123","72123","Software licences","income","","False","Software- und Softwarepaketlizenzen","Licences informatiques"
"lu_2011_account_72124","72124","Trademarks and franchises","income","","False","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"lu_2011_account_721251","721251","Copyrights and reproduction rights","income","","False","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"lu_2011_account_721258","721258","Other similar rights and assets","income","","False","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"lu_2011_account_7221","7221","Land, fittings and buildings","income","","False","Grundstücke, Erschließungen, Einrichtungen und Bauten","Terrains, aménagements et constructions"
"lu_2011_account_7222","7222","Plant and machinery","income","","False","Technische Anlagen und Maschinen","Installations techniques et machines"
"lu_2011_account_7223","7223","Other fixtures and fittings, tools and equipment (included motor vehicles)","income","","False","Sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Autres installations, outillage et mobilier (y compris matériel roulant)"
"lu_2020_account_7321","7321","RVA on development costs","income","","False","Wertaufholungen von Entwicklungskosten","RCV sur frais de développement"
"lu_2011_account_7322","7322","RVA on concessions, patents, licences, trademarks and similar rights and assets","income","","False","Wertaufholungen von Konzessionen, Patenten, Lizenzen, Warenzeichen und ähnlichen Rechten und Werten","RCV sur concessions, brevets, licences, marques ainsi que droits et valeurs similaires"
"lu_2011_account_7324","7324","RVA on down payments and intangible fixed assets under development","income","","False","Wertaufholungen von geleisteten Anzahlungen und immateriellen Vermögensgegenständen in Entwicklung","RCV sur acomptes versés et immobilisations incorporelles en cours"
"lu_2011_account_73311","73311","RVA on land","income","","False","Wertaufholungen von Grundstücken, Erschließungen, Einrichtungen und Bauten","RCV sur terrains"
"lu_2011_account_73312","73312","RVA on fixtures and fittings-out of land","income","","False","Wertaufholungen von Erschließungen von Grundstücken","RCV sur agencements et aménagements de terrains"
"lu_2011_account_73313","73313","RVA on buildings","income","","False","Wertaufholungen von Bauten","RCV sur constructions / bâtiments"
"lu_2011_account_73314","73314","RVA on fixtures and fittings-out of buildings","income","","False","Wertaufholungen von Einrichtungen von Bauten/Gebäuden","RCV sur agencements et aménagements de constructions / bâtiments"
"lu_2020_account_73315","73315","FVA on investment properties","income","","False","AFV von Anlageimmobilien","AJV sur immeubles de placement"
"lu_2011_account_7332","7332","RVA on plant and machinery","income","","False","Wertaufholungen von technischen Anlagen und Maschinen","RCV sur installations techniques et machines"
"lu_2011_account_7333","7333","Other fixtures and fittings, tools and equipment (included motor vehicles)","income","","False","Sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Autres installations, outillage et mobilier (y compris matériel roulant)"
"lu_2011_account_7334","7334","RVA on down payments and tangible fixed assets under development","income","","False","Wertaufholungen von geleisteten Anzahlungen und Anlagen im Bau","RCV sur acomptes versés et immobilisations corporelles en cours"
"lu_2011_account_7341","7341","RVA on inventories of raw materials and consumables","income","","False","Wertaufholungen von Roh-, Hilfs- und Betriebsstoffen","RCV sur stocks de matières premières et consommables"
"lu_2011_account_7342","7342","RVA on inventories of work and contracts in progress","income","","False","Wertaufholungen von unfertigen Erzeugnisse und sich in Arbeit befindlichen Aufträgen","RCV sur stocks de produits en cours de fabrication et commandes en cours"
"lu_2011_account_7343","7343","RVA on inventories of goods","income","","False","Erzeugnisse","RCV sur stocks de produits"
"lu_2011_account_7344","7344","RVA on inventories of merchandise and other goods for resale","income","","False","Waren und sonstige zum Verkauf bestimmte Güter/Vermögensgegenstände","RCV sur stocks de marchandises et autres biens destinés à la revente"
"lu_2011_account_7345","7345","RVA on down payments on inventories","income","","False","Geleistete Anzahlungen auf Vorräte","RCV sur acomptes versés sur stocks"
"lu_2011_account_7351","7351","RVA on trade receivables","income","","False","Forderungen aus Lieferungen und Leistungen","RCV sur créances résultant de ventes et prestations de services"
"lu_2011_account_7352","7352","RVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Forderungen gegen verbundene Unternehmen und gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","RCV sur créances sur des entreprises liées et sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_7353","7353","RVA on other receivables","income","","False","Sonstige Forderungen","RCV sur autres créances"
"lu_2020_account_7354","7354","FVA on receivables from current assets","income","","False","AFV von Forderungen des Umlaufvermögens","AJV sur créances de l'actif circulant"
"lu_2011_account_7411","7411","Concessions","income_other","","False","Konzessionen",""
"lu_2011_account_7412","7412","Patents","income_other","","False","Patente","Brevets"
"lu_2011_account_7413","7413","Software licences","income_other","","False","Software- und Softwarepaketlizenzen","Licences informatiques"
"lu_2011_account_7414","7414","Trademarks and franchises","income_other","","False","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"lu_2011_account_74151","74151","Copyrights and reproduction rights","income_other","","False","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"lu_2011_account_74158","74158","Other similar rights and assets","income_other","","False","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"lu_2020_account_7421","7421","Rental income on real property","income_other","","False","Mieterträge aus Immobilien","Revenus de location immobilière"
"lu_2020_account_7422","7422","Rental income on movable property","income_other","","False","Mieterträge aus beweglichem Vermögen","Revenus de location mobilière"
"lu_2011_account_743","743","Attendance fees, director's fees and similar remunerations","income_other","","False","Sitzungsgeld, Tantiemen und vergleichbare Erträge","Jetons de présence, tantièmes et rémunérations assimilées"
"lu_2020_account_74411","74411","Book value of yielded intangible fixed assets","income_other","","False","Buchwert der verkauften immateriellen Anlagewerte","Valeur comptable d'immobilisations incorporelles cédées"
"lu_2020_account_74412","74412","Disposal proceeds of intangible fixed assets","income_other","","False","Verkaufserlös von immateriellen Anlagewerten","Produits de cession d'immobilisations incorporelles"
"lu_2020_account_74421","74421","Book value of yielded tangible fixed assets","income_other","","False","Buchwert der verkauften materiellen Anlagewerte","Valeur comptable d'immobilisations corporelles cédées"
"lu_2020_account_74422","74422","Disposal proceeds of tangible fixed assets","income_other","","False","Verkaufserlös von materiellen Anlagewerten","Produits de cession d'immobilisations corporelles"
"lu_2020_account_7451","7451","Product subsidies","income_other","","False","Produktzuschüsse","Subventions sur produits"
"lu_2020_account_7452","7452","Interest subsidies","income_other","","False","Zinszuschüsse","Bonifications d'intérêt"
"lu_2020_account_7453","7453","Compensatory allowances","income_other","","False","Ausgleichszahlungen","Indemnités compensatoires"
"lu_2020_account_7454","7454","Subsidies in favour of employment development","income_other","","False","Zuschüsse zur Beschäftigungsförderung","Subventions destinées à promouvoir l'emploi"
"lu_2020_account_7458","7458","Other subsidies for operating activities","income_other","","False","Sonstige Zuschüsse für die laufende Geschäftstätigkeit","Autres subventions d'exploitation"
"lu_2011_account_746","746","Benefits in kind","income_other","","False","Sachleistungen","Avantages en nature"
"lu_2011_account_7471","7471","Temporarily not taxable capital gains not reinvested","income_other","","False","Sonderposten mit Rücklageanteil, nicht wiederangelegt","Plus-values immunisées non réinvesties"
"lu_2011_account_7472","7472","Temporarily not taxable capital gains reinvested","income_other","","False","Sonderposten mit Rücklageanteil, wiederangelegt","Plus-values immunisées réinvesties"
"lu_2011_account_7473","7473","Capital investment subsidies","income_other","","False","Investitionszuschüsse","Subventions d'investissement en capital"
"lu_2020_account_7481","7481","Insurance indemnities","income_other","","False","Versicherungsentschädigungen","Indemnités d'assurance"
"lu_2020_account_7488","7488","Miscellaneous operating income","income_other","","False","Andere betriebliche Erträge","Produits d'exploitation divers"
"lu_2020_account_7491","7491","Reversals of provisions for taxes","income_other","","False","Wertaufholungen von Steuerrückstellungen","Reprises de provisions pour impôts"
"lu_2020_account_7492","7492","Reversals of operating provisions","income_other","","False","Wertaufholungen von betrieblichen Rückstellungen","Reprises de provisions d'exploitation"
"lu_2011_account_75111","75111","Shares in affiliated undertakings","income","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2011_account_75112","75112","Amounts owed by affiliated undertakings","income","","False","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"lu_2011_account_75113","75113","Participating interests","income","","False","Anteile","Participations"
"lu_2011_account_75114","75114","Amounts owed by undertakings with which the company is linked by virtue of participating interests","income","","False","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_75115","75115","Securities held as fixed assets","income","","False","Wertpapiere des Anlagevermögens","Titres ayant le caractère d'immobilisations"
"lu_2011_account_75116","75116","Loans, deposits and claims held as fixed assets","income","","False","Ausleihungen, geleistete Hinterlegungen und Forderungen (Anlagevermögen)","Prêts, dépôts et créances immobilisés"
"lu_2011_account_7512","7512","FVA on financial fixed assets","income","","False","AFV der Finanzanlagen","AJV sur immobilisations financières"
"lu_2020_account_75211","75211","Shares in affiliated undertakings","income","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2020_account_75212","75212","Amounts owed by affiliated undertakings","income","","False","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"lu_2020_account_75213","75213","Participating interests","income","","False","Anteile","Participations"
"lu_2020_account_75214","75214","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_75215","75215","Securities held as fixed assets","income","","False","Wertpapiere des Anlagevermögens","Titres ayant le caractère d'immobilisations"
"lu_2020_account_75216","75216","Loans, deposits and claims held as fixed assets","income","","False","Ausleihungen, geleistete Hinterlegungen und Forderungen (Anlagevermögen)","Prêts, dépôts et créances immobilisés"
"lu_2020_account_752211","752211","Book value of yielded shares in affiliated undertakings","income","","False","Buchwert der verkauften Beteiligungen an verbundenen Unternehmen","Valeur comptable de parts cédées dans des entreprises liées"
"lu_2020_account_752212","752212","Disposal proceeds of shares in affiliated undertakings","income","","False","Veräußerungswert der Anteile an verbundenen Unternehmen","Produits de cession de parts dans des entreprises liées"
"lu_2020_account_752221","752221","Book value of yielded amounts owed by affiliated undertakings","income","","False","Buchwert der verkauften Forderungen gegen verbundene Unternehmen","Valeur comptable de créances cédées sur des entreprises liées"
"lu_2020_account_752222","752222","Disposal proceeds of amounts owed by affiliated undertakings","income","","False","Veräußerungswert der Forderungen gegen verbundene Unternehmen","Produits de cession de créances sur des entreprises liées"
"lu_2020_account_752231","752231","Book value of yielded participating interests","income","","False","Buchwert der verkauften Anteile","Valeur comptable de participations cédées"
"lu_2020_account_752232","752232","Disposal proceeds of participating interests","income","","False","Veräußerungswert der Anteile","Produits de cession de participations"
"lu_2020_account_752241","752241","Book value of yielded amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Buchwert der verkauften Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Valeur comptable de créances cédées sur participations"
"lu_2020_account_752242","752242","Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Veräußerungswert der Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Produits de cession de créances sur participations"
"lu_2020_account_752251","752251","Book value of yielded securities held as fixed assets","income","","False","Buchwert der verkauften Wertpapiere des Anlagevermögens","Valeur comptable de titres cédés ayant le caractère d'immobilisations"
"lu_2020_account_752252","752252","Disposal proceeds of securities held as fixed assets","income","","False","Veräußerungswert der Wertpapiere des Anlagevermögens","Produits de cession de titres ayant le caractère d'immobilisations"
"lu_2020_account_752261","752261","Book value of yielded loans, deposits and claims held as fixed assets","income","","False","Buchwert der verkauften Ausleihungen, geleistete Hinterlegungen und Forderungen des Umlaufvermögens","Valeur comptable de prêts, dépôts et créances immobilisés cédés"
"lu_2020_account_752262","752262","Disposal proceed of loans, deposits and claims held as fixed assets","income","","False","Veräußerungswert der Ausleihungen, der geleistete Hinterlegungen und der Forderungen des Anlagevermögens","Produits de cession de prêts, dépôts et créances immobilisés"
"lu_2020_account_75311","75311","Shares in affiliated undertakings","income","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2020_account_75312","75312","Own shares or corporate units","income","","False","Eigene Aktien oder Anteile","Actions propres ou parts propres"
"lu_2020_account_75313","75313","Shares in undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_75318","75318","Other transferable securities","income","","False","Sonstige Wertpapiere","Autres valeurs mobilières"
"lu_2011_account_7532","7532","Fair value adjustments on transferable securities","income","","False","AFV der Wertpapiere","AJV sur valeurs mobilières"
"lu_2011_account_75411","75411","On affiliated undertakings","income","","False","Forderungen gegen verbundene Unternehmen","Sur des entreprises liées"
"lu_2011_account_75412","75412","On undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_75413","75413","On other current receivables","income","","False","Sonstige Forderungen des Umlaufvermögens","Sur autres créances de l'actif circulant"
"lu_2020_account_75421","75421","Shares in affiliated undertakings","income","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2020_account_75422","75422","Own shares or corporate units","income","","False","Eigene Aktien oder Anteile","Actions propres ou parts propres"
"lu_2020_account_75423","75423","Shares in undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_75428","75428","Other transferable securities","income","","False","Sonstige Wertpapiere","Autres valeurs mobilières"
"lu_2011_account_75481","75481","Shares in affiliated undertakings","income","","False","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"lu_2011_account_75482","75482","Own shares or corporate units","income","","False","Eigene Aktien oder Anteile","Actions propres ou parts propres"
"lu_2011_account_75483","75483","Shares in undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2011_account_75488","75488","Other transferable securities","income","","False","Sonstige Wertpapiere","Autres valeurs mobilières"
"lu_2011_account_75521","75521","Interest on bank accounts","income","","False","Kontozinsen","Intérêts sur comptes bancaires"
"lu_2020_account_755231","755231","From affiliated undertakings","income","","False","Forderungen gegen verbundene Unternehmen","Sur des entreprises liées"
"lu_2020_account_755232","755232","From other","income","","False","Von sonstigen","Intérêts sur leasings financiers - autres"
"lu_2011_account_7553","7553","Interest on trade receivables","income","","False","Zinsen auf Handelsforderungen","Intérêts sur créances commerciales"
"lu_2020_account_75541","75541","Interest on amounts owed by affiliated undertakings","income","","False","Zinsen auf Forderungen gegen verbundene Unternehmen","Intérêts sur des entreprises liées"
"lu_2020_account_75542","75542","Interest on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","income","","False","Zinsen auf Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Intérêts sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"lu_2020_account_75551","75551","Discounts on bills of exchange - affiliated undertakings","income","","False","Diskonte auf Handelswechsel - verbundene Unternehmen","Escomptes d'effets de commerce - entreprises liées"
"lu_2020_account_75552","75552","Discounts on bills of exchange - other","income","","False","Diskonte auf Handelswechsel - sonstige","Escomptes d'effets de commerce - autres"
"lu_2020_account_75561","75561","Discounts received - affiliated undertakings","income","","False","Erhaltene Diskonten - verbundene Unternehmen","Escomptes obtenus - entreprises liées"
"lu_2020_account_75562","75562","Discounts received - other","income","","False","Erhaltene Diskonten - sonstige","Escomptes obtenus - autres"
"lu_2020_account_75581","75581","Interest on other amounts receivable - affiliated undertakings","income","","False","Zinsen auf sonstigen Forderungen - verbundene Unternehmen","Intérêts sur autres créances - entreprises liées"
"lu_2020_account_75582","75582","Interest on other amounts receivable - other","income","","False","Zinsen auf sonstigen Forderungen - sonstige","Intérêts sur autres créances - autres"
"lu_2020_account_7561","7561","Foreign currency exchange gains - affiliated undertakings","income","","False","Wechselkursgewinne - verbundene Unternehmen","Gains de change - entreprises liées"
"lu_2020_account_7562","7562","Foreign currency exchange gains - other","income","","False","Wechselkursgewinne - sonstige","Gains de change - autres"
"lu_2011_account_757","757","Share of profit from undertakings accounted for under the equity method","income","","False","Gewinnanteil aus Unternehmungen (andere als Kapitalgesellschaften)","Quote-part de bénéfice dans les entreprises mises en équivalence"
"lu_2020_account_7581","7581","Other financial income - affiliated undertakings","income","","False","Sonstige finanzielle Erträge - verbundene Unternehmen","Autres produits financiers - entreprises liées"
"lu_2020_account_7582","7582","Other financial income - other","income","","False","Sonstige finanzielle Erträge - sonstige","Autres produits financiers - autres"
"lu_2020_account_7591","7591","Reversals of financial provisions - affiliated undertakings","income","","False","Wertaufholungen von finanziellen Rückstellungen - verbundene Unternehmen","Reprises de provisions financières - entreprises liées"
"lu_2020_account_7592","7592","Reversals of financial provisions - other","income","","False","Wertaufholungen von finanziellen Rückstellungen - sonstige","Reprises de provisions financières - autres"
"lu_2011_account_771","771","Adjustments of corporate income tax (CIT)","income","","False","Erstattung der Körperschaftssteuer","Régularisations d'impôt sur le revenu des collectivités (IRC)"
"lu_2011_account_772","772","Adjustments of municipal business tax (MBT)","income","","False","Erstattung der Gewerbesteuer","Régularisations d'impôt commercial communal (ICC)"
"lu_2011_account_773","773","Adjustments of foreign income taxes","income","","False","Erstattung der ausländischen Ertragssteuern","Régularisations d'impôts étrangers sur le résultat"
"lu_2020_account_779","779","Reversals of provisions for deferred taxes","income","","False","Auflösungen von Rückstellungen für Ertragssteuern","Reprises de provisions pour impôts différés"
"lu_2011_account_781","781","Adjustments of net wealth tax (NWT)","income","","False","Erstattung der Vermögensteuer","Régularisations d'impôt sur la fortune (IF)"
"lu_2011_account_782","782","Adjustments of subscription tax","income","","False","Erstattung der Abgeltungssteuer","Régularisations de taxe d'abonnement"
"lu_2011_account_783","783","Adjustments of foreign taxes","income","","False","Erstattung der ausländischen Steuern","Régularisations d'impôts étrangers"
"lu_2011_account_788","788","Adjustments of other taxes","income","","False","Erstattung sonstiger Steuern","Régularisations d'autres impôts"

```

## File: data\template\account.fiscal.position-lu.csv

```csv
"id","name","sequence","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@de","name@fr"
"account_fiscal_position_template_LU_NO","Not liable to VAT","0","","","","","","","",""
"account_fiscal_position_template_LU_LU","Luxembourgish Taxable Person","1","1","1","base.lu","","","","",""
"account_fiscal_position_template_private_LU_IC","EU private","2","1","","","base.europe","","","EU privat","EU privé"
"account_fiscal_position_template_LU_IC","Intra-Community Taxable Person","3","1","1","","base.europe","lu_2015_tax_VB-PA-17","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-16","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-14","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-13","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-8","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-7","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2011_tax_VB-PA-3","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2011_tax_VB-PA-0","lu_2011_tax_VB-IC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-17","lu_2011_tax_VP-IC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-16","lu_2011_tax_VP-IC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-14","lu_2011_tax_VP-IC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-13","lu_2011_tax_VP-IC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-8","lu_2011_tax_VP-IC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-7","lu_2011_tax_VP-IC-0","",""
"","","","","","","","lu_2011_tax_VP-PA-3","lu_2011_tax_VP-IC-0","",""
"","","","","","","","lu_2015_tax_FB-PA-17","lu_2015_tax_FB-IC-17","",""
"","","","","","","","lu_2015_tax_FB-PA-16","lu_2015_tax_FB-IC-16","",""
"","","","","","","","lu_2015_tax_FB-PA-14","lu_2015_tax_FB-IC-14","",""
"","","","","","","","lu_2015_tax_FB-PA-13","lu_2015_tax_FB-IC-13","",""
"","","","","","","","lu_2015_tax_FB-PA-8","lu_2015_tax_FB-IC-8","",""
"","","","","","","","lu_2015_tax_FB-PA-7","lu_2015_tax_FB-IC-7","",""
"","","","","","","","lu_2011_tax_FB-PA-3","lu_2011_tax_FB-IC-3","",""
"","","","","","","","lu_2011_tax_FB-PA-0","lu_2011_tax_FB-IC-0","",""
"","","","","","","","lu_2015_tax_FP-PA-17","lu_2015_tax_FP-IC-17","",""
"","","","","","","","lu_2015_tax_FP-PA-16","lu_2015_tax_FP-IC-16","",""
"","","","","","","","lu_2015_tax_FP-PA-14","lu_2015_tax_FP-IC-14","",""
"","","","","","","","lu_2015_tax_FP-PA-13","lu_2015_tax_FP-IC-13","",""
"","","","","","","","lu_2015_tax_FP-PA-8","lu_2015_tax_FP-IC-8","",""
"","","","","","","","lu_2015_tax_FP-PA-7","lu_2015_tax_FP-IC-7","",""
"","","","","","","","lu_2011_tax_FP-PA-3","lu_2011_tax_FP-IC-3","",""
"","","","","","","","lu_2011_tax_FP-PA-0","lu_2011_tax_FP-IC-0","",""
"","","","","","","","lu_2015_tax_IB-PA-17","lu_2015_tax_IB-IC-17","",""
"","","","","","","","lu_2015_tax_IB-PA-16","lu_2015_tax_IB-IC-16","",""
"","","","","","","","lu_2015_tax_IB-PA-14","lu_2015_tax_IB-IC-14","",""
"","","","","","","","lu_2015_tax_IB-PA-13","lu_2015_tax_IB-IC-13","",""
"","","","","","","","lu_2015_tax_IB-PA-8","lu_2015_tax_IB-IC-8","",""
"","","","","","","","lu_2015_tax_IB-PA-7","lu_2015_tax_IB-IC-7","",""
"","","","","","","","lu_2011_tax_IB-PA-3","lu_2011_tax_IB-IC-3","",""
"","","","","","","","lu_2011_tax_IB-PA-0","lu_2011_tax_IB-IC-0","",""
"","","","","","","","lu_2015_tax_IP-PA-17","lu_2015_tax_IP-IC-17","",""
"","","","","","","","lu_2015_tax_IP-PA-16","lu_2015_tax_IP-IC-16","",""
"","","","","","","","lu_2015_tax_IP-PA-14","lu_2015_tax_IP-IC-14","",""
"","","","","","","","lu_2015_tax_IP-PA-13","lu_2015_tax_IP-IC-13","",""
"","","","","","","","lu_2015_tax_IP-PA-8","lu_2015_tax_IP-IC-8","",""
"","","","","","","","lu_2015_tax_IP-PA-7","lu_2015_tax_IP-IC-7","",""
"","","","","","","","lu_2011_tax_IP-PA-3","lu_2011_tax_IP-IC-3","",""
"","","","","","","","lu_2011_tax_IP-PA-0","lu_2011_tax_IP-IC-0","",""
"","","","","","","","lu_2015_tax_AB-PA-17","lu_2015_tax_AB-IC-17","",""
"","","","","","","","lu_2015_tax_AB-PA-16","lu_2015_tax_AB-IC-16","",""
"","","","","","","","lu_2015_tax_AB-PA-14","lu_2015_tax_AB-IC-14","",""
"","","","","","","","lu_2015_tax_AB-PA-13","lu_2015_tax_AB-IC-13","",""
"","","","","","","","lu_2015_tax_AB-PA-8","lu_2015_tax_AB-IC-8","",""
"","","","","","","","lu_2015_tax_AB-PA-7","lu_2015_tax_AB-IC-7","",""
"","","","","","","","lu_2011_tax_AB-PA-3","lu_2011_tax_AB-IC-3","",""
"","","","","","","","lu_2011_tax_AB-PA-0","lu_2011_tax_AB-IC-0","",""
"","","","","","","","lu_2015_tax_AP-PA-17","lu_2015_tax_AP-IC-17","",""
"","","","","","","","lu_2015_tax_AP-PA-16","lu_2015_tax_AP-IC-16","",""
"","","","","","","","lu_2015_tax_AP-PA-14","lu_2015_tax_AP-IC-14","",""
"","","","","","","","lu_2015_tax_AP-PA-13","lu_2015_tax_AP-IC-13","",""
"","","","","","","","lu_2015_tax_AP-PA-8","lu_2015_tax_AP-IC-8","",""
"","","","","","","","lu_2015_tax_AP-PA-7","lu_2015_tax_AP-IC-7","",""
"","","","","","","","lu_2011_tax_AP-PA-3","lu_2011_tax_AP-IC-3","",""
"","","","","","","","lu_2011_tax_AP-PA-0","lu_2011_tax_AP-IC-0","",""
"account_fiscal_position_template_LU_EC","Extra-Community Taxable Person","4","1","","","","lu_2015_tax_VB-PA-17","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-16","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-14","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-13","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-8","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2015_tax_VB-PA-7","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2011_tax_VB-PA-3","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2011_tax_VB-PA-0","lu_2011_tax_VB-EC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-17","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-16","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-14","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-13","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-8","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2015_tax_VP-PA-7","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2011_tax_VP-PA-3","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2011_tax_VP-PA-0","lu_2011_tax_VP-EC-0","",""
"","","","","","","","lu_2015_tax_FB-PA-17","lu_2015_tax_FB-EC-17","",""
"","","","","","","","lu_2015_tax_FB-PA-16","lu_2015_tax_FB-EC-16","",""
"","","","","","","","lu_2015_tax_FB-PA-14","lu_2015_tax_FB-EC-14","",""
"","","","","","","","lu_2015_tax_FB-PA-13","lu_2015_tax_FB-EC-13","",""
"","","","","","","","lu_2015_tax_FB-PA-8","lu_2015_tax_FB-EC-8","",""
"","","","","","","","lu_2015_tax_FB-PA-7","lu_2015_tax_FB-EC-7","",""
"","","","","","","","lu_2011_tax_FB-PA-3","lu_2011_tax_FB-EC-3","",""
"","","","","","","","lu_2011_tax_FB-PA-0","lu_2011_tax_FB-EC-0","",""
"","","","","","","","lu_2015_tax_FP-PA-17","lu_2015_tax_FP-EC-17","",""
"","","","","","","","lu_2015_tax_FP-PA-16","lu_2015_tax_FP-EC-16","",""
"","","","","","","","lu_2015_tax_FP-PA-14","lu_2015_tax_FP-EC-14","",""
"","","","","","","","lu_2015_tax_FP-PA-13","lu_2015_tax_FP-EC-13","",""
"","","","","","","","lu_2015_tax_FP-PA-8","lu_2015_tax_FP-EC-8","",""
"","","","","","","","lu_2015_tax_FP-PA-7","lu_2015_tax_FP-EC-7","",""
"","","","","","","","lu_2011_tax_FP-PA-3","lu_2011_tax_FP-EC-3","",""
"","","","","","","","lu_2011_tax_FP-PA-0","lu_2011_tax_FP-EC-0","",""
"","","","","","","","lu_2015_tax_IB-PA-17","lu_2015_tax_IB-EC-17","",""
"","","","","","","","lu_2015_tax_IB-PA-16","lu_2015_tax_IB-EC-16","",""
"","","","","","","","lu_2015_tax_IB-PA-14","lu_2015_tax_IB-EC-14","",""
"","","","","","","","lu_2015_tax_IB-PA-13","lu_2015_tax_IB-EC-13","",""
"","","","","","","","lu_2015_tax_IB-PA-8","lu_2015_tax_IB-EC-8","",""
"","","","","","","","lu_2015_tax_IB-PA-7","lu_2015_tax_IB-EC-7","",""
"","","","","","","","lu_2011_tax_IB-PA-3","lu_2011_tax_IB-EC-3","",""
"","","","","","","","lu_2011_tax_IB-PA-0","lu_2011_tax_IB-EC-0","",""
"","","","","","","","lu_2015_tax_IP-PA-17","lu_2015_tax_IP-EC-17","",""
"","","","","","","","lu_2015_tax_IP-PA-16","lu_2015_tax_IP-EC-16","",""
"","","","","","","","lu_2015_tax_IP-PA-14","lu_2015_tax_IP-EC-14","",""
"","","","","","","","lu_2015_tax_IP-PA-13","lu_2015_tax_IP-EC-13","",""
"","","","","","","","lu_2015_tax_IP-PA-8","lu_2015_tax_IP-EC-8","",""
"","","","","","","","lu_2015_tax_IP-PA-7","lu_2015_tax_IP-EC-7","",""
"","","","","","","","lu_2011_tax_IP-PA-3","lu_2011_tax_IP-EC-3","",""
"","","","","","","","lu_2011_tax_IP-PA-0","lu_2011_tax_IP-EC-0","",""
"","","","","","","","lu_2015_tax_AB-PA-17","lu_2015_tax_AB-EC-17","",""
"","","","","","","","lu_2015_tax_AB-PA-16","lu_2015_tax_AB-EC-16","",""
"","","","","","","","lu_2015_tax_AB-PA-14","lu_2015_tax_AB-EC-14","",""
"","","","","","","","lu_2015_tax_AB-PA-13","lu_2015_tax_AB-EC-13","",""
"","","","","","","","lu_2015_tax_AB-PA-8","lu_2015_tax_AB-EC-8","",""
"","","","","","","","lu_2015_tax_AB-PA-7","lu_2015_tax_AB-EC-7","",""
"","","","","","","","lu_2011_tax_AB-PA-3","lu_2011_tax_AB-EC-3","",""
"","","","","","","","lu_2011_tax_AB-PA-0","lu_2011_tax_AB-EC-0","",""
"","","","","","","","lu_2015_tax_AP-PA-17","lu_2015_tax_AP-EC-17","",""
"","","","","","","","lu_2015_tax_AP-PA-16","lu_2015_tax_AP-EC-16","",""
"","","","","","","","lu_2015_tax_AP-PA-14","lu_2015_tax_AP-EC-14","",""
"","","","","","","","lu_2015_tax_AP-PA-13","lu_2015_tax_AP-EC-13","",""
"","","","","","","","lu_2015_tax_AP-PA-8","lu_2015_tax_AP-EC-8","",""
"","","","","","","","lu_2015_tax_AP-PA-7","lu_2015_tax_AP-EC-7","",""
"","","","","","","","lu_2011_tax_AP-PA-3","lu_2011_tax_AP-EC-3","",""
"","","","","","","","lu_2011_tax_AP-PA-0","lu_2011_tax_AP-EC-0","",""

```

## File: data\template\account.group-lu.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@de","name@fr"
"account_group_1","1","","EQUITY, PROVISIONS AND FINANCIAL LIABILITIES ACCOUNTS","KONTEN FÜR EIGENKAPITAL, RÜCKSTELLUNGEN UND FINANZIELLE VERBINDLICHKEITEN","COMPTES DE CAPITAUX PROPRES, DE PROVISIONS ET DE PASSIFS FINANCIERS"
"account_group_10","10","","Subscribed capital or branches' assigned capital and owner's account","Gezeichnetes Kapital oder zugewiesenes Kapital der Zweigniederlassungen und Eigentümerkonto","Capital souscrit ou capital attribué par les succursales et compte du propriétaire"
"account_group_101","101","","Subscribed capital","Gezeichnetes Kapital","Capital souscrit"
"account_group_102","102","","Subscribed capital not called","Gezeichnetes nicht eingefordertes Kapital","Capital souscrit non appelé"
"account_group_103","103","","Subscribed capital called but unpaid","Gezeichnetes eingefordertes und nicht eingezahltes Kapital","Capital souscrit appelé et non versé"
"account_group_104","104","","Capital of individual companies, corporate partnerships and similar","Kapital von Einzelkaufleuten, Personenhandelsgesellschaften und ähnlichen","Capital des entreprises individuelles, des sociétés de personnes et assimilées"
"account_group_105","105","","Endowment of branches","Dotationskapital von Niederlassungen","Dotation des succursales"
"account_group_106","106","","Account of the owner or the co-owners","Konto des Eigentümers oder der Miteigentümer","Compte du propriétaire ou des copropriétaires"
"account_group_11","11","","Share premium and similar premiums","Aktienagio und ähnliche Prämien","Primes d'émission et primes assimilées"
"account_group_111","111","","Share premium","Ausgabeagio","Primes d'émission"
"account_group_112","112","","Merger premium","Agio bei Verschmelzungen","Primes de fusion"
"account_group_113","113","","Contribution premium","Agio bei Einlagen","Primes d'apport"
"account_group_114","114","","Premiums on conversion of bonds into shares","Agio bei der Umwandlung von Anleihen in Aktien","Primes de conversion d'obligations en actions"
"account_group_115","115","","Capital contribution without issue of shares","Kapitaleinlagen ohne Ausgabe von Anteilen","Apport en capitaux propres non rémunéré par des  titres"
"account_group_12","12","","Revaluation reserves","Neubewertungsrücklagen","Réserves de réévaluation"
"account_group_122","122","","Reserves in application of the equity method","Rücklagen durch Anwendung der Kapitalanteilsmethode","Réserves de mise en équivalence"
"account_group_123","123","","Temporarily not taxable currency translation adjustments","Rücklagen aus der Währungsumrechnung (unversteuert)","Plus-values sur écarts de conversion immunisées"
"account_group_128","128","","Other revaluation reserves","Sonstige Neubewertungsrücklagen","Autres réserves de réévaluation"
"account_group_13","13","","Reserves","Reserven","Réserves"
"account_group_131","131","","Legal reserve","Gesetzliche Rücklage","Réserve légale"
"account_group_132","132","","Reserves for own shares or own corporate units","Rücklagen für eigene Aktien oder eigene Anteile","Réserve pour actions propres ou parts propres"
"account_group_133","133","","Reserves provided for by the articles of association","Satzungsmäßige Rücklagen","Réserves statutaires"
"account_group_138","138","","Other reserves, including fair-value reserve","Sonstige Rücklagen, einschließlich der Neubewertungsrücklage","Autres réserves, y compris la réserve de juste valeur"
"account_group_1381","1381","","Other reserves available for distribution","Sonstige freie Rücklagen","Autres réserves disponibles"
"account_group_1382","1382","","Other reserves not available for distribution","Sonstige nicht zur Ausschüttung verfügbare Rücklagen","Autres réserves non distribuables"
"account_group_13821","13821","","Reserve for net wealth tax (NWT)","Vermögensteuerrücklage","Réserve pour l'impôt sur la fortune (IF)"
"account_group_13822","13822","","Reserves in application of fair value","Rücklagen durch Anwendung des beizulegenden Zeitwerts (Fair Value)","Réserves en application de la juste valeur"
"account_group_13823","13823","","Temporarily not taxable capital gains","Vorübergehend nicht steuerpflichtige Kapitalgewinne","Plus-values temporairement non imposables"
"account_group_138231","138231","","Temporarily not taxable capital gains to reinvest","Sonderposten mit Rücklageanteil, zur Wiederanlage bestimmt","Plus-values immunisées à réinvestir"
"account_group_138232","138232","","Temporarily not taxable capital gains reinvested","Sonderposten mit Rücklageanteil, wiederangelegt","Plus-values immunisées réinvesties"
"account_group_13828","13828","","Reserves not available for distribution not mentioned above","Gebundene Rücklagen welche nicht oben aufgelistet wurden","Réserves non disponibles non visées ci-dessus"
"account_group_14","14","","Result for the financial year and results brought forward","Ergebnis des Haushaltsjahres und Ergebnisvortrag","Résultat de l'exercice et résultats reportés"
"account_group_141","141","","Results brought forward","Vorgezogene Ergebnisse","Résultats avancés"
"account_group_1411","1411","","Results brought forward in the process of assignment","Ergebnisvorträge in Zuweisung","Résultats reportés en instance d'affectation"
"account_group_1412","1412","","Results brought forward (assigned)","Ergebnisvorträge (zugewiesen)","Résultats reportés (affectés)"
"account_group_142","142","","Result for the financial year","Ergebnis des Geschäftsjahres","Résultat de l'exercice"
"account_group_15","15","","Interim dividends","Vorabdividenden","Acomptes sur dividendes"
"account_group_16","16","","Capital investment subsidies","Investitionszuschüsse","Subventions d'investissement en capital"
"account_group_161","161","","Subsidies on intangible fixed assets","Subventionen für immaterielle Anlagegüter","Subventions sur les immobilisations incorporelles"
"account_group_1611","1611","","Development costs","Entwicklungskosten","Frais de développement"
"account_group_1612","1612","","Concessions, patents, licences, trademarks and similar rights and assets","Konzessionen, Patente, Lizenzen, Warenzeichen und ähnliche Rechte und Werte","Concessions, brevets, licences, marques et autres droits et actifs similaires"
"account_group_16121","16121","","acquired against payment (except Goodwill)","entgeltlich erworben (außer Goodwill)","acquis à titre onéreux (à l'exception du goodwill)"
"account_group_16122","16122","","created by the undertaking itself","vom Unternehmen selbst geschaffen","créée par l'entreprise elle-même"
"account_group_1613","1613","","Goodwill acquired for consideration","Geschäfts- oder Firmenwert, soweit er entgeltlich erworben wurde","Fonds de commerce, dans la mesure où il a été acquis à titre onéreux"
"account_group_162","162","","Subsidies on tangible fixed assets","Subventionen für Sachanlagen","Subventions sur les immobilisations corporelles"
"account_group_1621","1621","","Subsidies on land, fitting-outs and buildings","Investitionszuschüsse auf Grundstücke, Erschließungen und Bauten","Subventions sur terrains, aménagements et constructions"
"account_group_1622","1622","","Subsidies on plant and machinery","Investitionszuschüsse auf Ttechnische Anlagen und Maschinen","Subventions sur installations techniques et machines"
"account_group_1623","1623","","Subsidies on other fixtures, fittings, tools and equipment (including rolling stock)","Investitionszuschüsse auf sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Subventions sur autres installations, outillage et mobilier (y compris matériel roulant)"
"account_group_168","168","","Other capital investment subsidies","Sonstige Kapitalsubventionen","Autres subventions d'investissement en capital"
"account_group_18","18","","Provisions","Bestimmungen","Dispositions"
"account_group_181","181","","Provisions for pensions and similar obligations","Rückstellungen für Pensionen und ähnliche Verpflichtungen","Provisions pour pensions et obligations similaires"
"account_group_182","182","","Provisions for taxation","Steuerrückstellungen","Provisions pour impôts"
"account_group_183","183","","Deferred tax provisions","Rückstellungen für latente Steuern","Provisions pour impôts différés"
"account_group_188","188","","Other provisions","Sonstige Bestimmungen","Autres dispositions"
"account_group_1881","1881","","Operating provisions","Betriebliche Rückstellungen","Provisions d'exploitation"
"account_group_1882","1882","","Financial provisions","Finanzielle Rückstellungen","Provisions financières"
"account_group_19","19","","Debenture loans and amounts owed to credit institutions","Obligationsanleihen und Verbindlichkeiten gegenüber Kreditinstituten","Emprunts obligataires et dettes envers les établissements de crédit"
"account_group_192","192","","Convertible debenture loans","Anleihen mit Wandelschuldverschreibungen","Emprunts obligataires convertibles"
"account_group_1921","1921","","due and payable within one year","fällig und zahlbar innerhalb eines Jahres","exigibles à moins d'un an"
"account_group_1922","1922","","due and payable after more than one year","fällig und zahlbar nach mehr als einem Jahr","exigibles à plus d'un an"
"account_group_193","193","","Non-convertible debenture loans","Nicht konvertierbare Schuldscheindarlehen","Emprunts obligataires non convertibles"
"account_group_1931","1931","","due and payable within one year","fällig und zahlbar innerhalb eines Jahres","exigibles à moins d'un an"
"account_group_1932","1932","","due and payable after more than one year","fällig und zahlbar nach mehr als einem Jahr","exigibles à plus d'un an"
"account_group_194","194","","Amounts owed to credit institutions","Verbindlichkeiten gegenüber Kreditinstituten","Dettes envers les établissements de crédit"
"account_group_1941","1941","","due and payable within one year","fällig und zahlbar innerhalb eines Jahres","exigibles à moins d'un an"
"account_group_1942","1942","","due and payable after more than one year","fällig und zahlbar nach mehr als einem Jahr","exigibles à plus d'un an"
"account_group_2","2","","FORMATION EXPENSES AND FIXED ASSETS ACCOUNTS","KONTEN FÜR GRÜNDUNGSKOSTEN UND ANLAGEVERMÖGEN","COMPTES DE FRAIS D'ÉTABLISSEMENT ET D'IMMOBILISATIONS"
"account_group_20","20","","Formation expenses and similar expenses","Gründungskosten und ähnliche Kosten","Frais d'établissement et frais assimilés"
"account_group_201","201","","Set-up and start-up costs","Gründungs- und Einrichtungskosten","Frais de constitution et de premier établissement"
"account_group_203","203","","Expenses for increases in capital and for various operations (merger, demerger, change of legal form)","Kosten für Kapitalerhöhung und für verschiedene umwandlugsrechtliche Vorgänge (Verschmelzungen, Spaltungen, Formwechsel)","Frais d'augmentation de capital et d'opérations diverses (fusions, scissions, transformations)"
"account_group_204","204","","Loan issuances expenses","Emissionskosten von Anleihen","Frais d'émission d'emprunts"
"account_group_208","208","","Other similar expenses","Sonstige ähnliche Kosten","Autres frais assimilés"
"account_group_21","21","","Intangible fixed assets","Immaterielle Anlagewerte","Immobilisations incorporelles"
"account_group_211","211","","Development costs","Entwicklungskosten","Frais de développement"
"account_group_212","212","","Concessions, patents, licences, trademarks and similar rights and assets","Konzessionen, Patente, Lizenzen, Warenzeichen und ähnliche Rechte und Werte","Concessions, brevets, licences, marques et autres droits et actifs similaires"
"account_group_2121","2121","","acquired for consideration (except Goodwill)","entgeltlich erworben (außer Goodwill)","acquis à titre onéreux (à l'exception du goodwill)"
"account_group_21211","21211","","Concessions","Konzessionen",""
"account_group_21212","21212","","Patents","Patente","Brevets"
"account_group_21213","21213","","Software licences","Software- und Softwarepaketlizenzen","Licences informatiques"
"account_group_21214","21214","","Trademarks and franchises","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"account_group_21215","21215","","Similar rights and assets","Ähnliche Rechte und Werte","Droits et biens similaires"
"account_group_212151","212151","","Copyrights and reproduction rights","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"account_group_212152","212152","","Greenhous gas and similar emission quotas","Quoten für Treibhausgas und ähnliche Emissionen","Quotas d'émission de gaz à effet de serre et similaires"
"account_group_212158","212158","","Other similar rights and assets acquired for consideration","Sonstige entgeltlich erworbene Rechte und Werte","Autres droits et valeurs similaires acquis à titre onéreux"
"account_group_2122","2122","","created by the undertaking itself","vom Unternehmen selbst geschaffen","créée par l'entreprise elle-même"
"account_group_21221","21221","","Concessions","Konzessionen",""
"account_group_21222","21222","","Patents","Patente","Brevets"
"account_group_21223","21223","","Software licences","Software- und Softwarepaketlizenzen","Licences informatiques"
"account_group_21224","21224","","Trademarks and franchises","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"account_group_21225","21225","","Similar rights and assets","Ähnliche Rechte und Werte","Droits et biens similaires"
"account_group_212251","212251","","Copyrights and reproduction rights","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"account_group_212258","212258","","Other similar rights and assets created by the undertaking itself","Sonstige vergleichbare vom Unternehmen selbst erstellte Rechte und Werte","Autres droits et valeurs similaires créés par l'entreprise elle-même"
"account_group_213","213","","Goodwill acquired for consideration","Geschäfts- oder Firmenwert, soweit er entgeltlich erworben wurde","Fonds de commerce, dans la mesure où il a été acquis à titre onéreux"
"account_group_214","214","","Down payments and intangible fixed assets under development","Geleistete Anzahlungen und immaterielle Vermögensgegenstände in Entwicklung","Acomptes versés et immobilisations incorporelles en cours"
"account_group_22","22","","Tangible fixed assets","Materielle Anlagewerte","Immobilisations corporelles"
"account_group_221","221","","Land, fixtures and fitting-outs and buildings","Grundstücke, Einrichtungsgegenstände und Gebäude","Terrains, agencements, aménagements et constructions"
"account_group_2211","2211","","Land","Grundstücke","Terrains"
"account_group_22111","22111","","Land in Luxembourg","Grundstücke in Luxemburg","Terrains au Luxembourg"
"account_group_221111","221111","","Developed land","Bebaute Grundstücke","Terrains bâtis"
"account_group_221112","221112","","Property rights and similar","Immobilien- / Eigentumsrechte auf Sachanlagen und ähnliche","Droits immobiliers et assimilés"
"account_group_221118","221118","","Other land","Sonstige Grundstücke","Autres terrains"
"account_group_22112","22112","","Land in foreign countries","Grundstücke im Ausland","Terrains à l'étranger"
"account_group_2212","2212","","Fixtures and fittings-out of land","Betriebs- und Geschäftsausstattung - außerhalb von Grundstücken","Agencements et aménagements des terrains"
"account_group_22121","22121","","Fixtures and fitting-outs of land in Luxembourg","Erschließung von Grundstücken in Luxembourg","Agencements et aménagements de terrains au Luxembourg"
"account_group_22122","22122","","Fixtures and fitting-outs of land in foreign countries","Erschließung von Grundstücken im Ausland","Agencements et aménagements de terrains à l'étranger"
"account_group_2213","2213","","Buildings","Bauten/Gebäude","Constructions / Bâtiments"
"account_group_22131","22131","","Buildings in Luxembourg","Gebäude in Luxemburg","Bâtiments au Luxembourg"
"account_group_221311","221311","","Residential buildings","Wohngebäude","Constructions / Bâtiments résidentiels"
"account_group_221312","221312","","Non-residential buildings","Zweckbauten","Constructions / Bâtiments non résidentiels"
"account_group_221313","221313","","Mixed-use buildings","Mischnutzungsbauten","Constructions / Bâtiments à usage mixte"
"account_group_221318","221318","","Other buildings","Sonstige Bauten / Gebäude","Autres constructions / bâtiments"
"account_group_22132","22132","","Buildings in foreign countries","Bauten / Gebäude im Ausland","Constructions / Bâtiments à l'étranger"
"account_group_2214","2214","","Fixtures and fitting-outs of buildings","Einrichtungen und Ausstattungen von Gebäuden","Aménagements et installations des bâtiments"
"account_group_22141","22141","","Fixtures and fitting-outs of buildings in Luxembourg","Einrichtung von Bauten / Gebäuden in Luxemburg","Agencements et aménagements de constructions / bâtiments au Luxembourg"
"account_group_22142","22142","","Fixtures and fitting-outs of buildings in foreign countries","Einrichtung von Bauten / Gebäuden im Ausland","Agencements et aménagements de constructions / bâtiments à l'étranger"
"account_group_2215","2215","","Investment properties","Anlageimmobilien","Immeubles de placement"
"account_group_22151","22151","","Investment properties in Luxembourg","Anlageimmobilien in Luxembourg","Immeubles de placement au Luxembourg"
"account_group_22152","22152","","Investment properties in foreign countries","Anlageimmobilien im Ausland","Immeubles de placement à l'étranger"
"account_group_222","222","","Plant and machinery","Technische Anlagen und Maschinen","Installations techniques et machines"
"account_group_2221","2221","","Plant","Technische Anlagen","Installations techniques"
"account_group_2222","2222","","Machinery","Maschinen","Machines"
"account_group_223","223","","Other fixtures and fittings, tools and equipment (including rolling stock)","Sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Autres installations, outillage et mobilier (y compris matériel roulant)"
"account_group_2231","2231","","Transportation and handling equipment","Transport- und Wartungsmittel","Equipement de transport et de manutention"
"account_group_2232","2232","","Motor vehicles","Transportfahrzeuge","Véhicules de transport"
"account_group_2233","2233","","Tools","Werkzeuge","Outillage"
"account_group_2234","2234","","Furniture","Mobiliar","Mobilier"
"account_group_2235","2235","","Computer equipment","IT-Ausstattung","Matériel informatique"
"account_group_2236","2236","","Livestock","Viehbestand","Cheptel"
"account_group_2237","2237","","Returnable packaging","Wiederverwendbare Verpackungen","Emballages récupérables"
"account_group_2238","2238","","Other fixtures","Sonstige Anlagen","Autres installations"
"account_group_224","224","","Down payments and tangible fixed assets under development","Anzahlungen und in Entwicklung befindliche Sachanlagen","Acomptes et immobilisations corporelles en cours de développement"
"account_group_2241","2241","","Land, fitting-outs and buildings","Grundstücke, Ausstattungen und Gebäude","Terrains, aménagements et bâtiments"
"account_group_22411","22411","","Land, fitting-outs and buildings in Luxembourg","Grundstücke, Erschließungen, Einrichtungen und Bauten in Luxemburg","Terrains, aménagements et constructions au Luxembourg"
"account_group_22412","22412","","Land, fitting-outs and buildings in foreign countries","Grundstücke, Erschließungen, Einrichtungen und Bauten im Ausland","Terrains, aménagements et constructions à l'étranger"
"account_group_2242","2242","","Plant and machinery","Technische Anlagen und Maschinen","Installations techniques et machines"
"account_group_2243","2243","","Other fixtures and fittings, tools and equipment (including rolling stock)","Sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Autres installations, outillage et mobilier (y compris matériel roulant)"
"account_group_23","23","","Financial fixed assets","Finanzielle Anlagewerte","Immobilisations financières"
"account_group_231","231","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_232","232","","Amounts owed by affiliated undertakings","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"account_group_233","233","","Participating interests","Anteile","Participations"
"account_group_234","234","","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_235","235","","Securities held as fixed assets","Wertpapiere des Anlagevermögens","Titres ayant le caractère d'immobilisations"
"account_group_2351","2351","","Securities held as fixed assets (equity right)","Wertpapiere des Anlagevermögens (Eigenkapitalrecht)","Titres détenus en tant qu'actifs immobilisés (droit de participation)"
"account_group_23511","23511","","Shares or corporate units","Aktien oder Unternehmensanteile","Actions ou parts sociales"
"account_group_235111","235111","","Listed shares","Notierte Aktien","Actions cotées"
"account_group_235112","235112","","Unlisted shares","Nicht notierte Aktien","Actions non cotées"
"account_group_23518","23518","","Other securities held as fixed assets (equity right)","Sonstige Wertpapiere des Anlagevermögens (Eigentumsrecht)","Autres titres immobilisés (droit de propriété)"
"account_group_2352","2352","","Securities held as fixed assets (creditor's right)","Wertpapiere des Anlagevermögens (Recht des Gläubigers)","Titres immobilisés (droit du créancier)"
"account_group_23521","23521","","Debentures","Anleihen","Obligations"
"account_group_23528","23528","","Other securities held as fixed assets (creditor's right)","Sonstige Wertpapiere des Anlagevermögens (Forderungsrecht)","Autres titres immobilisés (droit de créance)"
"account_group_2353","2353","","Shares of collective investment funds","OPC Anteile","Parts d'OPC"
"account_group_2358","2358","","Other securities held as fixed assets","Sonstige Wertpapiere des Anlagevermögens","Autres titres ayant le caractère d'immobilisations"
"account_group_236","236","","Loans, deposits and claims held as fixed assets","Ausleihungen, geleistete Hinterlegungen und Forderungen (Anlagevermögen)","Prêts, dépôts et créances immobilisés"
"account_group_2361","2361","","Loans","Ausleihungen","Prêts"
"account_group_2362","2362","","Deposits and guarantees paid","Geleistete Hinterlegungen und Kautionen","Dépôts et cautionnements versés"
"account_group_2363","2363","","Long-term receivables","Forderungen","Créances immobilisées"
"account_group_3","3","","INVENTORIES ACCOUNTS","BESTANDSKONTEN","COMPTES D'INVENTAIRE"
"account_group_30","30","","Inventories of raw materials and consumables","Vorräte an Roh-, Hilfs- und Betriebsstoffen","Stocks de matières premières et consommables"
"account_group_301","301","","Inventories of raw materials","Vorräte an Rohstoffen","Stocks de matières premières"
"account_group_303","303","","Inventories of consumable materials and supplies","Vorräte an Hilfs-und Betriebsstoffen","Stocks de matières et fournitures consommables"
"account_group_304","304","","Inventories of packaging","Vorräte an Verpackungen","Stocks d'emballages"
"account_group_31","31","","Inventories of work and contracts in progress","Vorräte an unfertigen Erzeugnissen und laufenden Aufträgen","Stocks de travaux et de contrats en cours"
"account_group_311","311","","Inventories of work in progress","Vorräte an unfertigen Erzeugnissen","Stocks de produits en cours de fabrication"
"account_group_312","312","","Contracts in progress - goods","In Arbeit befindlichen Aufträge - Waren","Commandes en cours – produits"
"account_group_313","313","","Contracts in progress - services","In Arbeit befindlichen Aufträge - Dienstleistungen","Commandes en cours – prestations de services"
"account_group_314","314","","Buildings under construction","Im Bau befindliche Bauten / Gebäude","Immeubles en construction"
"account_group_315","315","","Down payments received on inventories of work and on contracts in progress","Erhaltene Anzahlungen auf Vorräte an Bauleistungen und auf laufende Aufträge","Acomptes reçus sur stocks de travaux et sur commandes en cours d'exécution"
"account_group_32","32","","Inventories of goods","Vorräte an Waren","Stocks de marchandises"
"account_group_321","321","","Inventories of finished goods","Vorräte an fertigen Erzeugnissen","Stocks de produits finis"
"account_group_322","322","","Inventories of semi-finished goods","Vorräte an Zwischenprodukten","Stocks de produits intermédiaires"
"account_group_323","323","","Inventories of residual goods (waste, rejected and recuperable material)","Vorräte an Restprodukten","Stocks de produits résiduels (déchets, rebuts, matières de récupération)"
"account_group_36","36","","Inventories of merchandises and other goods for resale","Vorräte an Handelswaren und sonstigen Waren zum Wiederverkauf","Stocks de marchandises et autres biens destinés à la revente"
"account_group_361","361","","Inventories of merchandise","Vorräte an Waren","Stocks de marchandises"
"account_group_362","362","","Inventories of land for resale","Vorräte an zum Wiederverkauf bestimmten Grundstücken","Stocks de terrains destinés à la revente"
"account_group_3621","3621","","Inventories of land for resale in Luxembourg","Vorräte an, zum Verkauf bestimmten, Grundstücken in Luxemburg","Stocks de terrains au Luxembourg"
"account_group_3622","3622","","Inventories of land for resale in foreign countries","Vorräte an, zum Verkauf bestimmten, Grundstücken im Ausland","Stocks de terrains à l'étranger"
"account_group_363","363","","Inventories of buildings for resale","Vorräte an Gebäuden zum Wiederverkauf","Stocks de bâtiments destinés à la revente"
"account_group_3631","3631","","Inventories of buildings for resale in Luxembourg","Vorräte an, zum Verkauf bestimmten, Bauten / Gebäuden in Luxemburg","Stocks d'immeubles au Luxembourg"
"account_group_3632","3632","","Inventories of buildings for resale in foreign countries","Vorräte an, zum Verkauf bestimmten, Bauten / Gebäuden im Ausland","Stocks d'immeubles à l'étranger"
"account_group_37","37","","Down payments on account on inventories","Geleistete Anzahlungen auf Vorräte","Acomptes versés sur stocks"
"account_group_4","4","","DEBTORS AND CREDITORS","SCHULDNER UND GLÄUBIGER","DÉBITEURS ET CRÉANCIERS"
"account_group_40","40","","Trade receivables (Receivables from sales and rendering of services)","Forderungen aus Lieferungen und Leistungen (Forderungen aus Verkäufen und Dienstleistungen)","Créances commerciales (créances résultant de ventes et de prestations de services)"
"account_group_401","401","","Trade receivables due and payable within one year","Forderungen aus Lieferungen und Leistungen, die innerhalb eines Jahres fällig sind","Créances commerciales exigibles à moins d'un an"
"account_group_4011","4011","","Customers","Kunden","Clients"
"account_group_4012","4012","","Customers - Receivable bills of exchange","Kunden - Einzutreibende Wechsel (Besitzwechsel)","Clients – Effets à recevoir"
"account_group_4013","4013","","Doubtful or disputed customers","Zweifelhafte oder strittige Kunden","Clients douteux ou litigieux"
"account_group_4014","4014","","Customers - Unbilled sales","Noch auszustellende Rechnungen","Clients – Factures à établir"
"account_group_4015","4015","","Customers with a credit balance","Kunden - Kreditorische Debitoren","Clients créditeurs"
"account_group_4019","4019","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_402","402","","Trade receivables due and payable after more than one year","Forderungen aus Lieferungen und Leistungen mit einer Restlaufzeit von mehr als einem Jahr","Créances commerciales à plus d'un an"
"account_group_4021","4021","","Customers","Kunden","Clients"
"account_group_4025","4025","","Customers with creditor balance","Kreditorische Debitoren","Clients créditeurs"
"account_group_4029","4029","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_41","41","","Amounts owed by affiliated undertakings and by undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen verbundene Unternehmen und gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises liées et sur des entreprises avec lesquelles l'entreprise est liée par des participations"
"account_group_411","411","","Amounts owed by affiliated undertakings","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"account_group_4111","4111","","Amounts owed by affiliated undertakings receivable within one year","Forderungen gegen verbundene Unternehmen, die innerhalb eines Jahres fällig sind","Créances sur des entreprises liées à moins d'un an"
"account_group_41111","41111","","Trade receivables","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"account_group_41112","41112","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_41118","41118","","Other receivables","Sonstige Forderungen","Autres créances"
"account_group_41119","41119","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_4112","4112","","Amounts owed by affiliated undertakings receivable after more than one year","Forderungen an verbundene Unternehmen mit einer Laufzeit von mehr als einem Jahr","Créances sur des entreprises liées à plus d'un an"
"account_group_41121","41121","","Trade receivables","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"account_group_41122","41122","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_41128","41128","","Other receivables","Sonstige Forderungen","Autres créances"
"account_group_41129","41129","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_412","412","","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_4121","4121","","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests receivable within one year","Forderungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht, die innerhalb eines Jahres fällig sind","Créances sur des entreprises avec lesquelles l'entreprise est liée par des participations à recevoir dans un délai d'un an"
"account_group_41211","41211","","Trade receivables","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"account_group_41212","41212","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_41218","41218","","Other receivables","Sonstige Forderungen","Autres créances"
"account_group_41219","41219","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_4122","4122","","Amounts receivable after more than one year","Forderungen mit einer Laufzeit von mehr als einem Jahr","Créances à plus d'un an"
"account_group_41221","41221","","Trade receivables","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"account_group_41222","41222","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_41228","41228","","Other receivables","Sonstige Forderungen","Autres créances"
"account_group_41229","41229","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_42","42","","Other receivables","Sonstige Forderungen","Autres créances"
"account_group_421","421","","Other receivables within one year","Sonstige Forderungen innerhalb eines Jahres","Autres créances à moins d'un an"
"account_group_4211","4211","","Staff - Advances and down payments","Personal - Vorschüsse und Anzahlungen","Personnel - Avances et acomptes"
"account_group_42111","42111","","Advances and down payments","Vorschüsse und geleistete Anzahlungen","Avances et acomptes"
"account_group_42119","42119","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_4212","4212","","Amounts owed by partners and shareholders (others than from affiliated undertakings)","Forderungen gegen Gesellschafter und Aktionäre (andere als von verbundenen Unternehmen)","Créances sur associés ou actionnaires (autres qu'entreprises liées)"
"account_group_4213","4213","","State - Subsidies to be received","Staat - Zu erhaltende Subventionen","État - Subventions à recevoir"
"account_group_42131","42131","","Investment subsidies","Investitionszuschüsse","Subventions d'investissement"
"account_group_42132","42132","","Operating subsidies","Betriebszuschüsse","Subventions d'exploitation"
"account_group_42138","42138","","Other subsidies","Sonstige Zuschüsse","Autres subventions"
"account_group_4214","4214","","Direct Tax Authority (ACD)","Direkte Steuerbehörde (ACD)","Autorité des impôts directs (ACD)"
"account_group_42141","42141","","Corporate income tax","Körperschaftssteuer","Impôt sur le revenu des collectivités (IRC)"
"account_group_42142","42142","","Municipal business tax","Gewerbesteuer","Impôt commercial communal (ICC)"
"account_group_42143","42143","","Net wealth tax","Vermögensteuer","Impôt sur la fortune (IF)"
"account_group_42144","42144","","Withholding tax on wages and salaries","Einbehaltene Steuer auf Gehälter und Löhne","Retenue d'impôt sur traitements et salaires (RTS)"
"account_group_42145","42145","","Withholding tax on financial investment income","Einbehaltene Kapitalertragsteuer","Retenue d'impôt sur revenus de capitaux mobiliers"
"account_group_42146","42146","","Withholding tax on director's fees","Einbehaltene Steuer auf Tantiemen","Retenue d'impôt sur les tantièmes"
"account_group_42148","42148","","ACD - Other amounts receivable","ACD - Sonstige Verbindlichkeiten","ACD – Autres créances"
"account_group_4215","4215","","Customs and Excise Authority (ADA)","Zoll- und Verbrauchssteuerverwaltung (ADA)","Administration des Douanes et Accises (ADA)"
"account_group_4216","4216","","Indirect Tax Authority (AED)","Behörde für indirekte Steuern (AED)","Autorité des impôts indirects (AED)"
"account_group_42161","42161","","Value-added tax (VAT)","Mehrwertsteuer (VAT)","Taxe sur la valeur ajoutée (TVA)"
"account_group_421611","421611","","VAT paid and recoverable","Vorsteuer","TVA en amont"
"account_group_421612","421612","","VAT receivable","Zu erhaltende MwSt","TVA à recevoir"
"account_group_421613","421613","","VAT down payments made","Geleistete Anzahlungen auf MwSt","TVA acomptes versés"
"account_group_421618","421618","","VAT - Other receivables","MwSt - Sonstige Forderungen","TVA – Autres créances"
"account_group_42162","42162","","Indirect taxes","Indirekte Steuern","Impôts indirects"
"account_group_421621","421621","","Registration duties","Registriergebühren","Droits d'enregistrement"
"account_group_421622","421622","","Subscription tax","Abgeltungssteuer","Taxe d'abonnement"
"account_group_421628","421628","","Other indirect taxes","Sonstige indirekte Steuern","Autres impôts indirects"
"account_group_42168","42168","","Other receivables","Sonstige Forderungen","Autres créances"
"account_group_4217","4217","","Amounts owed by the Social Security and other social bodies","Von der Sozialversicherung und anderen sozialen Einrichtungen geschuldete Beträge","Montants dus par la sécurité sociale et d'autres organismes sociaux"
"account_group_42171","42171","","Joint Social Security Centre (CCSS)","Gemeinsames Zentrum für soziale Sicherheit (CCSS)","Centre commun de la sécurité sociale (CCSS)"
"account_group_42172","42172","","Foreign social security offices","Ausländische Sozialversicherungen","Organismes étrangers de sécurité sociale"
"account_group_42178","42178","","Other social bodies","Sonstige Einrichtungen der sozialen Sicherheit","Autres organismes sociaux"
"account_group_4218","4218","","Miscellaneous receivables","Verschiedene Forderungen","Créances diverses"
"account_group_42181","42181","","Foreign taxes","Ausländische Steuern","Impôts étrangers"
"account_group_421811","421811","","Foreign VAT","Ausländische MwSt","TVA étrangères"
"account_group_421818","421818","","Other foreign taxes","Sonstige ausländische Steuern","Autres impôts étrangers"
"account_group_42187","42187","","Derivative financial instruments","Derivative Finanzinstrumente","Instruments financiers dérivés"
"account_group_42188","42188","","Other miscellaneous receivables","Andere sonstige Forderungen","Autres créances diverses"
"account_group_42189","42189","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_422","422","","Other receivables after one year","Sonstige Forderungen nach einem Jahr","Autres créances à plus d'un an"
"account_group_4221","4221","","Staff - advances and down payments","Personal - Vorschüsse und geleistete Anzahlungen","Personnel – Avances et acomptes"
"account_group_4222","4222","","Amounts owed by partners and shareholders (others than from affiliated undertakings)","Forderungen gegen Gesellschafter und Aktionäre (andere als von verbundenen Unternehmen)","Créances sur associés ou actionnaires (autres qu'entreprises liées)"
"account_group_4223","4223","","State - Subsidies to be received","Staat - Zu erhaltende Subventionen","État - Subventions à recevoir"
"account_group_42231","42231","","Investment subsidies","Investitionszuschüsse","Subventions d'investissement"
"account_group_42232","42232","","Operating subsidies","Betriebszuschüsse","Subventions d'exploitation"
"account_group_42238","42238","","Other subsidies","Sonstige Zuschüsse","Autres subventions"
"account_group_4228","4228","","Miscellaneous receivables","Verschiedene Forderungen","Créances diverses"
"account_group_42287","42287","","Derivative financial instruments","Derivative Finanzinstrumente","Instruments financiers dérivés"
"account_group_42288","42288","","Other miscellaneous receivables","Andere sonstige Forderungen","Autres créances diverses"
"account_group_42289","42289","","Value adjustments","Wertberichtigungen","Corrections de valeur"
"account_group_43","43","","Down payments received on orders as far as they are not deducted distinctly from inventories","Erhaltene Anzahlungen auf Bestellungen, soweit sie nicht gesondert von den Vorräten abgesetzt werden","Acomptes reçus sur commandes dans la mesure où ils ne sont pas déduits distinctement des stocks"
"account_group_431","431","","Down payments received within one year","Erhaltene Anzahlungen mit einer Restlaufzeit von bis zu einem Jahr","Acomptes reçus dont la durée résiduelle est inférieure ou égale à un an"
"account_group_4311","4311","","Down payments received on orders","Erhaltene Anzahlungen auf Bestellungen","Acomptes reçus sur commandes"
"account_group_4312","4312","","Inventories of work and contracts in progress less down payments received","Vorräte an unfertigen Leistungen und Aufträgen abzüglich erhaltener Anzahlungen","Stocks de travaux et de commandes en cours d'exécution diminués des acomptes reçus"
"account_group_432","432","","Down payments received after more than one year","Erhaltene Anzahlungen mit einer Restlaufzeit von mehr als einem Jahr","Acomptes reçus dont la durée résiduelle est supérieure à un an"
"account_group_4321","4321","","Down payments received on orders","Erhaltene Anzahlungen auf Bestellungen","Acomptes reçus sur commandes"
"account_group_4322","4322","","Inventories of work and contracts in progress less down payments received","Vorräte an unfertigen Leistungen und Aufträgen abzüglich erhaltener Anzahlungen","Stocks de travaux et de commandes en cours d'exécution diminués des acomptes reçus"
"account_group_44","44","","Trade payables and bills of exchange","Verbindlichkeiten aus Lieferungen und Leistungen und Wechselverbindlichkeiten","Dettes commerciales et effets de commerce"
"account_group_441","441","","Trade payables","Verbindlichkeiten aus Lieferungen und Leistungen","Dettes commerciales"
"account_group_4411","4411","","Trade payables within one year","Verbindlichkeiten aus Lieferungen und Leistungen innerhalb eines Jahres","Dettes commerciales à moins d'un an"
"account_group_44111","44111","","Suppliers","Lieferanten","Fournisseurs"
"account_group_44112","44112","","Suppliers - invoices not yet received","Lieferanten - Noch nicht erhaltene Rechnungen","Fournisseurs – Factures non parvenues"
"account_group_44113","44113","","Suppliers with a debit balance","Lieferanten - Debitorische Kreditoren","Fournisseurs débiteurs"
"account_group_4412","4412","","Trade payables after more than one year","Verbindlichkeiten aus Lieferungen und Leistungen nach mehr als einem Jahr","Dettes commerciales à plus d'un an"
"account_group_44121","44121","","Suppliers","Lieferanten","Fournisseurs"
"account_group_44123","44123","","Suppliers with a debit balance","Lieferanten - Debitorische Kreditoren","Fournisseurs débiteurs"
"account_group_442","442","","Bills of exchange payable","Verbindlichkeiten aus Wechseln","Lettres de change à payer"
"account_group_4421","4421","","Bills of exchange payable within one year","Durch Handelswechsel entstandene Verbindlichkeiten (Schuldwechsel) mit einer Restlaufzeit von bis zu einem Jahr","Effets à payer dont la durée résiduelle est inférieure ou égale à un an"
"account_group_4422","4422","","Bills of exchange payable after more than one year","Durch Handelswechsel entstandene Verbindlichkeiten (Schuldwechsel) mit einer Restlaufzeit von mehr als einem Jahr","Effets à payer dont la durée résiduelle est supérieure à un an"
"account_group_45","45","","Amounts payable to affiliated undertakings and to undertakings with which the undertaking is linked by virtue of participating interests","Verbindlichkeiten gegenüber verbundenen Unternehmen und gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Dettes envers des entreprises liées et des entreprises avec lesquelles l'entreprise est liée par des participations"
"account_group_451","451","","Amounts payable to affiliated undertakings","Verbindlichkeiten gegenüber verbundenen Unternehmen","Montants à payer aux entreprises liées"
"account_group_4511","4511","","Amounts payable to affiliated undertakings within one year","An verbundene Unternehmen zu zahlende Beträge innerhalb eines Jahres","Dettes envers des entreprises liées à moins d'un an"
"account_group_45111","45111","","Purchases and services","Käufe und Dienstleistungen","Achats et prestations de services"
"account_group_45112","45112","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_45118","45118","","Other payables","Sonstige Verbindlichkeiten","Autres dettes"
"account_group_4512","4512","","Amounts payable to affiliated undertakings after more than one year","An verbundene Unternehmen zu zahlende Beträge innerhalb eines Jahres","Dettes à plus d'un an envers des entreprises liées"
"account_group_45121","45121","","Purchases and services","Käufe und Dienstleistungen","Achats et prestations de services"
"account_group_45122","45122","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_45128","45128","","Other payables","Sonstige Verbindlichkeiten","Autres dettes"
"account_group_452","452","","Amounts payable to undertakings with which the undertaking is linked by virtue of participating interests","Beträge, die an Unternehmen zu zahlen sind, mit denen das Unternehmen durch Beteiligungsverhältnisse verbunden ist","Dettes envers des entreprises auxquelles l'entreprise est liée par des participations"
"account_group_4521","4521","","Amounts payable to undertakings with which the undertaking is linked by virtue of participating interests within one year","Beträge, die innerhalb eines Jahres an Unternehmen zu zahlen sind, mit denen ein Beteiligungsverhältnis besteht","Dettes envers des entreprises avec lesquelles l'entreprise est liée par des participations dans un délai d'un an"
"account_group_45211","45211","","Purchases and services","Käufe und Dienstleistungen","Achats et prestations de services"
"account_group_45212","45212","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_45218","45218","","Other payables","Sonstige Verbindlichkeiten","Autres dettes"
"account_group_4522","4522","","Amounts payable to undertakings with which the undertaking is linked by virtue of participating interests payable after more than one year","Verbindlichkeiten gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht, die nach mehr als einem Jahr fällig werden","Dettes à l'égard d'entreprises avec lesquelles l'entreprise est liée par des participations payables à plus d'un an"
"account_group_45221","45221","","Purchases and services","Käufe und Dienstleistungen","Achats et prestations de services"
"account_group_45222","45222","","Loans and advances","Ausleihungen und geleistete Anzahlungen","Prêts et avances"
"account_group_45228","45228","","Other payables","Sonstige Verbindlichkeiten","Autres dettes"
"account_group_46","46","","Tax and social security debts","Schulden bei Steuern und Sozialversicherung","Dettes fiscales et sociales"
"account_group_461","461","","Tax debts","Steuerschulden","Dettes fiscales"
"account_group_4611","4611","","Municipal authorities","Gemeindeverwaltungen","Administrations communales"
"account_group_4612","4612","","Direct Tax Authority (ACD)","Direkte Steuerbehörde (ACD)","Autorité des impôts directs (ACD)"
"account_group_46121","46121","","Corporate income tax (CIT)","Körperschaftssteuer (CIT)","Impôt sur les sociétés (IS)"
"account_group_461211","461211","","Corporate income tax - Tax accrual","Körperschaftssteuer - Ermittelte Steuerschuld","IRC – charge fiscale estimée"
"account_group_461212","461212","","CIT - Tax payable","Körperschaftssteuer - Zu zahlende Steuerschuld","IRC – dette fiscale à payer"
"account_group_46122","46122","","Municipal business tax (MBT)","Gewerbesteuer","Impôt commercial communal (ICC)"
"account_group_461221","461221","","MBT - Tax accrual","Gewerbesteuer - Ermittelte Steuerschuld","ICC – charge fiscale estimée"
"account_group_461222","461222","","MBT - Tax payable","Gewerbesteuer - Zu zahlende Steuerschuld","ICC – dette fiscale à payer"
"account_group_46123","46123","","Net wealth tax (NWT)","Netto-Vermögensteuer (NWT)","Impôt sur le patrimoine net (ISN)"
"account_group_461231","461231","","NWT - Tax accrual","Vermögensteuer - Ermittelte Steuerschuld","IF – charge fiscale estimée"
"account_group_461232","461232","","NWT - Tax payable","Vermögensteuer - Zu zahlende Steuerschuld","IF – dette fiscale à payer"
"account_group_46124","46124","","Withholding tax on wages and salaries","Einbehaltene Steuer auf Gehälter und Löhne","Retenue d'impôt sur traitements et salaires (RTS)"
"account_group_46125","46125","","Withholding tax on financial investment income","Einbehaltene Kapitalertragsteuer","Retenue d'impôt sur revenus de capitaux mobiliers"
"account_group_46126","46126","","Withholding tax on director's fees","Einbehaltene Steuer auf Tantiemen","Retenue d'impôt sur les tantièmes"
"account_group_46128","46128","","ACD - Other amounts payable","ACD - Sonstige Verbindlichkeiten","ACD – Autres dettes"
"account_group_4613","4613","","Customs and Excise Authority (ADA)","Zoll- und Verbrauchssteuerverwaltung (ADA)","Administration des Douanes et Accises (ADA)"
"account_group_4614","4614","","Indirect tax authorities (AED)","Indirekte Steuerbehörden (AED)","Administrations fiscales indirectes (AED)"
"account_group_46141","46141","","Value-added tax (VAT)","Mehrwertsteuer (VAT)","Taxe sur la valeur ajoutée (TVA)"
"account_group_461411","461411","","VAT received","Umsatzsteuer","TVA en aval"
"account_group_461412","461412","","VAT payable","Zu zahlende MwSt","TVA à payer"
"account_group_461413","461413","","VAT down payments received","Erhaltene Anzahlungen auf MwSt","TVA acomptes reçus"
"account_group_461418","461418","","VAT - Other payables","MwSt - Sonstige Verbindlichkeiten","TVA – Autres dettes"
"account_group_46142","46142","","Indirect taxes","Indirekte Steuern","Impôts indirects"
"account_group_461421","461421","","Registration duties","Registriergebühren","Droits d'enregistrement"
"account_group_461422","461422","","Subscription tax","Abgeltungssteuer","Taxe d'abonnement"
"account_group_461428","461428","","Other indirect taxes","Sonstige indirekte Steuern","Autres impôts indirects"
"account_group_46148","46148","","AED - Other debts","Steuerverwaltung - Indirekte Steuern (AED) - Sonstige Verbindlichkeiten","AED - Autres dettes"
"account_group_4615","4615","","Foreign tax authorities","Ausländische Steuerbehörden","Autorités fiscales étrangères"
"account_group_46151","46151","","Foreign VAT","Ausländische MwSt","TVA étrangères"
"account_group_46158","46158","","Other foreign taxes","Sonstige ausländische Steuern","Autres impôts étrangers"
"account_group_462","462","","Social security debts and other social securities offices","Schulden bei der Sozialversicherung und anderen Sozialversicherungsträgern","Dettes de sécurité sociale et autres organismes de sécurité sociale"
"account_group_4621","4621","","Joint Social Security Centre (CCSS)","Gemeinsames Zentrum für soziale Sicherheit (CCSS)","Centre commun de la sécurité sociale (CCSS)"
"account_group_4622","4622","","Foreign Social Security offices","Ausländische Sozialversicherungen","Organismes étrangers de sécurité sociale"
"account_group_4628","4628","","Other social bodies","Sonstige Einrichtungen der sozialen Sicherheit","Autres organismes sociaux"
"account_group_47","47","","Other debts","Sonstige Schulden","Autres dettes"
"account_group_471","471","","Other debts payable within one year","Sonstige Schulden, die innerhalb eines Jahres fällig sind","Autres dettes à moins d'un an"
"account_group_4711","4711","","Received deposits and guarantees","Erhaltene Hinterlegungen und Kautionen","Dépôts et cautionnements reçus"
"account_group_4712","4712","","Amounts payable to partners and shareholders (others than from affiliated undertakings)","Verbindlichkeiten gegenüber Gesellschaftern und Aktionären (andere als gegnüber verbundenen Unternehmen)","Dettes envers associés et actionnaires (autres qu'entreprises liées)"
"account_group_4713","4713","","Amounts payable to directors, managers, statutory auditors and similar","Verbindlichkeiten gegenüber Verwaltern, Geschäftsführern, Kommissaren und ähnlichen","Dettes envers administrateurs, gérants, commissaires et organes assimilés"
"account_group_4714","4714","","Amounts payable to staff","Verbindlichkeiten gegenüber dem Personal","Dettes envers le personnel"
"account_group_4715","4715","","State - Greenhous gas and similar emission quotas to be returned or acquired","Staat - zurückzugebende oder zu erwerbende Kontigente für Treibhausgasemissionen und ähnliche","Etat – Quotas d'émission de gaz à effet de serre et assimilés à restituer ou à acquérir"
"account_group_4716","4716","","Loans and similar debts","Darlehen und ähnliche Schulden","Emprunts et dettes assimilées"
"account_group_47161","47161","","Other loans","Sonstige Ausleihungen","Autres emprunts"
"account_group_47162","47162","","Lease debts","Verbindlichkeiten aus Finanzierungsleasing","Dettes de leasing"
"account_group_47163","47163","","Life annuities","Leibrenten","Rentes viagères"
"account_group_47168","47168","","Other similar debts","Sonstige vergleichbare Verbindlichkeiten","Autres dettes assimilées"
"account_group_4717","4717","","Derivative financial instruments","Derivative Finanzinstrumente","Instruments financiers dérivés"
"account_group_4718","4718","","Other miscellaneous debts","Andere sonstige Verbindlichkeiten","Autres dettes diverses"
"account_group_472","472","","Other debts payable after more than one year","Sonstige Schulden mit einer Laufzeit von mehr als einem Jahr","Autres dettes à plus d'un an"
"account_group_4721","4721","","Received deposits and guarantees","Erhaltene Hinterlegungen und Kautionen","Dépôts et cautionnements reçus"
"account_group_4722","4722","","Amounts payable to partners and shareholders (others than from affiliated undertakings)","Verbindlichkeiten gegenüber Gesellschaftern und Aktionären (andere als gegnüber verbundenen Unternehmen)","Dettes envers associés et actionnaires (autres qu'entreprises liées)"
"account_group_4723","4723","","Amounts payable to directors, managers, statutory auditors and similar","Verbindlichkeiten gegenüber Verwaltern, Geschäftsführern, Kommissaren und ähnlichen","Dettes envers administrateurs, gérants, commissaires et organes assimilés"
"account_group_4724","4724","","Amounts payable to staff","Verbindlichkeiten gegenüber dem Personal","Dettes envers le personnel"
"account_group_4726","4726","","Loans and similar debts","Darlehen und ähnliche Schulden","Emprunts et dettes assimilées"
"account_group_47261","47261","","Other loans","Sonstige Ausleihungen","Autres emprunts"
"account_group_47262","47262","","Lease debts","Verbindlichkeiten aus Finanzierungsleasing","Dettes de leasing"
"account_group_47263","47263","","Life annuities","Leibrenten","Rentes viagères"
"account_group_47268","47268","","Other similar debts","Sonstige vergleichbare Verbindlichkeiten","Autres dettes assimilées"
"account_group_4727","4727","","Derivative financial instruments","Derivative Finanzinstrumente","Instruments financiers dérivés"
"account_group_4728","4728","","Other miscellaneous debts","Andere sonstige Verbindlichkeiten","Autres dettes diverses"
"account_group_48","48","","Deferred charges and income","Rechnungsabgrenzungsposten und Einnahmen","Charges et produits différés"
"account_group_481","481","","Deferred charges (on one or more financial years)","Abgegrenzte Aufwendungen (über ein oder mehrere Geschäfstjahre)","Charges à reporter (sur un ou plusieurs exercices)"
"account_group_482","482","","Deferred income (on one or more financial years)","Abgegrenzte Erträge (über ein oder mehrere Geschäfstjahre)","Produits à reporter (sur un ou plusieurs exercices)"
"account_group_483","483","","State - Greenhous gas and similar emission quotas received","Staat - Erhaltene Quoten für Treibhausgas und ähnliche Emissionen","État - Quotas d'émission de gaz à effet de serre et similaires reçus"
"account_group_484","484","","Transitory or suspense accounts - Assets","Transitorische Konten - Aktiva","Comptes transitoires ou d'attente – Actif"
"account_group_485","485","","Transitory or suspense accounts - Liabilities","Transitorische Konten - Passiva","Comptes transitoires ou d'attente – Passif"
"account_group_486","486","","Linking accounts (branches) - Assets","Verbindungskonten (Niederlassungen) - Aktiva","Comptes de liaison (succursales) – Actif"
"account_group_487","487","","Linking accounts (branches) - Liabilities","Verbindungskonten (Niederlassungen) - Passiva","Comptes de liaison (succursales) – Passif"
"account_group_5","5","","FINANCIAL ACCOUNTS","FINANZKONTEN","COMPTES FINANCIERS"
"account_group_50","50","","Transferable securities","Übertragbare Wertpapiere","Valeurs mobilières"
"account_group_501","501","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_502","502","","Own shares or own corporate units","Eigene Aktien oder eigene Anteile","Actions propres ou parts propres"
"account_group_503","503","","Shares in undertakings with which the undertaking is linked by virtue of participating interests","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_508","508","","Other transferable securities","Sonstige Wertpapiere","Autres valeurs mobilières"
"account_group_5081","5081","","Shares - listed securities","Aktien - Notierte Wertpapiere","Actions – Titres cotés"
"account_group_5082","5082","","Shares - unlisted securities","Aktien - Nicht notierte Wertpapiere","Actions – Titres non cotés"
"account_group_5083","5083","","Debenture loans and other notes issued and repurchased by the company","Eigene Anleihen und sonstige eigene Schuldtitel","Obligations et autres titres de créance émis par l'entreprise et rachetés par elle"
"account_group_5084","5084","","Listed debenture loans","Anleihen - Notierte Wertpapiere","Obligations – Titres cotés"
"account_group_5085","5085","","Unlisted debenture loans","Anleihen - Nicht notierte Wertpapiere","Obligations – Titres non cotés"
"account_group_5088","5088","","Other miscellaneous transferable securities","Andere sonstige Wertpapiere","Autres valeurs mobilières diverses"
"account_group_51","51","","Cash at bank, in postal cheques accounts, cheques and in hand","Bankguthaben, Postcheckguthaben, Schecks und Kassenbestand","Liquidités en banque, sur les comptes de chèques postaux, sur les chèques et en caisse"
"account_group_513","513","","Banks and postal cheques accounts (CCP)","Bank- und Postscheckkonten (CCP)","Comptes bancaires et chèques postaux (CCP)"
"account_group_5131","5131","","Banks and CCP : available balance","Kreditinstitute und CCP : Guthaben","Banques et CCP : avoirs"
"account_group_5132","5132","","Banks and CCP : overdraft","Kreditinstitute und CCP : Überziehungen","Banques et CCP : découverts"
"account_group_516","516","","Cash in hand","Kassenbestand","Caisse"
"account_group_517","517","","Internal transfers","Interne Transfers","Transferts internes"
"account_group_5171","5171","","Internal transfers : debit balance","Interne Transferkonten : Sollsaldo","Virements internes : solde débiteur"
"account_group_5172","5172","","Internal transfers : credit balance","Interne Transferkonten : Guthaben","Virements internes : solde créditeur"
"account_group_518","518","","Other cash amounts","Sonstige Guthaben","Autres avoirs"
"account_group_6","6","","CHARGES ACCOUNTS","SPESENKONTEN","COMPTES DE CHARGES"
"account_group_60","60","","Use of merchandise, raw and consumable materials","Einsatz von Handelswaren, Roh- und Verbrauchsmaterialien","Utilisation de marchandises, de matières premières et de consommables"
"account_group_601","601","","Purchases of raw materials","Einkäufe von Rohstoffen","Achats de matières premières"
"account_group_603","603","","Purchases of consumable materials and supplies","Kauf von Verbrauchsmaterial und Verbrauchsgütern","Achats de matériaux et de fournitures consommables"
"account_group_6031","6031","","Fuels, gas, water and electricity","Brennstoffe, Gas, Wasser und Strom","Combustibles, gaz, eau et électricité"
"account_group_60311","60311","","Solid fuels","Feste Brennstoffe","Combustibles solides"
"account_group_60312","60312","","Liquid fuels","Flüssige Brennstoffe","Combustibles liquides"
"account_group_60313","60313","","Gas","Gas","Gaz"
"account_group_60314","60314","","Water and sewage","Wasser und Abwasser","Eau et eaux usées"
"account_group_60315","60315","","Electricity","Strom","Electricité"
"account_group_6032","6032","","Maintenance supplies","Pflegemittel","Produits d'entretien"
"account_group_6033","6033","","Workshop, factory and store supplies and small equipment","Werkstatt-, Fabrik- und Ladenausstattung","Fournitures et petit équipement d'atelier, d'usine et de magasin"
"account_group_6034","6034","","Work clothes","Berufsbekleidung","Vêtements professionnels"
"account_group_6035","6035","","Office and administrative supplies","Büroausstattung","Fournitures administratives et de bureau"
"account_group_6036","6036","","Motor fuels","Treibstoffe","Carburants"
"account_group_6037","6037","","Lubricants","Schmiermittel","Lubrifiants"
"account_group_6038","6038","","Other consumable supplies","Sonstige Betriebsstoffe","Autres fournitures consommables"
"account_group_604","604","","Purchases of packaging","Einkäufe von Verpackungen","Achats d'emballages"
"account_group_606","606","","Purchases of merchandise and other goods for resale","Kauf von Handelswaren und sonstigen Waren zum Wiederverkauf","Achats de marchandises et d'autres biens destinés à la revente"
"account_group_6061","6061","","Purchases of merchandise","Einkäufe von Waren","Achats de marchandises"
"account_group_6062","6062","","Purchases of land for resale","Einkäufe von zum Verkauf bestimmten Grundstücken","Achats de terrains destinés à la revente"
"account_group_6063","6063","","Purchases of buildings for resale","Einkäufe von zum Verkauf bestimmten Bauten/Gebäuden","Achats d'immeubles destinés à la revente"
"account_group_607","607","","Changes in inventory","Veränderungen im Bestand","Variations des stocks"
"account_group_6071","6071","","Changes in inventory of raw materials","Bestandsveränderungen an Rohstoffen","Variation des stocks de matières premières"
"account_group_6073","6073","","Changes in inventory of consumable materials and supplies","Bestandsveränderungen an Hilfs- und Betriebsstoffen","Variation des stocks de matières et fournitures consommables"
"account_group_6074","6074","","Changes in inventory of packaging","Bestandsveränderungen an Verpackungen","Variation des stocks d'emballages"
"account_group_6076","6076","","Changes in inventory of merchandise and other goods for resale","Veränderung der Vorräte an Handelswaren und sonstigen Waren zum Wiederverkauf","Variation des stocks de marchandises et autres biens destinés à la revente"
"account_group_60761","60761","","Merchandise","Waren","Marchandises"
"account_group_60762","60762","","Land for resale","Zum Verkauf bestimme Grundstücke","Terrains destinés à la revente"
"account_group_60763","60763","","Buildings for resale","Zum Verkauf bestimmte Bauten/Gebäude","Immeubles destinés à la revente"
"account_group_608","608","","Purchases of items included in the production of goods and services","Käufe von Gegenständen, die zur Produktion von Waren und Dienstleistungen gehören","Achats d'articles entrant dans la production de biens et de services"
"account_group_6081","6081","","Services included in the production of goods and services","Dienstleistungen, die in der Produktion von Waren und Dienstleistungen enthalten sind","Services inclus dans la production de biens et de services"
"account_group_60811","60811","","Tailoring","Lohnverarbeitung","Travail à façon"
"account_group_60812","60812","","Research and development","Forschung und Entwicklung","Recherche et développement"
"account_group_60813","60813","","Architects' and engineers' fees","Honorare für Architekten und Ingenieure","Frais d'architectes et d'ingénieurs"
"account_group_60814","60814","","Outsourcing included in the production of goods and services","Zulieferungen für Werklieferungen und -leistungen","Sous-traitance incorporée aux ouvrages et produits"
"account_group_6082","6082","","Other purchases of material included in the production of goods and services","Sonstige Einkäufe von Material, Ausstattungen, Ersatzteilen und Arbeiten für Werklieferungen und -leistungen","Autres achats de matériel incorporés aux ouvrages et produits"
"account_group_6083","6083","","Purchase of greenhous gas and similar emission quotas","Kauf von Treibhausgas- und ähnlichen Emissionsquoten","Achat de quotas d'émission de gaz à effet de serre et similaires"
"account_group_6088","6088","","Other purchases included in the production of goods and services","Sonstige bezogene Gutachten und Dienstleistungen","Autres achats incorporés aux ouvrages et produits"
"account_group_609","609","","Rebates, discounts and refunds (RDR) received and not directly deducted from purchases","Erhaltene Rabatte, Skonti und Rückerstattungen (RDR), die nicht direkt von den Käufen abgezogen werden","Rabais, remises et remboursements (RDR) reçus et non directement déduits des achats"
"account_group_6091","6091","","RDR on purchases of raw materials","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Rohstoffe","RRR sur achats de matières premières"
"account_group_6093","6093","","RDR on purchases of consumable materials and supplies","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Hilfs- und Betriebsstoffe","RRR sur achats de matières et fournitures consommables"
"account_group_6094","6094","","RDR on purchases of packaging","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Verpackungen","RRR sur achats d'emballages"
"account_group_6096","6096","","RDR on purchases of merchandise and other goods for resale","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Waren und sonstige zum Verkauf bestimmte Güter/Vermögensgegenstände","RRR sur achats de marchandises et d'autres biens destinés à la revente"
"account_group_6098","6098","","RDR on purchases included in the production of goods and services","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf Einkäufe für Werklieferungen und -leistungen","RRR sur achats incorporés aux ouvrages et produits"
"account_group_6099","6099","","Unallocated RDR","Nicht zugeordnete Rabatte, Preisnachlässe und Rückvergütungen","RRR non affectés"
"account_group_61","61","","Other external charges","Sonstige externe Kosten","Autres charges externes"
"account_group_611","611","","Rents and service charges","Mieten und Nebenkosten","Loyers et charges"
"account_group_6111","6111","","Rents and operationnal leasing for real property","Mieten und operatives Leasing für Grundbesitz","Loyers et leasing opérationnel pour les biens immobiliers"
"account_group_61111","61111","","Land","Grundstücke","Terrains"
"account_group_61112","61112","","Buildings","Bauten/Gebäude","Constructions / Bâtiments"
"account_group_6112","6112","","Rents and operational leasing on movable property","Mieten und operatives Leasing von beweglichen Sachen","Loyers et leasing opérationnel sur biens meubles"
"account_group_61123","61123","","Rolling stock","Fuhrpark","Matériel roulant"
"account_group_61128","61128","","Other","Sonstige","Autres"
"account_group_6113","6113","","Service charges and co-ownership expenses","Mietnebenkosten und Miteigentumsgemeinschaftskosten","Charges locatives et de copropriété"
"account_group_6114","6114","","Financial leasing on real property","Immobilienfinanzierungsleasing","Leasing financier immobilier"
"account_group_6115","6115","","Financial leasing on movable property","Finanzierungsleasing von beweglichen Sachen","Crédit-bail mobilier"
"account_group_61153","61153","","Rolling stock","Fuhrpark","Matériel roulant"
"account_group_61158","61158","","Other","Sonstige","Autres"
"account_group_612","612","","Subcontracting, servicing, repairs and maintenance","Vergabe von Unteraufträgen, Wartung, Reparatur und Instandhaltung","Sous-traitance, entretien, réparation et maintenance"
"account_group_6121","6121","","General subcontracting (not included in the production of goods and services)","Allgemeine Zulieferung (nicht für Werklieferungen und -leistungen und sonstige Arbeiten)","Sous-traitance générale (non incorporée aux ouvrages et produits)"
"account_group_6122","6122","","Servicing, repairs and maintenance","Wartung, Reparatur und Instandhaltung","Entretien, réparation et maintenance"
"account_group_61221","61221","","Buildings","Bauten/Gebäude","Constructions / Bâtiments"
"account_group_61223","61223","","Rolling stock","Fuhrpark","Matériel roulant"
"account_group_61228","61228","","Other","Sonstige","Autres"
"account_group_613","613","","Remuneration of intermediaries and professional fees","Vergütungen für Vermittler und Honorare","Rémunération des intermédiaires et honoraires"
"account_group_6131","6131","","Commissions and brokerage fees","Provisionen und Maklergebühren","Commissions et courtages"
"account_group_6132","6132","","IT services","IT - Instandhaltung","Services informatiques"
"account_group_6133","6133","","Banking and similar services","Bank- und ähnliche Dienstleistungen","Services bancaires et similaires"
"account_group_61332","61332","","Loans' issuance expenses","Emissionskosten von Anleihen","Frais sur émission d'emprunts"
"account_group_61333","61333","","Bank account charges and bank commissions (included custody fees on securities)","Kontogebühren und Bankkommissionen (Verwahrungsrechte von Wertpapieren inbegriffen)","Frais de comptes et commissions bancaires (y compris droits de garde sur titres)"
"account_group_61334","61334","","Charges for electronic means of paiment","Entgelte für elektronische Zahlungsmittel","Frais pour les moyens électroniques de paiement"
"account_group_61336","61336","","Factoring services","Factoringgebühren","Rémunérations d'affacturage"
"account_group_61338","61338","","Other banking and similar services (except interest and similar expenses)","Sonstige Bankdienstleistungen (außer Zinsen und vergleichbare Kosten)","Autres services bancaires et assimilés (hors intérêts et frais assimilés)"
"account_group_6134","6134","","Professional fees","Honorare","Honoraires professionnels"
"account_group_61341","61341","","Legal, litigation and similar fees","Rechts- und Prozesskosten und ähnliche","Honoraires juridiques, de contentieux et assimilés"
"account_group_61342","61342","","Accounting, tax consulting, auditing and similar fees","Buchführungs-, Steuerberatungs- und Prüfungskosten und ähnliche","Honoraires comptables, fiscaux, d'audit et assimilés"
"account_group_61348","61348","","Other professional fees","Sonstige Honorare","Autres honoraires"
"account_group_6135","6135","","Notarial and similar fees","Notarielle Beurkundungskosten und ähnliche","Frais d'actes notariés et assimilés"
"account_group_6138","6138","","Other remuneration of intermediaries and professional fees","Sonstige Vermittlervergütungen und Honorare","Autres rémunérations d'intermédiaires et honoraires"
"account_group_614","614","","Insurance premiums","Versicherungsprämien","Primes d'assurance"
"account_group_6141","6141","","Insurance for assets","Versicherung für Vermögenswerte","Assurance des actifs"
"account_group_61411","61411","","Buildings","Bauten/Gebäude","Constructions / Bâtiments"
"account_group_61412","61412","","Rolling stock","Fuhrpark","Matériel roulant"
"account_group_61418","61418","","Other","Sonstige","Autres"
"account_group_6142","6142","","Insurance on rented assets","Versicherung für gemietete Vermögensgegenstände","Assurance sur biens pris en location"
"account_group_6143","6143","","Transport insurance","Transportversicherung","Assurance-transport"
"account_group_6144","6144","","Business risk insurance","Versicherung für betriebliche Risiken","Assurance risque d'exploitation"
"account_group_6145","6145","","Customers credit insurance","Forderungsausfallversicherung","Assurance insolvabilité clients"
"account_group_6146","6146","","Third-party insurance","Haftpflichtversicherung","Assurance responsabilité civile"
"account_group_6148","6148","","Other insurances","Sonstige Versicherungen","Autres assurances"
"account_group_615","615","","Marketing and communication costs","Marketing- und Kommunikationskosten","Frais de marketing et de communication"
"account_group_6151","6151","","Marketing and advertising costs","Marketing- und Werbekosten","Frais de marketing et de publicité"
"account_group_61511","61511","","Press advertising","Annoncen und Inserate","Annonces et insertions"
"account_group_61512","61512","","Samples","Muster","Echantillons"
"account_group_61513","61513","","Fairs and exhibitions","Messen und Ausstellungen","Foires et expositions"
"account_group_61514","61514","","Gifts to customers","Geschenke für Kunden","Cadeaux à la clientèle"
"account_group_61515","61515","","Catalogues, printed materials and publications","Kataloge, Drucksachen und Veröffentlichungen","Catalogues et imprimés et publications"
"account_group_61516","61516","","Donations","Spenden","Dons courants"
"account_group_61517","61517","","Sponsorship","Sponsoring","Sponsoring"
"account_group_61518","61518","","Other purchases of advertising services","Sonstige Werbung","Autres achats de services publicitaires"
"account_group_6152","6152","","Travel and entertainment expenses","Reise- und Bewirtungskosten","Frais de voyage et de représentation"
"account_group_61521","61521","","Travel expenses","Reisekosten","Frais de voyage"
"account_group_615211","615211","","Management (if appropriate owner and partner)","Management (ggf. Eigentümer und Partner)","Direction (le cas échéant, propriétaire et partenaire)"
"account_group_615212","615212","","Staff","Personal","Personnel"
"account_group_61522","61522","","Relocation expenses","Umzugskosten des Unternehmens","Frais de déménagement de l'entreprise"
"account_group_61523","61523","","Business assignments","Dienstreisen","Missions"
"account_group_61524","61524","","Receptions and entertainment costs","Bewirtungs- und Repräsentationskosten","Réceptions et frais de représentation"
"account_group_6153","6153","","Postal charges and telecommunication costs","Postgebühren und Telekommunikationskosten","Frais postaux et de télécommunication"
"account_group_61531","61531","","Postal charges","Postgebühren","Frais postaux"
"account_group_61532","61532","","Telecommunication costs","Sonstige Telekommunikationskosten","Frais de télécommunication"
"account_group_616","616","","Transportation of goods and collective staff transportation","Transport von Gütern und Sammeltransport von Personal","Transport de marchandises et transport collectif de personnel"
"account_group_6161","6161","","Transportation of purchased goods","Transporte von Einkäufen","Transports sur achats"
"account_group_6162","6162","","Transportation of sold goods","Transporte von Verkäufen","Transports sur ventes"
"account_group_6165","6165","","Collective staff transportation","Beförderungen von Personal","Transports collectifs du personnel"
"account_group_6168","6168","","Other transportation","Sonstige Transporte","Autres transports"
"account_group_617","617","","External staff of the company","Externes Personal des Unternehmens","Personnel externe de l'entreprise"
"account_group_6171","6171","","Temporary staff","Aushilfskräfte","Personnel intérimaire"
"account_group_6172","6172","","External staff on secondment","Ausgeliehenes Personal","Personnel prêté à l'entreprise"
"account_group_618","618","","Miscellaneous external charges","Verschiedene externe Kosten","Diverses charges externes"
"account_group_6181","6181","","Documentation","Dokumentation",""
"account_group_6182","6182","","Costs of training, symposiums, seminars, conferences","Kosten von Weiterbildungen, Kolloquien, Seminaren, Konferenzen","Frais de formation, colloques, séminaires, conférences"
"account_group_6183","6183","","Industrial and non-industrial waste treatment","Abfallbeseitigung von Industrieabfällen und sonstigen Abfällen","Elimination des déchets industriels et non industriels"
"account_group_6184","6184","","Fuels, gas, water and electricity (not included in the production of goods and services)","Brennstoffe, Gas, Wasser und Strom (nicht in der Produktion von Waren und Dienstleistungen enthalten)","Combustibles, gaz, eau et électricité (non inclus dans la production de biens et services)"
"account_group_61841","61841","","Solid fuels","Feste Brennstoffe","Combustibles solides"
"account_group_61842","61842","","Liquid fuels (oil, motor fuel, etc.)","Flüssige Brennsoffe","Combustibles liquides (mazout, carburants, etc.)"
"account_group_61843","61843","","Gas","Gas","Gaz"
"account_group_61844","61844","","Water and waste water","Wasserversorgung und Abwasserbeseitigung","Eau et eaux usées"
"account_group_61845","61845","","Electricity","Strom","Electricité"
"account_group_6185","6185","","Supplies and small equipment","Verbrauchsmaterial und Kleingeräte","Fournitures et petit matériel"
"account_group_61851","61851","","Office supplies","Büromaterial","Fournitures administratives et de bureau"
"account_group_61852","61852","","Small equipment","Kleines Werkzeug","Petit équipement"
"account_group_61853","61853","","Work clothes","Berufsbekleidung","Vêtements professionnels"
"account_group_61854","61854","","Maintenance supplies","Pflegemittel","Produits d'entretien"
"account_group_61858","61858","","Other","Sonstige","Autres"
"account_group_6186","6186","","Surveillance and security charges","Wach- und Sicherheitsdienste","Frais de surveillance et de gardiennage"
"account_group_6187","6187","","Contributions to professional associations","Beiträge für Berufsverbände","Cotisations aux associations professionnelles"
"account_group_6188","6188","","Other miscellaneous external charges","Andere sonstige externe Aufwendungen","Autres charges externes diverses"
"account_group_619","619","","Rebates, discounts and refunds received on other external charges","Erhaltene Rabatte, Preisnachlässe und Rückvergütungen auf sonstige externe Aufwendungen","Rabais, remises et ristournes (RRR) obtenus et non directement déduits des autres charges externes"
"account_group_62","62","","Staff expenses","Personalausgaben","Dépenses de personnel"
"account_group_621","621","","Staff remuneration","Vergütung des Personals","Rémunération du personnel"
"account_group_6211","6211","","Gross wages","Bruttolöhne","Salaires bruts"
"account_group_62111","62111","","Base wages","Grundlöhne","Salaires de base"
"account_group_62112","62112","","Wage supplements","Lohnzuschläge","Compléments de salaire"
"account_group_621121","621121","","Sunday","Sonntagsarbeit","Dimanche"
"account_group_621122","621122","","Public holidays","Feiertagsarbeit","Jours fériés légaux"
"account_group_621123","621123","","Overtime","Überstunden","Heures supplémentaires"
"account_group_621128","621128","","Other supplements","Sonstige Zuschläge","Autres suppléments"
"account_group_62114","62114","","Incentives, bonuses and commissions","Gratifikationen, Prämien und Provisionen","Gratifications, primes et commissions"
"account_group_62115","62115","","Benefits in kind","Sachleistungen","Avantages en nature"
"account_group_62116","62116","","Severance pay","Abfindungen","Indemnités de licenciement"
"account_group_62117","62117","","Survivor's pay","Hinterbliebenenzuschuss","Trimestre de faveur"
"account_group_6218","6218","","Other benefits","Sonstige Zulagen","Autres avantages"
"account_group_6219","6219","","Refunds on wages paid","Erstattungen von gezahlten Löhnen","Remboursements sur salaires"
"account_group_622","622","","Other staff remuneration","Sonstige Bezüge des Personals","Autres rémunérations du personnel"
"account_group_6221","6221","","Students","Studentische Aushilfskräfte","Etudiants"
"account_group_6222","6222","","Casual workers","Gelegenheitsarbeitskräfte","Salaires occasionnels"
"account_group_6228","6228","","Other","Sonstige","Autres"
"account_group_623","623","","Social security costs (employer's share)","Sozialversicherungskosten (Arbeitgeberanteil)","Coûts de la sécurité sociale (part de l'employeur)"
"account_group_6231","6231","","Social security on pensions","Soziale Aufwendungen für Renten","Charges sociales couvrant les pensions"
"account_group_6232","6232","","Other social security costs (including illness, accidents, a.s.o.)","Sonstige soziale Aufwendungen (einschließlich für Krankheits- und Arbeitsunfallversicherung)","Autres charges sociales (y inclus maladie, accident, etc.)"
"account_group_624","624","","Other staff expenses","Sonstige Personalausgaben","Autres dépenses de personnel"
"account_group_6241","6241","","Complementary pensions","Komplementäre Renten","Pensions complémentaires"
"account_group_62411","62411","","Premiums for external pensions funds","Beiträge für externe Rentenfonds","Primes à des fonds de pensions extérieurs"
"account_group_62412","62412","","Changes to provisions for complementary pensions","Zuführung zu Rückstellungen für Zusatzrenten","Variations sur provisions pour pensions complémentaires"
"account_group_62413","62413","","Withholding tax on complementary pensions","Einbehaltene Steuer auf Zusatzrenten","Retenue d'impôt sur pension complémentaire"
"account_group_62414","62414","","Insolvency insurance premiums","Beiträge zur Insolvenzversicherung","Prime d'assurance insolvabilité"
"account_group_62415","62415","","Complementary pensions paid by the employer","Ausgezahlte betriebliche Zusatzrente","Pensions complémentaires versées par l'employeur"
"account_group_6248","6248","","Other staff expenses not mentioned above","Sonstige nicht oben aufgeführte Personalaufwendungen","Autres frais de personnel non visés ci-dessus"
"account_group_63","63","","Allocations to value adjustments (AVA) and fair value adjustments (FVA) on formation expenses, intangible, tangible and current assets (except transferable securities)","Zuweisungen zu Wertberichtigungen (AVA) und Fair-Value-Anpassungen (FVA) auf Gründungskosten, immaterielle Vermögenswerte, Sachanlagen und Umlaufvermögen (außer Wertpapieren)","Allocations aux corrections de valeur (AVA) et aux corrections de juste valeur (FVA) sur les frais d'établissement, les immobilisations incorporelles, corporelles et les actifs circulants (à l'exception des valeurs mobilières)"
"account_group_631","631","","AVA on formation expenses and similar expenses","ZWb auf Gründungskosten und ähnliche Kosten","DCV sur frais d'établissement et frais assimilés"
"account_group_6311","6311","","AVA on set-up and start-up costs","ZWb von Gründungs- und Errichtungskosten (Rechts- und Beratungskosten)","DCV sur frais de constitution et de premier établissement"
"account_group_6313","6313","","AVA on expenses for capital increases and various operations (mergers, demergers, changes of legal form)","ZWb von Kosten für Kapitalerhöhung und anderen Vorgängen (Verschmelzungen, Spaltungen und Umwandlungen)","DCV sur frais d'augmentation de capital et d'opérations diverses (fusions, scissions, transformations)"
"account_group_6314","6314","","AVA on loan-issuance expenses","ZWb von Emissionskosten von Anleihen","DCV sur frais d'émission d'emprunts"
"account_group_6318","6318","","AVA on other similar expenses","ZWb von sonstigen vergleichbaren Kosten","DCV sur autres frais assimilés"
"account_group_632","632","","AVA on intangible fixed assets","ZWb auf immaterielle Anlagewerte","DCV sur immobilisations incorporelles"
"account_group_6321","6321","","AVA on development costs","ZWb von Entwicklungskosten","DCV sur frais de développement"
"account_group_6322","6322","","AVA on concessions, patents, licences, trademarks and similar rights and assets","ZWb von Konzessionen, Patenten, Lizenzen, Warenzeichen und vergleichbaren Rechten und Werten","DCV sur concessions, brevets, licences, marques ainsi que droits et valeurs similaires"
"account_group_6323","6323","","AVA on goodwill acquired for consideration","ZWb vom Geschäfts- oder Firmenwert, soweit er entgeltlich erworben wurde","DCV sur fonds de commerce dans la mesure où il a été acquis à titre onéreux"
"account_group_6324","6324","","AVA on down payments and intangible fixed assets under development","ZWb von geleisteten Anzahlungen und immateriellen Vermögensgegenständen in Entwicklung","DCV sur acomptes versés et immobilisations incorporelles en cours"
"account_group_633","633","","AVA on tangible fixed assets and fair value adjustments (FVA) on investment properties","ZWb auf Sachanlagen und Fair-Value-Anpassungen (FVA) auf als Finanzinvestition gehaltene Immobilien","DCV sur immobilisations corporelles et aux corrections de juste valeur (FVA) sur immeubles de placement"
"account_group_6331","6331","","AVA on land, fittings-out and buildings and FVA on investment properties","ZWb auf Grundstücke, Ausstattungen und Gebäude und FVA auf Anlageimmobilien","DCV sur terrains, aménagements et constructions et FVA sur immeubles de placement"
"account_group_63311","63311","","AVA on land","ZWb von Grundstücken","DCV sur terrains"
"account_group_63312","63312","","AVA on fixtures and fittings-out of land","ZWb von Erschließungen von Grundstücken","DCV sur agencements et aménagements de terrains"
"account_group_63313","63313","","AVA on buildings","ZWb von Bauten / Gebäuden","DCV sur constructions / bâtiments"
"account_group_63314","63314","","AVA on fixtures and fittings-out of buildings","ZWb von Einrichtungen von Bauten/Gebäuden","DCV sur agencements et aménagements de constructions / bâtiments"
"account_group_63315","63315","","FVA on investment properties","AFV von Anlageimmobilien","AJV sur immeubles de placement"
"account_group_6332","6332","","AVA on plant and machinery","ZWb von technischen Anlagen und Maschinen","DCV sur installations techniques et machines"
"account_group_6333","6333","","AVA on other fixtures and fittings, tools and equipment (including rolling stock)","ZWb von anderen Anlagen, Betriebs- und Geschäftsausstattung und Fuhrpark","DCV sur autres installations, outillage et mobilier (y compris matériel roulant)"
"account_group_6334","6334","","AVA on down payments and tangible fixed assets under development","ZWb von geleisteten Anzahlungen und Anlagen im Bau","DCV sur acomptes versés et immobilisations corporelles en cours"
"account_group_634","634","","AVA on inventories","ZWb auf Vorräte","DCV sur les stocks"
"account_group_6341","6341","","AVA on inventories of raw materials and consumables","ZWb von Roh-, Hilfs- und Betriebsstoffen","DCV sur stocks de matières premières et consommables"
"account_group_6342","6342","","AVA on inventories of work and contracts in progress","ZWb von unfertigen Erzeugnissen und in Arbeit befindlichen Aufträgen","DCV sur stocks de produits en cours de fabrication et commandes en cours"
"account_group_6343","6343","","AVA on inventories of goods","ZWb von Erzeugnissen","DCV sur stocks de produits"
"account_group_6344","6344","","AVA on inventories of merchandise and other goods for resale","ZWb von Waren und zum Verkauf bestimmten Gütern/Vermögensgegenständen","DCV sur stocks de marchandises et d'autres biens destinés à la revente"
"account_group_6345","6345","","AVA on down payments on inventories","ZWb von geleisteten Anzahlungen auf Vorräte","DCV sur acomptes versés sur stocks"
"account_group_635","635","","AVA and FVA on receivables from current assets","ZWb und FVA auf Forderungen aus dem Umlaufvermögen","Dotations aux corrections de valeur sur créances de l'actif circulant"
"account_group_6351","6351","","AVA on trade receivables","ZWb von Forderungen aus Lieferungen und Leistungen","DCV sur créances résultant de ventes et prestations de services"
"account_group_6352","6352","","AVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests","ZWb von Forderungen gegen verbundene Unternehmen und Unternehmen, mit denen ein Beteiligungsverhältnis besteht","DCV sur créances sur des entreprises liées et sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_6353","6353","","AVA on other receivables","ZWb von sonstigen Forderungen","DCV sur autres créances"
"account_group_6354","6354","","FVA on receivables from current assets","AFV von Forderungen des Umlaufvermögens","AJV sur créances de l'actif circulant"
"account_group_64","64","","Other operating charges","Sonstige betriebliche Aufwendungen","Autres charges d'exploitation"
"account_group_641","641","","Fees and royalties for concessions, patents, licences, trademarks and similar rights and assets","Gebühren und Entgelte für Konzessionen, Patente, Lizenzen, Warenzeichen und ähnliche Rechte und Werte","Redevances pour concessions, brevets, licences, marques, droits et valeurs similaires"
"account_group_6411","6411","","Concessions","Konzessionen",""
"account_group_6412","6412","","Patents","Patente","Brevets"
"account_group_6413","6413","","Software licences","Software- und Softwarepaketlizenzen","Licences informatiques"
"account_group_6414","6414","","Trademarks and franchise","Warenzeichen und Franchising (Verkaufskonzession/Alleinverkaufsrecht)","Marques et franchises"
"account_group_6415","6415","","Similar rights and assets","Ähnliche Rechte und Werte","Droits et valeurs similaires"
"account_group_64151","64151","","Copyrights and reproduction rights","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"account_group_64158","64158","","Other similar rights and assets","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"account_group_642","642","","Indemnities, damages and interest","Entschädigungen, Schadensersatz und Zinsen","Indemnités, dommages et intérêts"
"account_group_643","643","","Attendance fees, director's fees and similar remuneration","Sitzungsgelder, Verwaltungsratshonorare und ähnliche Vergütungen","Jetons de présence, tantièmes et autres rémunérations similaires"
"account_group_6431","6431","","Attendance fees","Sitzungsgeld","Jetons de présence"
"account_group_6432","6432","","Director's fees","Tantiemen","Tantièmes"
"account_group_6438","6438","","Other similar remuneration","Sonstige ähnliche Vergütungen","Autres rémunérations assimilées"
"account_group_644","644","","Loss on disposal of intangible and tangible fixed assets","Verlust aus dem Abgang von immateriellen Vermögensgegenständen und Sachanlagen","Perte sur la cession d'immobilisations incorporelles et corporelles"
"account_group_6441","6441","","Loss on disposal of intangible fixed assets","Verlust aus dem Abgang von immateriellen Anlagegütern","Perte sur la cession d'immobilisations incorporelles"
"account_group_64411","64411","","Book value of intangible fixed assets disposed of","Buchwert der verkauften immateriellen Anlagewerte","Valeur comptable d'immobilisations incorporelles cédées"
"account_group_64412","64412","","Disposal proceeds of intangible fixed assets","Verkaufserlös von immateriellen Anlagewerten","Produits de cession d'immobilisations incorporelles"
"account_group_6442","6442","","Loss on disposal of tangible fixed assets","Verlust aus dem Abgang von Sachanlagen","Perte sur la cession d'immobilisations corporelles"
"account_group_64421","64421","","Book value of tangible fixed assets disposed of","Buchwert der verkauften materiellen Anlagewerte","Valeur comptable d'immobilisations corporelles cédées"
"account_group_64422","64422","","Disposal proceeds of tangible fixed assets","Verkaufserlös von materiellen Anlagewerten","Produits de cession d'immobilisations corporelles"
"account_group_645","645","","Losses on bad debts","Verluste aus Forderungsausfällen","Pertes sur créances irrécouvrables"
"account_group_6451","6451","","Trade receivables","Verkäufe und Dienstleistungen","Ventes et prestations de services"
"account_group_6452","6452","","Amounts owed by affiliated undertakings","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"account_group_6453","6453","","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_6454","6454","","Other receivables","Sonstige Forderungen","Autres créances"
"account_group_646","646","","Taxes, duties and similar expenses","Steuern, Abgaben und ähnliche Kosten","Impôts, taxes et versements assimilés"
"account_group_6461","6461","","Real property tax","Grundsteuer","Impôt foncier"
"account_group_6462","6462","","Non-refundable VAT","Nicht erstattungsfähige Mehrwertsteuer (MwSt)","TVA non récupérable"
"account_group_6463","6463","","Duties on imported merchandise","Zölle und Einfuhrabgaben","Droits sur les marchandises en provenance de l'étranger"
"account_group_6464","6464","","Excise duties on production and tax on consumption","Verbrauchsabgaben aus der Produktion und Verbrauchssteuer","Droits d'accises à la production et taxe de consommation"
"account_group_6465","6465","","Registration fees, stamp duties and mortgage duties","Eintragungsgebühren, Stempelgebühren und Hypothekengebühren","Droits d'enregistrement et de timbre, droits d'hypothèques"
"account_group_64651","64651","","Registration fees","Eintragungsgebühren","Droits d'enregistrement"
"account_group_64658","64658","","Other registration fees, stamp duties and mortgage duties","Sonstige Eintragungs- und Stempelgebühren und Hypothekensteuern","Autres droits d'enregistrement et de timbre, droits d'hypothèques"
"account_group_6466","6466","","Motor-vehicle taxes","Kfz-Steuern","Taxes sur les véhicules"
"account_group_6467","6467","","Bar licence tax","Getränkesteuer","Taxe de cabaretage"
"account_group_6468","6468","","Other duties and taxes","Sonstige Gebühren und Steuern","Autres droits et taxes"
"account_group_647","647","","Allocations to tax-exempt capital gains","Zuführungen zu Sonderposten mit Rücklageanteil","Dotations aux plus-values immunisées"
"account_group_648","648","","Other miscellaneous operating charges","Sonstige sonstige betriebliche Aufwendungen","Autres charges d'exploitation diverses"
"account_group_6481","6481","","Fines, sanctions and penalties","Steuer- und strafrechtliche Bußgelder","Amendes, sanctions et pénalités"
"account_group_6488","6488","","Miscellaneous operating charges","Sonstige betriebliche Aufwendungen","Charges d'exploitation diverses"
"account_group_649","649","","Allocations to provisions","Zuführungen zu Rückstellungen","Dotations aux provisions d'exploitation"
"account_group_6491","6491","","Allocations to tax provisions","Zuführungen zu Steuerrückstellungen","Dotations aux provisions pour impôts"
"account_group_6492","6492","","Allocations to operating provisions","Zuführungen zu betrieblichen Rückstellungen","Dotations aux provisions d'exploitation"
"account_group_65","65","","Financial charges","Finanzielle Belastungen","Charges financières"
"account_group_651","651","","Allocations to value adjustments (AVA) and fair-value adjustments (FVA) of financial fixed assets","Zuweisungen zu Wertberichtigungen und Anpassungen des beizulegenden Zeitwerts von Finanzanlagen","Dotations aux corrections de valeur et ajustements pour juste valeur sur immobilisations financières"
"account_group_6511","6511","","AVA on financial fixed assets","Zuführungen zu Wertberichtigungen auf Finanzanlagen","Dotations aux corrections de valeur sur immobilisations financières"
"account_group_65111","65111","","AVA on shares in affiliated undertakings","ZWb von Anteilen an verbundenen Unternehmen","DCV sur parts dans des entreprises liées"
"account_group_65112","65112","","AVA on amounts owed by affiliated undertakings","ZWb von Forderungen gegen verbundene Unternehmen","DCV sur créances sur des entreprises liées"
"account_group_65113","65113","","AVA on participating interests","ZWb von Anteilen","DCV sur participations"
"account_group_65114","65114","","AVA on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","ZWb von Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","DCV sur créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_65115","65115","","AVA on securities held as fixed assets","ZWb von Wertpapieren des Anlagevermögens","DCV sur titres ayant le caractère d'immobilisations"
"account_group_65116","65116","","AVA on loans, deposits and claims held as fixed assets","ZWb von Ausleihungen, geleisteten Hinterlegungen und Forderungen des Anlagevermögens","DCV sur prêts, dépôts et créances immobilisés"
"account_group_6512","6512","","FVA on financial fixed assets","AFV der Finanzanlagen","AJV sur immobilisations financières"
"account_group_652","652","","Charges and loss of disposal of financial fixed assets","Aufwendungen und Verluste aus der Veräußerung von Finanzanlagen","Charges et pertes de cession d'immobilisations financières"
"account_group_6521","6521","","Charges of financial fixed assets","Belastungen von Finanzanlagen","Charges des immobilisations financières"
"account_group_65211","65211","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_65212","65212","","Amounts owed by affiliated undertakings","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"account_group_65213","65213","","Participating interests","Anteile","Participations"
"account_group_65214","65214","","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_65215","65215","","Securities held as fixed assets","Wertpapiere des Anlagevermögens","Titres ayant le caractère d'immobilisations"
"account_group_65216","65216","","Loans, deposits and claims held as fixed assets","Ausleihungen, geleistete Hinterlegungen und Forderungen (Anlagevermögen)","Prêts, dépôts et créances immobilisés"
"account_group_6522","6522","","Loss on disposal of financial fixed assets","Verlust aus der Veräußerung von Finanzanlagen","Perte sur la cession d'immobilisations financières"
"account_group_65221","65221","","Loss on disposal of shares in affiliated undertakings","Verlust aus der Veräußerung von Anteilen an verbundenen Unternehmen","Perte sur la cession de parts dans des entreprises liées"
"account_group_652211","652211","","Book value of shares in affiliated undertakings disposed of","Buchwert der verkauften Beteiligungen an verbundenen Unternehmen","Valeur comptable de parts cédées dans des entreprises liées"
"account_group_652212","652212","","Disposal proceeds of shares in affiliated undertakings","Veräußerungswert der Anteile an verbundenen Unternehmen","Produits de cession de parts dans des entreprises liées"
"account_group_65222","65222","","Loss on disposal of amounts owed by affiliated undertakings","Verlust aus dem Abgang von Forderungen gegenüber verbundenen Unternehmen","Perte sur la cession de créances sur des entreprises liées"
"account_group_652221","652221","","Book value of amounts owed by affiliated undertakings disposed of","Buchwert der verkauften Forderungen gegen verbundene Unternehmen","Valeur comptable de créances cédées sur des entreprises liées"
"account_group_652222","652222","","Disposal proceeds of amounts owed by affiliated undertakings","Veräußerungswert der Forderungen gegen verbundene Unternehmen","Produits de cession de créances sur des entreprises liées"
"account_group_65223","65223","","Loss on disposal of participating interests","Verlust aus der Veräußerung von Beteiligungen","Perte sur la cession de participations"
"account_group_652231","652231","","Book value of participating interests disposed of","Buchwert der verkauften Anteile","Valeur comptable de participations cédées"
"account_group_652232","652232","","Disposal proceeds of participating interests","Veräußerungswert der Anteile","Produits de cession de participations"
"account_group_65224","65224","","Loss on disposal of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Verlust aus dem Abgang von Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Perte sur la cession de créances sur des entreprises avec lesquelles l'entreprise est liée par des participations"
"account_group_652241","652241","","Book value of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests disposed of","Buchwert der verkauften Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Valeur comptable de créances cédées sur participations"
"account_group_652242","652242","","Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Veräußerungswert der Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Produits de cession de créances sur participations"
"account_group_65225","65225","","Loss on disposal of securities held as fixed assets","Verlust aus der Veräußerung von Wertpapieren des Anlagevermögens","Perte sur la cession de titres détenus en tant qu'actifs immobilisés"
"account_group_652251","652251","","Book value of securities held as fixed assets disposed of","Buchwert der verkauften Wertpapiere des Anlagevermögens","Valeur comptable de titres cédés ayant le caractère d'immobilisations"
"account_group_652252","652252","","Disposal proceeds of securities held as fixed assets","Veräußerungswert der Wertpapiere des Anlagevermögens","Produits de cession de titres ayant le caractère d'immobilisations"
"account_group_65226","65226","","Loss on disposal of loans, deposits and claims held as fixed assets","Verlust aus der Veräußerung von Krediten, Einlagen und Forderungen des Anlagevermögens","Perte sur la cession de prêts, dépôts et créances détenus en tant qu'actifs immobilisés"
"account_group_652261","652261","","Book value of yielded loans, deposits and claims held as fixed assets","Buchwert der verkauften Ausleihungen, geleistete Hinterlegungen und Forderungen des Umlaufvermögens","Valeur comptable de prêts, dépôts et créances immobilisés cédés"
"account_group_652262","652262","","Disposal proceeds of loans, deposits and claims held as fixed assets","Veräußerungswert der Ausleihungen, geleistete Hinterlegungen und Forderungen des Umlaufvermögens","Produits de cession de prêts, dépôts et créances immobilisés"
"account_group_653","653","","Allocations to value adjustment (AVA) and fair-value adjustments (FVA) on transferable securities","Zuweisungen zu Wertberichtigungen und Anpassungen des beizulegenden Zeitwerts für übertragbare Wertpapiere","Dotations aux corrections de valeur et ajustements pour juste valeur sur éléments financiers de l'actif circulant"
"account_group_6531","6531","","AVA on transferable securities","Zuweisungen zu Wertberichtigungen auf übertragbare Wertpapiere","Dotations aux corrections de valeur sur valeurs mobilières"
"account_group_65311","65311","","AVA on shares in affiliated undertakings","ZWb von Anteilen an verbundenen Unternehmen","DCV sur parts dans des entreprises liées"
"account_group_65312","65312","","AVA on own shares or own corporate units","ZWb von eigenen Aktien oder Anteilen","DCV sur actions propres ou parts propres"
"account_group_65313","65313","","AVA on shares in undertakings with which the undertaking is linked by virtue of participating interests","ZWb von Anteilen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","DCV sur parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_65318","65318","","AVA on other transferable securities","ZWb von sonstigen Wertpapieren","DCV sur autres valeurs mobilières"
"account_group_6532","6532","","FVA on transferable securities","AFV der Wertpapiere","AJV sur valeurs mobilières"
"account_group_654","654","","Loss on disposal of receivables and transferable securities from current assets","Verlust aus dem Abgang von Forderungen und Wertpapieren des Umlaufvermögens","Moins-values de cession de valeurs mobilières de l'actif circulant"
"account_group_6541","6541","","Loss on disposal of receivables from current assets","Verlust aus dem Abgang von Forderungen des Umlaufvermögens","Perte sur la cession de créances de l'actif circulant"
"account_group_65411","65411","","From affiliated undertakings","Forderungen gegen verbundene Unternehmen","Sur des entreprises liées"
"account_group_65412","65412","","From undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_65413","65413","","from other receivables from current assets","aus sonstigen Forderungen aus dem Umlaufvermögen","d'autres créances de l'actif circulant"
"account_group_6542","6542","","Loss on disposal of transferable securities","Verlust aus der Veräußerung von Wertpapieren","Perte sur la cession de valeurs mobilières"
"account_group_65421","65421","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_65422","65422","","Own shares or corporate units","Eigene Aktien oder Anteile","Actions propres ou parts propres"
"account_group_65423","65423","","Shares in in undertakings with which the undertaking is linked by virtue of participating interests","Anteile an Unternehmen, mit denen das Unternehmen durch eine Beteiligung verbunden ist","Actions dans des entreprises avec lesquelles l'entreprise est liée par des participations"
"account_group_65428","65428","","Other transferable securities","Sonstige Wertpapiere","Autres valeurs mobilières"
"account_group_655","655","","Interest and discounts","Zinsen und Rabatte","Intérêts et escomptes"
"account_group_6551","6551","","Interest on debenture loans","Zinsen für Schuldverschreibungsdarlehen","Intérêts des dettes financières"
"account_group_65511","65511","","Interest on debenture loans - affiliated undertakings","Zinsen auf Anleihen - verbundene Unternehmen","Intérêts sur emprunts obligataires - entreprises liées"
"account_group_65512","65512","","Interest on debenture loans - other","Zinsen auf Anleihen - sonstige","Intérêts sur emprunts obligataires - autres"
"account_group_6552","6552","","Banking and similar interest","Bankwesen und ähnliche Zinsen","Intérêts bancaires et assimilés"
"account_group_65521","65521","","Banking interest on current accounts","Kontozinsen","Intérêts sur comptes bancaires"
"account_group_65522","65522","","Banking interest on financing operations","Bankzinsen auf Finanzoperationen","Intérêts bancaires sur opérations de financement"
"account_group_65523","65523","","Interest on financial leases","Zinsen aus Finanzierungsleasingverträgen","Intérêts sur les contrats de location-financement"
"account_group_655231","655231","","Interest on financial leases - affiliated undertakings","Zinsen auf Finanzierungsleasings - verbundene Unternehmen","Intérêts sur leasings financiers - entreprises liées"
"account_group_655232","655232","","Interest on financial leases - other","Zinsen auf Finanzierungsleasings - sonstige","Intérêts sur leasings financiers - autres"
"account_group_6553","6553","","Interest on trade payables","Zinsen auf Verbindlichkeiten aus Lieferungen und Leistungen","Intérêts sur dettes commerciales"
"account_group_6554","6554","","Interest payable to affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests","Zinsen an verbundene Unternehmen und Unternehmen, mit denen das Unternehmen durch Beteiligungen verbunden ist","Intérêts dus aux entreprises liées et aux entreprises avec lesquelles l'entreprise est liée par des participations"
"account_group_65541","65541","","Interest payable to affiliated undertakings","Zinsen auf Verbindlichkeiten gegenüber verbundenen Unternehmen","Intérêts sur des entreprises liées"
"account_group_65542","65542","","Interest payable to undertakings with which the undertaking is linked by virtue of participating interests","Zinsen auf Verbindlichkeiten gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Intérêts sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_6555","6555","","Discounts and charges on bills of exchange","Wechseldiskont und Wechselspesen","Escomptes et frais sur effets de commerce"
"account_group_65551","65551","","Discounts and charges on bills of exchange - affiliated undertakings","Diskonte und Kosten von Handelswechsel - verbundene Unternehmen","Escomptes et frais sur effets de commerce - entreprises liées"
"account_group_65552","65552","","Discounts and charges on bills of exchange - other","Diskonte und Kosten von Handelswechsel - sonstige","Escomptes et frais sur effets de commerce - autres"
"account_group_6556","6556","","Granted discounts","Gewährung von Rabatten","Remises accordées"
"account_group_65561","65561","","Granted discounts - affiliated undertakings","Gewährte Diskonten - verbundene Unternehmen","Escomptes accordés - entreprises liées"
"account_group_65562","65562","","Granted discounts - other","Gewährte Diskonten - sonstige","Escomptes accordés - autres"
"account_group_6558","6558","","Interest payable on other loans and debts","Zinsaufwendungen für sonstige Darlehen und Schulden","Intérêts dus sur d'autres emprunts et dettes"
"account_group_65581","65581","","Interest payable on other loans and debts - affiliated undertakings","Zinsen auf sonstige Ausleihungen und Verbindlichkeiten - verbundene Unternehmen","Intérêts sur autres emprunts et dettes - entreprises liées"
"account_group_65582","65582","","Interest payable on other loans and debts - other","Zinsen auf sonstige Ausleihungen und Verbindlichkeiten - sonstige","Intérêts sur autres emprunts et dettes - autres"
"account_group_656","656","","Foreign currency exchange losses","Fremdwährungsverluste","Pertes de change"
"account_group_6561","6561","","Foreign currency exchange losses - affiliated undertakings","Wechselkursverluste - verbundene Unternehmen","Pertes de change - entreprises liées"
"account_group_6562","6562","","Foreign currency exchange losses - other","Wechselkursverluste - sonstige","Pertes de change - autres"
"account_group_657","657","","Share in the losses of undertakings accounted for under the equity method","Verlustanteile in den gemeinsamen Unternehmen (andere als Kapitalgesellschaften)","Quote-part dans la perte des entreprises mises en équivalence"
"account_group_658","658","","Other financial charges","Sonstige finanzielle Aufwendungen","Autres charges financières"
"account_group_6581","6581","","Other financial charges - affiliated undertakings","Sonstige finanzielle Aufwendungen - verbundene Unternehmen","Autres charges financières - entreprises liées"
"account_group_6582","6582","","Other financial charges - other","Sonstige finanzielle Aufwendungen - sonstige","Autres charges financières - autres"
"account_group_659","659","","Allocations to financial provisions","Zuführungen zu Finanzrückstellungen","Dotations aux provisions financières"
"account_group_6591","6591","","Allocations to financial provisions - affiliated undertakings","Zuführungen zu finanziellen Rückstellungen - verbundene Unternehmen","Dotations aux provisions financières - entreprises liées"
"account_group_6592","6592","","Allocations to financial provisions - other","Zuführungen zu finanziellen Rückstellungen - sonstige","Dotations aux provisions financières - autres"
"account_group_67","67","","Income taxes","Einkommensteuer","Impôts sur le revenu"
"account_group_671","671","","Corporate income tax (CIT)","Körperschaftssteuer (CIT)","Impôt sur les sociétés (IS)"
"account_group_6711","6711","","CIT - current financial year","KSt - laufendes Geschäftsjahr","IRC - exercice courant"
"account_group_6712","6712","","CIT - previous financial years","KSt - vorhergehende Geschäftsjahre","IRC - exercices antérieurs"
"account_group_672","672","","Municipal business tax","Gewerbesteuer","Impôt commercial communal (ICC)"
"account_group_6721","6721","","MBT - current financial year","GewSt - laufendes Geschäftsjahr","ICC - exercice courant"
"account_group_6722","6722","","MBT - previous financial years","GewSt - vorhergehende Geschäftsjahre","ICC - exercices antérieurs"
"account_group_673","673","","Foreign income taxes","Ausländische Einkommensteuern","Impôts sur le revenu à l'étranger"
"account_group_6731","6731","","Withholding taxes","Quellensteuern","Retenues d'impôt à la source"
"account_group_6732","6732","","Taxes levied on permanent establishments","Auf Betriebsstätten erhobene Steuern","Impôts prélevés sur les établissements stables"
"account_group_67321","67321","","Current financial year","Laufendes Geschäftsjahr","Exercice courant"
"account_group_67322","67322","","Previous financial years","Vorhergehende Geschäftsjahre","Exercices antérieurs"
"account_group_6733","6733","","Taxes levied on non-resident undertakings","Steuern, die durch die nicht gebietsansässige Unternehmen getragen wurden","Impôts supportés par les entreprises non résidentes"
"account_group_6738","6738","","Other foreign income taxes","Sonstige ausländische Steuern auf Einkommen und Erträge","Autres impôts étrangers sur le résultat"
"account_group_679","679","","Allocations to provisions for deferred taxes","Zuführungen zu Rückstellungen für Steuern","Dotations aux provisions pour impôts différés"
"account_group_68","68","","Other taxes not included in the previous caption","Sonstige Steuern, die in der vorstehenden Rubrik nicht enthalten sind","Autres taxes non comprises dans la rubrique précédente"
"account_group_681","681","","Net wealth tax (NWT)","Netto-Vermögensteuer (NWT)","Impôt sur le patrimoine net (ISN)"
"account_group_6811","6811","","NWT - current financial year","VermSt - laufendes Geschäftsjahr","IF - exercice courant"
"account_group_6812","6812","","NWT - previous financial years","VermSt - vorhergehende Geschäftsjahre","IF - exercices antérieurs"
"account_group_682","682","","Subscription tax","Abgeltungssteuer","Taxe d'abonnement"
"account_group_683","683","","Foreign taxes","Ausländische Steuern","Impôts étrangers"
"account_group_688","688","","Other taxes","Sonstige Steuern","Autres impôts"
"account_group_7","7","","INCOME ACCOUNTS","EINKOMMENSKONTEN","COMPTES DE REVENUS"
"account_group_70","70","","Net turnover","Nettoumsatz","Chiffre d'affaires net"
"account_group_702","702","","Sales of goods","Verkauf von Waren","Ventes de marchandises"
"account_group_7021","7021","","Sales of finished goods","Verkäufe von fertigen Erzeugnissen","Ventes de produits finis"
"account_group_7022","7022","","Sales of semi-finished goods","Verkäufe von Zwischenprodukten","Ventes de produits intermédiaires"
"account_group_7023","7023","","Sales of residual products","Verkäufe von Restprodukten","Ventes de produits résiduels"
"account_group_7029","7029","","Sales of work in progress","Verkäufe von unfertigen Erzeugnissen","Ventes de produits en cours de fabrication"
"account_group_703","703","","Sales of services","Verkauf von Dienstleistungen","Ventes de services"
"account_group_7031","7031","","Fees and royalties for concessions, patents, licences, trademarks and similar rights and assets","Gebühren und Entgelte für Konzessionen, Patente, Lizenzen, Warenzeichen und ähnliche Rechte und Werte","Redevances et royalties pour concessions, brevets, licences, marques et autres droits et actifs similaires"
"account_group_70311","70311","","Concessions","Konzessionen",""
"account_group_70312","70312","","Patents","Patente","Brevets"
"account_group_70313","70313","","Software licences","Software- und Softwarepaketlizenzen","Licences informatiques"
"account_group_70314","70314","","Trademarks and franchises","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"account_group_70315","70315","","Similar rights and assets","Ähnliche Rechte und Werte","Droits et biens similaires"
"account_group_703151","703151","","Copyrights and reproduction rights","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"account_group_703158","703158","","Other similar rights and assets","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"account_group_7032","7032","","Rental income","Mieteinnahmen","Revenus locatifs"
"account_group_70321","70321","","Rental income from real property","Mieterträge aus Immobilien","Revenus de location immobilière"
"account_group_70322","70322","","Rental income from movable property","Mieterträge aus beweglichem Vermögen","Revenus de location mobilière"
"account_group_7033","7033","","Sales of services not mentioned above","Dienstleistungen welche nicht oben erwähnt wurden","Prestations de services non visées ci-dessus"
"account_group_7039","7039","","Sales of services in the course of completion","Nicht ausgeführte Dienstleistungen","Prestations de services en cours de réalisation"
"account_group_704","704","","Sales of packaging","Verkäufe von Verpackungen","Ventes d'emballages"
"account_group_705","705","","Commissions and brokerage fees","Provisionen und Maklergebühren","Commissions et courtages"
"account_group_706","706","","Sales of merchandise and other goods for resale","Verkauf von Handelswaren und anderen Waren zum Wiederverkauf","Ventes de marchandises et d'autres biens destinés à la revente"
"account_group_7061","7061","","Sales of merchandise","Verkäufe von Waren","Ventes de marchandises"
"account_group_7062","7062","","Sales of land resale","Verkäufe von zum Verkauf bestimmten Grundstücken","Ventes de terrains destinés à la revente"
"account_group_7063","7063","","Sales of buildings for resale","Verkäufe von zum Verkauf bestimmten Bauten/Gebäuden","Ventes d'immeubles destinés à la revente"
"account_group_708","708","","Other components of turnover","Sonstige Umsatzerlöse","Autres éléments du chiffre d'affaires"
"account_group_709","709","","Rebates, discounts and refunds (RDR) granted and not immediately deducted from sales","Rabatte, Skonti und Rückvergütungen, die gewährt und nicht sofort vom Umsatz abgezogen werden","Rabais, remises et ristournes (RRR) accordés et non immédiatement déduits des ventes"
"account_group_7092","7092","","RDR on sales of goods","RPR auf Verkäufe von Erzeugnissen","RRR sur ventes de produits"
"account_group_7093","7093","","RDR on sales of services","RPR auf Verkäufe von Dienstleistungen","RRR sur prestations de services"
"account_group_7094","7094","","RDR on sales of packages","RPR auf Verkäufe von Verpackungen","RRR sur ventes d'emballages"
"account_group_7095","7095","","RDR on commissions and brokerage fees","RPR auf Kommissionen und Courtagen","RRR sur commissions et courtages"
"account_group_7096","7096","","RDR on sales of merchandise and other goods for resale","RPR auf Verkäufe von Waren und zum Verkauf bestimmten Gütern/Vermögensgegenständen","RRR sur ventes de marchandises et d'autres biens destinés à la revente"
"account_group_7098","7098","","RDR on other components of turnover","RPR auf sonstige Umsatzerlöse","RRR sur autres éléments du chiffre d'affaires"
"account_group_7099","7099","","Not allocated rebates, discounts and refunds","Nicht zugeordnete RPR","RRR non affectés"
"account_group_71","71","","Change in inventories of goods and of work in progress","Veränderung der Vorräte an Waren und unfertigen Erzeugnissen","Variation des stocks de marchandises et de travaux en cours"
"account_group_711","711","","Change in inventories of work and contracts in progress","Veränderung des Bestandes an unfertigen Leistungen und Aufträgen","Variation des stocks de travaux et de contrats en cours"
"account_group_7111","7111","","Change in inventories of work in progress","Bestandsveränderung von unfertigen Erzeugnissen","Variation des stocks de produits en cours de fabrication"
"account_group_7112","7112","","Change in inventories: contracts in progress - goods","Bestandsveränderung von in Arbeit befindlichen Aufträgen - Erzeugnisse","Variation des stocks : commandes en cours – produits"
"account_group_7113","7113","","Change in inventories: contracts in progress - services","Bestandsveränderung von in Arbeit befindlichen Aufträgen - Dienstleistungen","Variation des stocks : commandes en cours – prestations de services"
"account_group_7114","7114","","Change in inventories: buildings under construction","Bestandsveränderung von im Bau befindlichen Bauten","Variation des stocks : immeubles en construction"
"account_group_712","712","","Change in inventories of goods","Veränderung der Vorräte an Waren","Variation des stocks de marchandises"
"account_group_7121","7121","","Change in inventories of finished goods","Bestandsveränderung von fertigen Erzeugnissen","Variation des stocks de produits finis"
"account_group_7122","7122","","Change in inventories of semi-finished goods","Bestandsveränderung von Zwischenprodukten","Variation des stocks de produits intermédiaires"
"account_group_7123","7123","","Change in inventories of residual goods","Bestandsveränderung von Restprodukten","Variation des stocks de produits résiduels"
"account_group_72","72","","Capitalised production","Aktivierte Eigenleistungen","Production immobilisée"
"account_group_721","721","","Intangible fixed assets","Immaterielle Anlagewerte","Immobilisations incorporelles"
"account_group_7211","7211","","Development costs","Entwicklungskosten","Frais de développement"
"account_group_7212","7212","","Concessions, patents, licences, trademarks and similar rights and assets","Konzessionen, Patente, Lizenzen, Warenzeichen und ähnliche Rechte und Werte","Concessions, brevets, licences, marques, droits et valeurs similaires"
"account_group_72121","72121","","Concessions","Konzessionen",""
"account_group_72122","72122","","Patents","Patente","Brevets"
"account_group_72123","72123","","Software licences","Software- und Softwarepaketlizenzen","Licences informatiques"
"account_group_72124","72124","","Trademarks and franchises","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"account_group_72125","72125","","Similar rights and assets","Ähnliche Rechte und Werte","Droits et valeurs similaires"
"account_group_721251","721251","","Copyrights and reproduction rights","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"account_group_721258","721258","","Other similar rights and assets","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"account_group_722","722","","Tangible fixed assets","Materielle Anlagewerte","Immobilisations corporelles"
"account_group_7221","7221","","Land, fittings and buildings","Grundstücke, Erschließungen, Einrichtungen und Bauten","Terrains, aménagements et constructions"
"account_group_7222","7222","","Plant and machinery","Technische Anlagen und Maschinen","Installations techniques et machines"
"account_group_7223","7223","","Other fixtures and fittings, tools and equipment (included motor vehicles)","Sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Autres installations, outillage et mobilier (y compris matériel roulant)"
"account_group_73","73","","Reversals of value adjustments (RVA) on intangible, tangible and current assets (except transferable securities)","Auflösung von Wertberichtigungen auf immaterielle Anlagewerte, Sachanlagen und Umlaufvermögen (mit Ausnahme von übertragbaren Wertpapieren)","Reprises de corrections de valeur sur actifs incorporels, corporels et circulants (à l'exception des valeurs mobilières)"
"account_group_732","732","","RVA on intangible fixed assets","Auflösungen von Wertberichtigungen auf immaterielle Anlagewerte","Reprises de corrections de valeur sur immobilisations incorporelles"
"account_group_7321","7321","","RVA on development costs","Wertaufholungen von Entwicklungskosten","RCV sur frais de développement"
"account_group_7322","7322","","RVA on concessions, patents, licences, trademarks and similar rights and assets","Wertaufholungen von Konzessionen, Patenten, Lizenzen, Warenzeichen und ähnlichen Rechten und Werten","RCV sur concessions, brevets, licences, marques ainsi que droits et valeurs similaires"
"account_group_7324","7324","","RVA on down payments and intangible fixed assets under development","Wertaufholungen von geleisteten Anzahlungen und immateriellen Vermögensgegenständen in Entwicklung","RCV sur acomptes versés et immobilisations incorporelles en cours"
"account_group_733","733","","RVA on tangible fixed assets and fair value adjustments (FVA) on investment properties","Auflösungen von Wertberichtigungen auf Sachanlagen und Zeitwertanpassungen auf als Finanzinvestition gehaltene Immobilien","Reprises de corrections de valeur sur les immobilisations corporelles et de corrections de juste valeur sur les immeubles de placement"
"account_group_7331","7331","","RVA on land, fixtures and fittings-out and buildings and FVA on investment properties","Auflösungen von Wertberichtigungen auf Grundstücke, Betriebs- und Geschäftsausstattung sowie Gebäude und Zeitwertanpassungen auf als Finanzinvestition gehaltene Immobilien","Reprises de corrections de valeur sur terrains, agencements et constructions et de corrections de juste valeur sur immeubles de placement"
"account_group_73311","73311","","RVA on land","Wertaufholungen von Grundstücken, Erschließungen, Einrichtungen und Bauten","RCV sur terrains"
"account_group_73312","73312","","RVA on fixtures and fittings-out of land","Wertaufholungen von Erschließungen von Grundstücken","RCV sur agencements et aménagements de terrains"
"account_group_73313","73313","","RVA on buildings","Wertaufholungen von Bauten","RCV sur constructions / bâtiments"
"account_group_73314","73314","","RVA on fixtures and fittings-out of buildings","Wertaufholungen von Einrichtungen von Bauten/Gebäuden","RCV sur agencements et aménagements de constructions / bâtiments"
"account_group_73315","73315","","FVA on investment properties","AFV von Anlageimmobilien","AJV sur immeubles de placement"
"account_group_7332","7332","","RVA on plant and machinery","Wertaufholungen von technischen Anlagen und Maschinen","RCV sur installations techniques et machines"
"account_group_7333","7333","","Other fixtures and fittings, tools and equipment (included motor vehicles)","Sonstige Anlagen, Betriebs- und Geschäftsausstattung (Fuhrpark inbegriffen)","Autres installations, outillage et mobilier (y compris matériel roulant)"
"account_group_7334","7334","","RVA on down payments and tangible fixed assets under development","Wertaufholungen von geleisteten Anzahlungen und Anlagen im Bau","RCV sur acomptes versés et immobilisations corporelles en cours"
"account_group_734","734","","RVA on inventories","Rückbuchung von Wertberichtigungen auf Vorräte","Reprises de corrections de valeur sur stocks"
"account_group_7341","7341","","RVA on inventories of raw materials and consumables","Wertaufholungen von Roh-, Hilfs- und Betriebsstoffen","RCV sur stocks de matières premières et consommables"
"account_group_7342","7342","","RVA on inventories of work and contracts in progress","Wertaufholungen von unfertigen Erzeugnisse und sich in Arbeit befindlichen Aufträgen","RCV sur stocks de produits en cours de fabrication et commandes en cours"
"account_group_7343","7343","","RVA on inventories of goods","Erzeugnisse","RCV sur stocks de produits"
"account_group_7344","7344","","RVA on inventories of merchandise and other goods for resale","Waren und sonstige zum Verkauf bestimmte Güter/Vermögensgegenstände","RCV sur stocks de marchandises et autres biens destinés à la revente"
"account_group_7345","7345","","RVA on down payments on inventories","Geleistete Anzahlungen auf Vorräte","RCV sur acomptes versés sur stocks"
"account_group_735","735","","RVA and FVA on receivables from current assets","Auflösung von Wertberichtigungen und Anpassungen des beizulegenden Zeitwerts auf Forderungen des Umlaufvermögens","Reprises de corrections de valeur et de justes valeurs sur les créances de l'actif circulant"
"account_group_7351","7351","","RVA on trade receivables","Forderungen aus Lieferungen und Leistungen","RCV sur créances résultant de ventes et prestations de services"
"account_group_7352","7352","","RVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen verbundene Unternehmen und gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","RCV sur créances sur des entreprises liées et sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_7353","7353","","RVA on other receivables","Sonstige Forderungen","RCV sur autres créances"
"account_group_7354","7354","","FVA on receivables from current assets","AFV von Forderungen des Umlaufvermögens","AJV sur créances de l'actif circulant"
"account_group_74","74","","Other operating income","Sonstige betriebliche Erträge","Autres produits d'exploitation"
"account_group_741","741","","Fees and royalties for concessions, patents, licences, trademarks and similar rights and assets from ancillary activities","Gebühren und Entgelte für Konzessionen, Patente, Lizenzen, Warenzeichen und ähnliche Rechte und Werte aus Nebentätigkeiten","Redevances et royalties pour concessions, brevets, licences, marques et droits similaires et actifs provenant d'activités auxiliaires"
"account_group_7411","7411","","Concessions","Konzessionen",""
"account_group_7412","7412","","Patents","Patente","Brevets"
"account_group_7413","7413","","Software licences","Software- und Softwarepaketlizenzen","Licences informatiques"
"account_group_7414","7414","","Trademarks and franchises","Warenzeichen und Verkaufskonzession / Alleinverkaufsrecht","Marques et franchises"
"account_group_7415","7415","","Similar rights and assets","Ähnliche Rechte und Werte","Droits et biens similaires"
"account_group_74151","74151","","Copyrights and reproduction rights","Urheber- und Reproduktionsrechte","Droits d'auteur et de reproduction"
"account_group_74158","74158","","Other similar rights and assets","Sonstige vergleichbare Rechte und Werte","Autres droits et valeurs similaires"
"account_group_742","742","","Rental income from ancillary activities","Mieteinnahmen aus Nebentätigkeiten","Revenus locatifs des activités annexes"
"account_group_7421","7421","","Rental income on real property","Mieterträge aus Immobilien","Revenus de location immobilière"
"account_group_7422","7422","","Rental income on movable property","Mieterträge aus beweglichem Vermögen","Revenus de location mobilière"
"account_group_743","743","","Attendance fees, director's fees and similar remunerations","Sitzungsgeld, Tantiemen und vergleichbare Erträge","Jetons de présence, tantièmes et rémunérations assimilées"
"account_group_744","744","","Gain on disposal of intangible and tangible fixed assets","Gewinn aus dem Abgang von immateriellen Vermögenswerten und Sachanlagen","Gain sur la cession d'immobilisations incorporelles et corporelles"
"account_group_7441","7441","","Gain on disposal of intangible fixed assets","Gewinn aus dem Abgang von immateriellen Anlagegütern","Gain sur la cession d'immobilisations incorporelles"
"account_group_74411","74411","","Book value of yielded intangible fixed assets","Buchwert der verkauften immateriellen Anlagewerte","Valeur comptable d'immobilisations incorporelles cédées"
"account_group_74412","74412","","Disposal proceeds of intangible fixed assets","Verkaufserlös von immateriellen Anlagewerten","Produits de cession d'immobilisations incorporelles"
"account_group_7442","7442","","Income of yielded tangible fixed assets","Erträge aus veräußerten Sachanlagen","Revenus des immobilisations corporelles cédées"
"account_group_74421","74421","","Book value of yielded tangible fixed assets","Buchwert der verkauften materiellen Anlagewerte","Valeur comptable d'immobilisations corporelles cédées"
"account_group_74422","74422","","Disposal proceeds of tangible fixed assets","Verkaufserlös von materiellen Anlagewerten","Produits de cession d'immobilisations corporelles"
"account_group_745","745","","Subsidies for operating activities","Zuschüsse für operative Tätigkeiten","Subventions pour les activités opérationnelles"
"account_group_7451","7451","","Product subsidies","Produktzuschüsse","Subventions sur produits"
"account_group_7452","7452","","Interest subsidies","Zinszuschüsse","Bonifications d'intérêt"
"account_group_7453","7453","","Compensatory allowances","Ausgleichszahlungen","Indemnités compensatoires"
"account_group_7454","7454","","Subsidies in favour of employment development","Zuschüsse zur Beschäftigungsförderung","Subventions destinées à promouvoir l'emploi"
"account_group_7458","7458","","Other subsidies for operating activities","Sonstige Zuschüsse für die laufende Geschäftstätigkeit","Autres subventions d'exploitation"
"account_group_746","746","","Benefits in kind","Sachleistungen","Avantages en nature"
"account_group_747","747","","Reversals of temporarily not taxable capital gains and of investment subsidies","Rückbuchungen von vorübergehend nicht steuerpflichtigen Kapitalerträgen und von Investitionszuschüssen","Reprises de plus-values temporairement non imposables et de subventions d'investissement"
"account_group_7471","7471","","Temporarily not taxable capital gains not reinvested","Sonderposten mit Rücklageanteil, nicht wiederangelegt","Plus-values immunisées non réinvesties"
"account_group_7472","7472","","Temporarily not taxable capital gains reinvested","Sonderposten mit Rücklageanteil, wiederangelegt","Plus-values immunisées réinvesties"
"account_group_7473","7473","","Capital investment subsidies","Investitionszuschüsse","Subventions d'investissement en capital"
"account_group_748","748","","Other miscellaneous operating income","Sonstige übrige betriebliche Erträge","Autres produits d'exploitation divers"
"account_group_7481","7481","","Insurance indemnities","Versicherungsentschädigungen","Indemnités d'assurance"
"account_group_7488","7488","","Miscellaneous operating income","Andere betriebliche Erträge","Produits d'exploitation divers"
"account_group_749","749","","Reversals of provisions","Auflösungen von Rückstellungen","Reprises de provisions"
"account_group_7491","7491","","Reversals of provisions for taxes","Wertaufholungen von Steuerrückstellungen","Reprises de provisions pour impôts"
"account_group_7492","7492","","Reversals of operating provisions","Wertaufholungen von betrieblichen Rückstellungen","Reprises de provisions d'exploitation"
"account_group_75","75","","Financial income","Finanzielle Einnahmen","Produits financiers"
"account_group_751","751","","Reversals of value adjustments (RVA) and fair-value adjustments (FVA) on financial fixed assets","Rückgängigmachung von Wertberichtigungen (RVA) und Zeitwertanpassungen (FVA) bei Finanzanlagen","Reprises de corrections de valeur (RVA) et de corrections de juste valeur (FVA) sur immobilisations financières"
"account_group_7511","7511","","RVA on financial fixed assets","Rückbuchungen von Wertberichtigungen auf Finanzanlagen","Reprises de corrections de valeur sur immobilisations financières"
"account_group_75111","75111","","RVA on shares in affiliated undertakings","Auflösungen von Wertberichtigungen auf Anteile an verbundenen Unternehmen","Reprises de corrections de valeur sur les parts dans les entreprises liées"
"account_group_75112","75112","","RVA on amounts owed by affiliated undertakings","Auflösungen von Wertberichtigungen auf Forderungen gegenüber verbundenen Unternehmen","Reprises de corrections de valeur sur créances d'entreprises liées"
"account_group_75113","75113","","RVA on participating interests","Rückbuchung von Wertberichtigungen auf Beteiligungen","Reprises de corrections de valeur sur participations"
"account_group_75114","75114","","RVA on amounts owed by undertakings with which the company is linked by virtue of participating interests","Auflösungen von Wertberichtigungen auf Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Reprises de corrections de valeur sur créances d'entreprises auxquelles la société est liée par des participations"
"account_group_75115","75115","","RVA on securities held as fixed assets","Auflösungen von Wertberichtigungen auf Wertpapiere des Anlagevermögens","Reprises de corrections de valeur sur titres immobilisés"
"account_group_75116","75116","","RVA on loans, deposits and claims held as fixed assets","Auflösungen von Wertberichtigungen auf Darlehen, Einlagen und Forderungen des Anlagevermögens","Reprises de corrections de valeur sur prêts, dépôts et créances détenus à titre d'actifs immobilisés"
"account_group_7512","7512","","FVA on financial fixed assets","AFV der Finanzanlagen","AJV sur immobilisations financières"
"account_group_752","752","","Income and gain on disposal of financial fixed assets","Erträge und Gewinne aus der Veräußerung von Finanzanlagen","Produits et gains sur la cession d'immobilisations financières"
"account_group_7521","7521","","Income from financial fixed assets","Erträge aus Finanzanlagen","Produits des immobilisations financières"
"account_group_75211","75211","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_75212","75212","","Amounts owed by affiliated undertakings","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"account_group_75213","75213","","Participating interests","Anteile","Participations"
"account_group_75214","75214","","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_75215","75215","","Securities held as fixed assets","Wertpapiere des Anlagevermögens","Titres ayant le caractère d'immobilisations"
"account_group_75216","75216","","Loans, deposits and claims held as fixed assets","Ausleihungen, geleistete Hinterlegungen und Forderungen (Anlagevermögen)","Prêts, dépôts et créances immobilisés"
"account_group_7522","7522","","Gain on disposal of financial fixed assets","Gewinn aus der Veräußerung von Finanzanlagen","Gain sur la cession d'immobilisations financières"
"account_group_75221","75221","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_752211","752211","","Book value of yielded shares in affiliated undertakings","Buchwert der verkauften Beteiligungen an verbundenen Unternehmen","Valeur comptable de parts cédées dans des entreprises liées"
"account_group_752212","752212","","Disposal proceeds of shares in affiliated undertakings","Veräußerungswert der Anteile an verbundenen Unternehmen","Produits de cession de parts dans des entreprises liées"
"account_group_75222","75222","","Amounts owed by affiliated undertakings","Forderungen gegen verbundene Unternehmen","Créances sur des entreprises liées"
"account_group_752221","752221","","Book value of yielded amounts owed by affiliated undertakings","Buchwert der verkauften Forderungen gegen verbundene Unternehmen","Valeur comptable de créances cédées sur des entreprises liées"
"account_group_752222","752222","","Disposal proceeds of amounts owed by affiliated undertakings","Veräußerungswert der Forderungen gegen verbundene Unternehmen","Produits de cession de créances sur des entreprises liées"
"account_group_75223","75223","","Participating interests","Anteile","Participations"
"account_group_752231","752231","","Book value of yielded participating interests","Buchwert der verkauften Anteile","Valeur comptable de participations cédées"
"account_group_752232","752232","","Disposal proceeds of participating interests","Veräußerungswert der Anteile","Produits de cession de participations"
"account_group_75224","75224","","Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Créances sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_752241","752241","","Book value of yielded amounts owed by undertakings with which the undertaking is linked by virtue of participating  interests","Buchwert der ertragsabhängigen Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Valeur comptable des créances cédées par des entreprises auxquelles l'entreprise est liée par des participations"
"account_group_752242","752242","","Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Veräußerungswert der Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Produits de cession de créances sur participations"
"account_group_75225","75225","","Securities held as fixed assets","Wertpapiere des Anlagevermögens","Titres ayant le caractère d'immobilisations"
"account_group_752251","752251","","Book value of yielded securities held as fixed assets","Buchwert der verkauften Wertpapiere des Anlagevermögens","Valeur comptable de titres cédés ayant le caractère d'immobilisations"
"account_group_752252","752252","","Disposal proceeds of securities held as fixed assets","Veräußerungswert der Wertpapiere des Anlagevermögens","Produits de cession de titres ayant le caractère d'immobilisations"
"account_group_75226","75226","","Loans, deposits and claims held as fixed assets","Ausleihungen, geleistete Hinterlegungen und Forderungen (Anlagevermögen)","Prêts, dépôts et créances immobilisés"
"account_group_752261","752261","","Book value of yielded loans, deposits and claims held as fixed assets","Buchwert der verkauften Ausleihungen, geleistete Hinterlegungen und Forderungen des Umlaufvermögens","Valeur comptable de prêts, dépôts et créances immobilisés cédés"
"account_group_752262","752262","","Disposal proceed of loans, deposits and claims held as fixed assets","Veräußerungswert der Ausleihungen, der geleistete Hinterlegungen und der Forderungen des Anlagevermögens","Produits de cession de prêts, dépôts et créances immobilisés"
"account_group_753","753","","Reversals of value adjustments (RVA) and fair-value adjustments (FVA) on transferable securities","Auflösungen von Wertberichtigungen und Anpassungen des beizulegenden Zeitwerts bei Wertpapieren","Reprises de corrections de valeur et de justes valeurs sur valeurs mobilières"
"account_group_7531","7531","","RVA on transferable securities","Auflösungen von Wertberichtigungen auf übertragbare Wertpapiere","Reprises de corrections de valeur sur valeurs mobilières"
"account_group_75311","75311","","RVA on shares in affiliated undertakings","Auflösungen von Wertberichtigungen auf Anteile an verbundenen Unternehmen","Reprises de corrections de valeur sur les parts dans les entreprises liées"
"account_group_75312","75312","","RVA on own shares or corporate units","Auflösungen von Wertberichtigungen auf eigene Handlungen oder auf Sozialleistungen","Reprises de corrections de valeur sur les actions propres ou les parts sociales"
"account_group_75313","75313","","RVA on shares in undertakings with which the undertaking is linked by virtue of participating interests","Auflösung von Wertberichtigungen auf Anteile an Unternehmen, mit denen das Unternehmen durch ein Beteiligungsverhältnis verbunden ist","Reprises de corrections de valeur sur les parts d'entreprises auxquelles l'entreprise est liée par des participations"
"account_group_75318","75318","","RVA on other transferable securities","Rückbuchungen von Wertberichtigungen auf sonstige Wertpapiere","Reprises de corrections de valeur sur autres valeurs mobilières"
"account_group_7532","7532","","FVA on transferable securities","AFV der Wertpapiere","AJV sur valeurs mobilières"
"account_group_754","754","","Gain on disposal and other income from current receivables and transferable securities of current assets","Veräußerungsgewinne und sonstige Erträge aus kurzfristigen Forderungen und Wertpapieren des Umlaufvermögens","Plus-values de cession et autres revenus des créances à court terme et des valeurs mobilières de l'actif circulant"
"account_group_7541","7541","","Gain on disposal of receivables from current assets","Gewinn aus dem Abgang von Forderungen aus dem Umlaufvermögen","Gain sur la cession de créances de l'actif circulant"
"account_group_75411","75411","","on affiliated undertakings","über verbundene Unternehmen","sur les entreprises liées"
"account_group_75412","75412","","on undertakings with which the undertaking is linked by virtue of participating interests","über Unternehmen, mit denen das Unternehmen durch Beteiligungen verbunden ist","sur les entreprises avec lesquelles l'entreprise est liée par des participations"
"account_group_75413","75413","","on other current receivables","auf sonstige kurzfristige Forderungen","sur les autres créances à court terme"
"account_group_7542","7542","","Gain on disposal of transferable securities","Gewinn aus der Veräußerung von übertragbaren Wertpapieren","Gain sur la cession de valeurs mobilières"
"account_group_75421","75421","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_75422","75422","","Own shares or corporate units","Eigene Aktien oder Anteile","Actions propres ou parts propres"
"account_group_75423","75423","","Shares in undertakings with which the undertaking is linked by virtue of participating interests","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_75428","75428","","Other transferable securities","Sonstige Wertpapiere","Autres valeurs mobilières"
"account_group_7548","7548","","Other income from transferable securities","Sonstige Erträge aus übertragbaren Wertpapieren","Autres revenus des valeurs mobilières"
"account_group_75481","75481","","Shares in affiliated undertakings","Anteile an verbundenen Unternehmen","Parts dans des entreprises liées"
"account_group_75482","75482","","Own shares or corporate units","Eigene Aktien oder Anteile","Actions propres ou parts propres"
"account_group_75483","75483","","Shares in undertakings with which the undertaking is linked by virtue of participating interests","Anteile an Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Parts dans des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_75488","75488","","Other transferable securities","Sonstige Wertpapiere","Autres valeurs mobilières"
"account_group_755","755","","Other interest income from current assets and discounts","Sonstige Zinserträge aus Umlaufvermögen und Disagio","Autres produits d'intérêts sur actifs circulants et escomptes"
"account_group_7552","7552","","Bank and similar interest","Bankzinsen und ähnliche Zinsen","Intérêts bancaires et assimilés"
"account_group_75521","75521","","Interest on bank accounts","Kontozinsen","Intérêts sur comptes bancaires"
"account_group_75523","75523","","Interest on financial leases","Zinsen aus Finanzierungsleasingverträgen","Intérêts sur les contrats de location-financement"
"account_group_755231","755231","","from affiliated undertakings","von verbundenen Unternehmen","provenant d'entreprises liées"
"account_group_755232","755232","","from other","von anderen","d'autres"
"account_group_7553","7553","","Interest on trade receivables","Zinsen auf Handelsforderungen","Intérêts sur créances commerciales"
"account_group_7554","7554","","Interest on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests","Zinsen auf Forderungen gegen verbundene Unternehmen und Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Intérêts sur les créances des entreprises liées et des entreprises avec lesquelles l'entreprise est liée par des participations"
"account_group_75541","75541","","Interest on amounts owed by affiliated undertakings","Zinsen auf Forderungen gegen verbundene Unternehmen","Intérêts sur des entreprises liées"
"account_group_75542","75542","","Interest on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests","Zinsen auf Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht","Intérêts sur des entreprises avec lesquelles l'entreprise a un lien de participation"
"account_group_7555","7555","","Discounts on bills of exchange","Wechseldiskont","Escomptes sur lettres de change"
"account_group_75551","75551","","Discounts on bills of exchange - affiliated undertakings","Diskonte auf Handelswechsel - verbundene Unternehmen","Escomptes d'effets de commerce - entreprises liées"
"account_group_75552","75552","","Discounts on bills of exchange - other","Diskonte auf Handelswechsel - sonstige","Escomptes d'effets de commerce - autres"
"account_group_7556","7556","","Discounts received","Erhaltene Ermäßigungen","Remises reçues"
"account_group_75561","75561","","Discounts received - affiliated undertakings","Erhaltene Diskonten - verbundene Unternehmen","Escomptes obtenus - entreprises liées"
"account_group_75562","75562","","Discounts received - other","Erhaltene Diskonten - sonstige","Escomptes obtenus - autres"
"account_group_7558","7558","","Interest on other amounts receivable","Zinsen auf sonstige Forderungen","Intérêts sur d'autres créances"
"account_group_75581","75581","","Interest on other amounts receivable - affiliated undertakings","Zinsen auf sonstigen Forderungen - verbundene Unternehmen","Intérêts sur autres créances - entreprises liées"
"account_group_75582","75582","","Interest on other amounts receivable - other","Zinsen auf sonstigen Forderungen - sonstige","Intérêts sur autres créances - autres"
"account_group_756","756","","Foreign currency exchange gains","Kursgewinne aus Fremdwährungen","Gains de change"
"account_group_7561","7561","","Foreign currency exchange gains - affiliated undertakings","Wechselkursgewinne - verbundene Unternehmen","Gains de change - entreprises liées"
"account_group_7562","7562","","Foreign currency exchange gains - other","Wechselkursgewinne - sonstige","Gains de change - autres"
"account_group_757","757","","Share of profit from undertakings accounted for under the equity method","Gewinnanteil aus Unternehmungen (andere als Kapitalgesellschaften)","Quote-part de bénéfice dans les entreprises mises en équivalence"
"account_group_758","758","","Other financial income","Sonstige Finanzerträge","Autres produits financiers"
"account_group_7581","7581","","Other financial income - affiliated undertakings","Sonstige finanzielle Erträge - verbundene Unternehmen","Autres produits financiers - entreprises liées"
"account_group_7582","7582","","Other financial income - other","Sonstige finanzielle Erträge - sonstige","Autres produits financiers - autres"
"account_group_759","759","","Reversals of financial provisions","Auflösungen von Finanzrückstellungen","Reprises de provisions financières"
"account_group_7591","7591","","Reversals of financial provisions - affiliated undertakings","Wertaufholungen von finanziellen Rückstellungen - verbundene Unternehmen","Reprises de provisions financières - entreprises liées"
"account_group_7592","7592","","Reversals of financial provisions - other","Wertaufholungen von finanziellen Rückstellungen - sonstige","Reprises de provisions financières - autres"
"account_group_77","77","","Adjustments of income taxes","Anpassungen der Ertragsteuern","Ajustements de l'impôt sur le revenu"
"account_group_771","771","","Adjustments of corporate income tax (CIT)","Erstattung der Körperschaftssteuer","Régularisations d'impôt sur le revenu des collectivités (IRC)"
"account_group_772","772","","Adjustments of municipal business tax (MBT)","Erstattung der Gewerbesteuer","Régularisations d'impôt commercial communal (ICC)"
"account_group_773","773","","Adjustments of foreign income taxes","Erstattung der ausländischen Ertragssteuern","Régularisations d'impôts étrangers sur le résultat"
"account_group_779","779","","Reversals of provisions for deferred taxes","Auflösungen von Rückstellungen für Ertragssteuern","Reprises de provisions pour impôts différés"
"account_group_78","78","","Adjustments of other taxes not included in the previous caption","Anpassungen der sonstigen Steuern, die nicht in der vorherigen Rubrik enthalten sind","Ajustements d'autres impôts non inclus dans la rubrique précédente"
"account_group_781","781","","Adjustments of net wealth tax (NWT)","Erstattung der Vermögensteuer","Régularisations d'impôt sur la fortune (IF)"
"account_group_782","782","","Adjustments of subscription tax","Erstattung der Abgeltungssteuer","Régularisations de taxe d'abonnement"
"account_group_783","783","","Adjustments of foreign taxes","Erstattung der ausländischen Steuern","Régularisations d'impôts étrangers"
"account_group_788","788","","Adjustments of other taxes","Erstattung sonstiger Steuern","Régularisations d'autres impôts"

```

## File: data\template\account.tax-lu.csv

```csv
"id","sequence","description","invoice_label","name","amount","amount_type","type_tax_use","tax_group_id","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","repartition_line_ids/account_id"
"lu_2011_tax_AB-EC-0","171","","0%","0% EX EC G","0.0","percent","purchase","tax_group_0","","base","invoice","+195","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-195","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_AB-EC-14","105","","14%","14% EC G","14.0","percent","purchase","tax_group_14","","base","invoice","+723","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+724","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-723","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-724","","lu_2020_account_421611"
"lu_2015_tax_AB-EC-17","111","","17%","17% EC G","17.0","percent","purchase","tax_group_17","","base","invoice","+721","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+722","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-721","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-722","","lu_2020_account_421611"
"lu_2015_tax_AB-EC-16","112","","16%","16% EC G","16.0","percent","purchase","tax_group_16","False","base","invoice","+921","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+922","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-921","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-922","","lu_2020_account_421611"
"lu_2015_tax_AB-EC-13","113","","13%","13% EC G","13.0","percent","purchase","tax_group_13","False","base","invoice","+923","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+924","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-923","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-924","","lu_2020_account_421611"
"lu_2015_tax_AB-EC-7","114","","7%","7% EC G","7.0","percent","purchase","tax_group_7","False","base","invoice","+925","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+926","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-925","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-926","","lu_2020_account_421611"
"lu_2011_tax_AB-EC-3","114","","3%","3% EC G","3.0","percent","purchase","tax_group_3","","base","invoice","+059","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+068","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-059","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-068","","lu_2020_account_421611"
"lu_2015_tax_AB-EC-8","120","","8%","8% EC G","8.0","percent","purchase","tax_group_8","","base","invoice","+725","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+726","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-725","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-726","","lu_2020_account_421611"
"lu_2015_tax_AB-ECP-0","123","","0%","0% EX EC(P) G","0.0","percent","purchase","tax_group_0","","base","invoice","+196","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-196","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_AB-ECP-14","127","","14%","14% EC(P) G","14.0","percent","purchase","tax_group_14","","base","invoice","+733","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+734","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-733","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-734","","lu_2020_account_421611"
"lu_2015_tax_AB-ECP-17","133","","17%","17% EC(P) G","17.0","percent","purchase","tax_group_17","","base","invoice","+731","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+732","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-731","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-732","","lu_2020_account_421611"
"lu_2015_tax_AB-ECP-16","134","","16%","16% EC(P) G","16.0","percent","purchase","tax_group_16","False","base","invoice","+931","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+932","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-931","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-932","","lu_2020_account_421611"
"lu_2015_tax_AB-ECP-13","135","","13%","13% EC(P) G","13.0","percent","purchase","tax_group_13","False","base","invoice","+933","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+934","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-933","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-934","","lu_2020_account_421611"
"lu_2015_tax_AB-ECP-7","136","","7%","7% EC(P) G","7.0","percent","purchase","tax_group_7","False","base","invoice","+935","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+936","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-935","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-936","","lu_2020_account_421611"
"lu_2015_tax_AB-ECP-3","137","","3%","3% EC(P) G","3.0","percent","purchase","tax_group_3","","base","invoice","+063","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+073","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-063","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-073","","lu_2020_account_421611"
"lu_2015_tax_AB-ECP-8","142","","8%","8% EC(P) G","8.0","percent","purchase","tax_group_8","","base","invoice","+735","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+736","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-735","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-736","","lu_2020_account_421611"
"lu_2011_tax_AB-IC-0","145","","0%","0% EX IC G","0.0","percent","purchase","tax_group_0","","base","invoice","+194","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-194","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_AB-IC-14","149","","14%","14% IC G","14.0","percent","purchase","tax_group_14","","base","invoice","+713","",""
"","","","","","","","","","","tax","invoice","-714","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-713","",""
"","","","","","","","","","","tax","refund","+714","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_AB-IC-17","155","","17%","17% IC G","17.0","percent","purchase","tax_group_17","","base","invoice","+711","",""
"","","","","","","","","","","tax","invoice","-712","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-711","",""
"","","","","","","","","","","tax","refund","+712","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_AB-IC-16","155","","16%","16% IC G","16.0","percent","purchase","tax_group_16","False","base","invoice","+911","",""
"","","","","","","","","","","tax","invoice","-912","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-911","",""
"","","","","","","","","","","tax","refund","+912","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_AB-IC-13","156","","13%","13% IC G","13.0","percent","purchase","tax_group_13","False","base","invoice","+913","",""
"","","","","","","","","","","tax","invoice","-914","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-913","",""
"","","","","","","","","","","tax","refund","+914","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_AB-IC-7","157","","7%","7% IC G","7.0","percent","purchase","tax_group_7","False","base","invoice","+915","",""
"","","","","","","","","","","tax","invoice","-916","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-915","",""
"","","","","","","","","","","tax","refund","+916","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2011_tax_AB-IC-3","158","","3%","3% IC G","3.0","percent","purchase","tax_group_3","","base","invoice","+049","",""
"","","","","","","","","","","tax","invoice","-054","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-049","",""
"","","","","","","","","","","tax","refund","+054","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_AB-IC-8","164","","8%","8% IC G","8.0","percent","purchase","tax_group_8","","base","invoice","+715","",""
"","","","","","","","","","","tax","invoice","-716","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-715","",""
"","","","","","","","","","","tax","refund","+716","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2011_tax_AB-PA-0","167","","0%","0% G","0.0","percent","purchase","tax_group_0","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_AB-PA-14","169","","14%","14% G","14.0","percent","purchase","tax_group_14","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AB-PA-17","101","","17%","17% G","17.0","percent","purchase","tax_group_17","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AB-PA-16","102","","16%","16% G","16.0","percent","purchase","tax_group_16","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AB-PA-13","103","","13%","13% G","13.0","percent","purchase","tax_group_13","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AB-PA-8","174","","8%","8% G","8.0","percent","purchase","tax_group_8","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AB-PA-7","104","","7%","7% G","7.0","percent","purchase","tax_group_7","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_AB-PA-3","172","","3%","3% G","3.0","percent","purchase","tax_group_3","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_AP-EC-0","175","","0%","0% EX EC S","0.0","percent","purchase","tax_group_0","","base","invoice","+445","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-445","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_AP-EC-14","179","","14%","14% EC S","14.0","percent","purchase","tax_group_14","","base","invoice","+753","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+754","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-753","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-754","","lu_2020_account_421611"
"lu_2015_tax_AP-EC-17","185","","17%","17% EC S","17.0","percent","purchase","tax_group_17","","base","invoice","+751","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+752","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-751","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-752","","lu_2020_account_421611"
"lu_2015_tax_AP-EC-16","185","","16%","16% EC S","16.0","percent","purchase","tax_group_16","False","base","invoice","+951","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+952","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-951","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-952","","lu_2020_account_421611"
"lu_2015_tax_AP-EC-13","186","","13%","13% EC S","13.0","percent","purchase","tax_group_13","False","base","invoice","+953","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+954","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-953","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-954","","lu_2020_account_421611"
"lu_2015_tax_AP-EC-7","187","","7%","7% EC S","7.0","percent","purchase","tax_group_7","False","base","invoice","+955","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+956","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-955","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-956","","lu_2020_account_421611"
"lu_2011_tax_AP-EC-3","188","","3%","3% EC S","3.0","percent","purchase","tax_group_3","","base","invoice","+441","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+442","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-441","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-442","","lu_2020_account_421611"
"lu_2015_tax_AP-EC-8","194","","8%","8% EC S","8.0","percent","purchase","tax_group_8","","base","invoice","+755","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+756","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-755","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-756","","lu_2020_account_421611"
"lu_2011_tax_AP-IC-0","197","","0%","0% EX IC S","0.0","percent","purchase","tax_group_0","","base","invoice","+435","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-435","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_AP-IC-14","201","","14%","14% IC S","14.0","percent","purchase","tax_group_14","","base","invoice","+743","",""
"","","","","","","","","","","tax","invoice","-744","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-743","",""
"","","","","","","","","","","tax","refund","+744","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_AP-IC-17","207","","17%","17% IC S","17.0","percent","purchase","tax_group_17","","base","invoice","+741","",""
"","","","","","","","","","","tax","invoice","-742","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-741","",""
"","","","","","","","","","","tax","refund","+742","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_AP-IC-16","208","","16%","16% IC S","16.0","percent","purchase","tax_group_16","False","base","invoice","+941","",""
"","","","","","","","","","","tax","invoice","-942","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-941","",""
"","","","","","","","","","","tax","refund","+942","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_AP-IC-13","209","","13%","13% IC S","13.0","percent","purchase","tax_group_13","False","base","invoice","+943","",""
"","","","","","","","","","","tax","invoice","-944","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-943","",""
"","","","","","","","","","","tax","refund","+944","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_AP-IC-7","210","","7%","7% IC S","7.0","percent","purchase","tax_group_7","False","base","invoice","+945","",""
"","","","","","","","","","","tax","invoice","-946","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-945","",""
"","","","","","","","","","","tax","refund","+946","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2011_tax_AP-IC-3","211","","3%","3% IC S","3.0","percent","purchase","tax_group_3","","base","invoice","+431","",""
"","","","","","","","","","","tax","invoice","-432","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-431","",""
"","","","","","","","","","","tax","refund","+432","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_AP-IC-8","216","","8%","8% IC S","8.0","percent","purchase","tax_group_8","","base","invoice","+745","",""
"","","","","","","","","","","tax","invoice","-746","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-745","",""
"","","","","","","","","","","tax","refund","+746","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2011_tax_AP-PA-0","219","","0%","0% S","0.0","percent","purchase","tax_group_0","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_AP-PA-14","221","","14%","14% S","14.0","percent","purchase","tax_group_14","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AP-PA-17","223","","17%","17% S","17.0","percent","purchase","tax_group_17","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AP-PA-16","223","","16%","16% S","16.0","percent","purchase","tax_group_16","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AP-PA-13","223","","13%","13% S","13.0","percent","purchase","tax_group_13","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AP-PA-7","223","","7%","7% S","7.0","percent","purchase","tax_group_7","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_AP-PA-3","224","","3%","3% S","3.0","percent","purchase","tax_group_3","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_AP-PA-8","226","","8%","8% S","8.0","percent","purchase","tax_group_8","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_FB-EC-0","227","","0%","0% EX EC E G","0.0","percent","purchase","tax_group_0","False","base","invoice","+195","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-195","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_FB-EC-14","231","","14%","14% EC E G","14.0","percent","purchase","tax_group_14","False","base","invoice","+723","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+724","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-723","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-724","","lu_2020_account_421611"
"lu_2015_tax_FB-EC-17","237","","17%","17% EC E G","17.0","percent","purchase","tax_group_17","False","base","invoice","+721","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+722","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-721","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-722","","lu_2020_account_421611"
"lu_2015_tax_FB-EC-16","238","","16%","16% EC E G","16.0","percent","purchase","tax_group_16","False","base","invoice","+921","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+922","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-921","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-922","","lu_2020_account_421611"
"lu_2015_tax_FB-EC-13","239","","13%","13% EC E G","13.0","percent","purchase","tax_group_13","False","base","invoice","+923","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+924","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-923","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-924","","lu_2020_account_421611"
"lu_2015_tax_FB-EC-7","240","","7%","7% EC E G","7.0","percent","purchase","tax_group_7","False","base","invoice","+925","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+926","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-925","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-926","","lu_2020_account_421611"
"lu_2011_tax_FB-EC-3","241","","3%","3% EC E G","3.0","percent","purchase","tax_group_3","False","base","invoice","+059","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+068","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-059","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-068","","lu_2020_account_421611"
"lu_2015_tax_FB-EC-8","246","","8%","8% EC E G","8.0","percent","purchase","tax_group_8","False","base","invoice","+725","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+726","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-725","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-726","","lu_2020_account_421611"
"lu_2015_tax_FB-ECP-0","249","","0%","0% EX EC(P) E G","0.0","percent","purchase","tax_group_0","False","base","invoice","+196","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-196","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_FB-ECP-14","253","","14%","14% EC(P) E G","14.0","percent","purchase","tax_group_14","False","base","invoice","+733","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+734","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-733","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-734","","lu_2020_account_421611"
"lu_2015_tax_FB-ECP-17","259","","17%","17% EC(P) E G","17.0","percent","purchase","tax_group_17","False","base","invoice","+731","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+732","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-731","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-732","","lu_2020_account_421611"
"lu_2015_tax_FB-ECP-16","260","","16%","16% EC(P) E G","16.0","percent","purchase","tax_group_16","False","base","invoice","+931","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+932","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-931","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-932","","lu_2020_account_421611"
"lu_2015_tax_FB-ECP-13","261","","13%","13% EC(P) E G","13.0","percent","purchase","tax_group_13","False","base","invoice","+933","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+934","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-933","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-934","","lu_2020_account_421611"
"lu_2015_tax_FB-ECP-7","262","","7%","7% EC(P) E G","7.0","percent","purchase","tax_group_7","False","base","invoice","+935","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+936","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-935","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-936","","lu_2020_account_421611"
"lu_2015_tax_FB-ECP-3","263","","3%","3% EC(P) E G","3.0","percent","purchase","tax_group_3","False","base","invoice","+063","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+073","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-063","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-073","","lu_2020_account_421611"
"lu_2015_tax_FB-ECP-8","268","","8%","8% EC(P) E G","8.0","percent","purchase","tax_group_8","False","base","invoice","+735","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+736","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-735","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-736","","lu_2020_account_421611"
"lu_2011_tax_FB-IC-0","271","","0%","0% EX IC E G","0.0","percent","purchase","tax_group_0","False","base","invoice","+194","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-194","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_FB-IC-14","275","","14%","14% IC E G","14.0","percent","purchase","tax_group_14","False","base","invoice","+713","",""
"","","","","","","","","","","tax","invoice","-714","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-713","",""
"","","","","","","","","","","tax","refund","+714","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_FB-IC-17","281","","17%","17% IC E G","17.0","percent","purchase","tax_group_17","False","base","invoice","+711","",""
"","","","","","","","","","","tax","invoice","-712","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-711","",""
"","","","","","","","","","","tax","refund","+712","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_FB-IC-16","282","","16%","16% IC E G","16.0","percent","purchase","tax_group_16","False","base","invoice","+911","",""
"","","","","","","","","","","tax","invoice","-912","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-911","",""
"","","","","","","","","","","tax","refund","+912","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_FB-IC-13","283","","13%","13% IC E G","13.0","percent","purchase","tax_group_13","False","base","invoice","+913","",""
"","","","","","","","","","","tax","invoice","-914","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-913","",""
"","","","","","","","","","","tax","refund","+914","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_FB-IC-7","284","","7%","7% IC E G","7.0","percent","purchase","tax_group_7","False","base","invoice","+915","",""
"","","","","","","","","","","tax","invoice","-916","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-915","",""
"","","","","","","","","","","tax","refund","+916","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2011_tax_FB-IC-3","285","","3%","3% IC E G","3.0","percent","purchase","tax_group_3","False","base","invoice","+049","",""
"","","","","","","","","","","tax","invoice","-054","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-049","",""
"","","","","","","","","","","tax","refund","+054","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_FB-IC-8","290","","8%","8% IC E G","8.0","percent","purchase","tax_group_8","False","base","invoice","+715","",""
"","","","","","","","","","","tax","invoice","-716","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-715","",""
"","","","","","","","","","","tax","refund","+716","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2011_tax_FB-PA-0","293","","0%","0% E G","0.0","percent","purchase","tax_group_0","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_FB-PA-14","295","","14%","14% E G","14.0","percent","purchase","tax_group_14","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FB-PA-17","297","","17%","17% E G","17.0","percent","purchase","tax_group_17","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FB-PA-16","298","","16%","16% E G","16.0","percent","purchase","tax_group_16","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FB-PA-13","299","","13%","13% E G","13.0","percent","purchase","tax_group_13","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FB-PA-7","300","","7%","7% E G","7.0","percent","purchase","tax_group_7","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_FB-PA-3","301","","3%","3% E G","3.0","percent","purchase","tax_group_3","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FB-PA-8","302","","8%","8% E G","8.0","percent","purchase","tax_group_8","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_FP-EC-0","303","","0%","0% EC E S","0.0","percent","purchase","tax_group_0","False","base","invoice","+445","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-445","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_FP-EC-14","305","","14%","14% EC E S","14.0","percent","purchase","tax_group_14","False","base","invoice","+753","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+754","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-753","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-754","","lu_2020_account_421611"
"lu_2015_tax_FP-EC-17","311","","17%","17% EC E S","17.0","percent","purchase","tax_group_17","False","base","invoice","+751","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+752","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-751","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-752","","lu_2020_account_421611"
"lu_2015_tax_FP-EC-16","312","","16%","16% EC E S","16.0","percent","purchase","tax_group_16","False","base","invoice","+951","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+952","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-951","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-952","","lu_2020_account_421611"
"lu_2015_tax_FP-EC-13","313","","13%","13% EC E S","13.0","percent","purchase","tax_group_13","False","base","invoice","+953","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+954","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-953","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-954","","lu_2020_account_421611"
"lu_2015_tax_FP-EC-7","314","","7%","7% EC E S","7.0","percent","purchase","tax_group_7","False","base","invoice","+955","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+956","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-955","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-956","","lu_2020_account_421611"
"lu_2011_tax_FP-EC-3","315","","3%","3% EC E S","3.0","percent","purchase","tax_group_3","False","base","invoice","+441","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+442","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-441","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-442","","lu_2020_account_421611"
"lu_2015_tax_FP-EC-8","320","","8%","8% EC E S","8.0","percent","purchase","tax_group_8","False","base","invoice","+755","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+756","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-755","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-756","","lu_2020_account_421611"
"lu_2011_tax_FP-IC-0","323","","0%","0% EX IC E S","0.0","percent","purchase","tax_group_0","False","base","invoice","+435","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-435","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_FP-IC-14","327","","14%","14% IC E S","14.0","percent","purchase","tax_group_14","False","base","invoice","+743","",""
"","","","","","","","","","","tax","invoice","-744","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-743","",""
"","","","","","","","","","","tax","refund","+744","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_FP-IC-17","333","","17%","17% IC E S","17.0","percent","purchase","tax_group_17","False","base","invoice","+741","",""
"","","","","","","","","","","tax","invoice","-742","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-741","",""
"","","","","","","","","","","tax","refund","+742","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_FP-IC-16","334","","16%","16% IC E S","16.0","percent","purchase","tax_group_16","False","base","invoice","+941","",""
"","","","","","","","","","","tax","invoice","-942","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-941","",""
"","","","","","","","","","","tax","refund","+942","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_FP-IC-13","335","","13%","13% IC E S","13.0","percent","purchase","tax_group_13","False","base","invoice","+943","",""
"","","","","","","","","","","tax","invoice","-944","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-943","",""
"","","","","","","","","","","tax","refund","+944","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_FP-IC-7","336","","7%","7% IC E S","7.0","percent","purchase","tax_group_7","False","base","invoice","+945","",""
"","","","","","","","","","","tax","invoice","-946","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-945","",""
"","","","","","","","","","","tax","refund","+946","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2011_tax_FP-IC-3","337","","3%","3% IC E S","3.0","percent","purchase","tax_group_3","False","base","invoice","+431","",""
"","","","","","","","","","","tax","invoice","-432","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-431","",""
"","","","","","","","","","","tax","refund","+432","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_FP-IC-8","342","","8%","8% IC E S","8.0","percent","purchase","tax_group_8","False","base","invoice","+745","",""
"","","","","","","","","","","tax","invoice","-746","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-745","",""
"","","","","","","","","","","tax","refund","+746","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2011_tax_FP-PA-0","345","","0%","0% E S","0.0","percent","purchase","tax_group_0","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_FP-PA-14","347","","14%","14% E S","14.0","percent","purchase","tax_group_14","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FP-PA-17","349","","17%","17% E S","17.0","percent","purchase","tax_group_17","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FP-PA-16","350","","16%","16% E S","16.0","percent","purchase","tax_group_16","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FP-PA-13","351","","13%","13% E S","13.0","percent","purchase","tax_group_13","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FP-PA-7","352","","7%","7% E S","7.0","percent","purchase","tax_group_7","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_FP-PA-3","353","","3%","3% E S","3.0","percent","purchase","tax_group_3","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_FP-PA-8","354","","8%","8% E S","8.0","percent","purchase","tax_group_8","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_IB-EC-0","355","","0%","0% EC IG","0.0","percent","purchase","tax_group_0","False","base","invoice","+195","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-195","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_IB-EC-14","357","","14%","14% EC IG","14.0","percent","purchase","tax_group_14","False","base","invoice","+723","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+724","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-723","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-724","","lu_2020_account_421611"
"lu_2015_tax_IB-EC-17","363","","17%","17% EC IG","17.0","percent","purchase","tax_group_17","False","base","invoice","+721","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+722","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-721","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-722","","lu_2020_account_421611"
"lu_2015_tax_IB-EC-16","364","","16%","16% EC IG","16.0","percent","purchase","tax_group_16","False","base","invoice","+921","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+922","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-921","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-922","","lu_2020_account_421611"
"lu_2015_tax_IB-EC-13","365","","13%","13% EC IG","13.0","percent","purchase","tax_group_13","False","base","invoice","+923","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+924","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-923","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-924","","lu_2020_account_421611"
"lu_2015_tax_IB-EC-7","366","","7%","7% EC IG","7.0","percent","purchase","tax_group_7","False","base","invoice","+925","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+926","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-925","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-926","","lu_2020_account_421611"
"lu_2011_tax_IB-EC-3","367","","3%","3% EC IG","3.0","percent","purchase","tax_group_3","False","base","invoice","+059","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+068","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-059","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-068","","lu_2020_account_421611"
"lu_2015_tax_IB-EC-8","372","","8%","8% EC IG","8.0","percent","purchase","tax_group_8","False","base","invoice","+725","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+726","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-725","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-726","","lu_2020_account_421611"
"lu_2015_tax_IB-ECP-0","375","","5%","0% EC(P) IG","0.0","percent","purchase","tax_group_0","False","base","invoice","+196","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-196","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_IB-ECP-14","379","","14%","14% EC(P) IG","14.0","percent","purchase","tax_group_14","False","base","invoice","+733","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+734","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-733","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-734","","lu_2020_account_421611"
"lu_2015_tax_IB-ECP-17","385","","17%","17% EC(P) IG","17.0","percent","purchase","tax_group_17","False","base","invoice","+731","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+732","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-731","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-732","","lu_2020_account_421611"
"lu_2015_tax_IB-ECP-16","386","","16%","16% EC(P) IG","16.0","percent","purchase","tax_group_16","False","base","invoice","+931","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+932","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-931","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-932","","lu_2020_account_421611"
"lu_2015_tax_IB-ECP-13","387","","13%","13% EC(P) IG","13.0","percent","purchase","tax_group_13","False","base","invoice","+933","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+934","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-933","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-934","","lu_2020_account_421611"
"lu_2015_tax_IB-ECP-7","388","","7%","7% EC(P) IG","7.0","percent","purchase","tax_group_7","False","base","invoice","+935","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+936","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-935","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-936","","lu_2020_account_421611"
"lu_2015_tax_IB-ECP-3","389","","3%","3% EC(P) IG","3.0","percent","purchase","tax_group_3","False","base","invoice","+063","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+073","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-063","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-073","","lu_2020_account_421611"
"lu_2015_tax_IB-ECP-8","394","","8%","8% EC(P) IG","8.0","percent","purchase","tax_group_8","False","base","invoice","+735","",""
"","","","","","","","","","","tax","invoice","-460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+736","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-735","",""
"","","","","","","","","","","tax","refund","+460","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-736","","lu_2020_account_421611"
"lu_2011_tax_IB-IC-0","397","","0%","0% IC IG","0.0","percent","purchase","tax_group_0","False","base","invoice","+194","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-194","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_IB-IC-14","401","","14%","14% IC IG","14.0","percent","purchase","tax_group_14","False","base","invoice","+713","",""
"","","","","","","","","","","tax","invoice","-714","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-713","",""
"","","","","","","","","","","tax","refund","+714","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_IB-IC-17","407","","17%","17% IC IG","17.0","percent","purchase","tax_group_17","False","base","invoice","+711","",""
"","","","","","","","","","","tax","invoice","-712","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-711","",""
"","","","","","","","","","","tax","refund","+712","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_IB-IC-16","407","","16%","16% IC IG","16.0","percent","purchase","tax_group_16","False","base","invoice","+911","",""
"","","","","","","","","","","tax","invoice","-912","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-911","",""
"","","","","","","","","","","tax","refund","+912","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_IB-IC-13","408","","13%","13% IC IG","13.0","percent","purchase","tax_group_13","False","base","invoice","+913","",""
"","","","","","","","","","","tax","invoice","-914","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-913","",""
"","","","","","","","","","","tax","refund","+914","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_IB-IC-7","409","","7%","7% IC IG","7.0","percent","purchase","tax_group_7","False","base","invoice","+915","",""
"","","","","","","","","","","tax","invoice","-916","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-915","",""
"","","","","","","","","","","tax","refund","+916","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2011_tax_IB-IC-3","410","","3%","3% IC IG","3.0","percent","purchase","tax_group_3","False","base","invoice","+049","",""
"","","","","","","","","","","tax","invoice","-054","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-049","",""
"","","","","","","","","","","tax","refund","+054","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2015_tax_IB-IC-8","416","","8%","8% IC IG","8.0","percent","purchase","tax_group_8","False","base","invoice","+715","",""
"","","","","","","","","","","tax","invoice","-716","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+459","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-715","",""
"","","","","","","","","","","tax","refund","+716","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-459","","lu_2020_account_421611"
"lu_2011_tax_IB-PA-0","419","","0%","0% IG","0.0","percent","purchase","tax_group_0","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_IB-PA-14","421","","14%","14% IG","14.0","percent","purchase","tax_group_14","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IB-PA-17","423","","17%","17% IG","17.0","percent","purchase","tax_group_17","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IB-PA-16","424","","16%","16% IG","16.0","percent","purchase","tax_group_16","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IB-PA-13","425","","13%","13% IG","13.0","percent","purchase","tax_group_13","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IB-PA-7","426","","7%","7% IG","7.0","percent","purchase","tax_group_7","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_IB-PA-3","427","","3%","3% IG","3.0","percent","purchase","tax_group_3","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IB-PA-8","428","","8%","8% IG","8.0","percent","purchase","tax_group_8","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_IP-EC-0","429","","0%","0% EC IS","0.0","percent","purchase","tax_group_0","False","base","invoice","+445","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-445","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_IP-EC-14","431","","14%","14% EC IS","14.0","percent","purchase","tax_group_14","False","base","invoice","+753","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+754","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-753","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-754","","lu_2020_account_421611"
"lu_2015_tax_IP-EC-17","437","","17%","17% EC IS","17.0","percent","purchase","tax_group_17","False","base","invoice","+751","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+752","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-751","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-752","","lu_2020_account_421611"
"lu_2015_tax_IP-EC-16","438","","16%","16% EC IS","16.0","percent","purchase","tax_group_16","False","base","invoice","+951","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+952","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-951","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-952","","lu_2020_account_421611"
"lu_2015_tax_IP-EC-13","439","","13%","13% EC IS","13.0","percent","purchase","tax_group_13","False","base","invoice","+953","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+954","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-953","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-954","","lu_2020_account_421611"
"lu_2015_tax_IP-EC-7","440","","7%","7% EC IS","7.0","percent","purchase","tax_group_7","False","base","invoice","+955","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+956","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-955","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-956","","lu_2020_account_421611"
"lu_2011_tax_IP-EC-3","441","","3%","3% EC IS","3.0","percent","purchase","tax_group_3","False","base","invoice","+441","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+442","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-441","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-442","","lu_2020_account_421611"
"lu_2015_tax_IP-EC-8","446","","8%","8% EC IS","8.0","percent","purchase","tax_group_8","False","base","invoice","+755","",""
"","","","","","","","","","","tax","invoice","-461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+756","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-755","",""
"","","","","","","","","","","tax","refund","+461","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-756","","lu_2020_account_421611"
"lu_2011_tax_IP-IC-0","449","","0%","0% IC IS","0.0","percent","purchase","tax_group_0","False","base","invoice","+435","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-435","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_IP-IC-14","453","","14%","14% IC IS","14.0","percent","purchase","tax_group_14","False","base","invoice","+743","",""
"","","","","","","","","","","tax","invoice","-744","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-743","",""
"","","","","","","","","","","tax","refund","+744","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_IP-IC-17","459","","17%","17% IC IS","17.0","percent","purchase","tax_group_17","False","base","invoice","+741","",""
"","","","","","","","","","","tax","invoice","-742","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-741","",""
"","","","","","","","","","","tax","refund","+742","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_IP-IC-16","459","","16%","16% IC IS","16.0","percent","purchase","tax_group_16","False","base","invoice","+941","",""
"","","","","","","","","","","tax","invoice","-942","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-941","",""
"","","","","","","","","","","tax","refund","+942","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_IP-IC-13","460","","13%","13% IC IS","13.0","percent","purchase","tax_group_13","False","base","invoice","+943","",""
"","","","","","","","","","","tax","invoice","-944","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-943","",""
"","","","","","","","","","","tax","refund","+944","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_IP-IC-7","461","","7%","7% IC IS","7.0","percent","purchase","tax_group_7","False","base","invoice","+945","",""
"","","","","","","","","","","tax","invoice","-946","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-945","",""
"","","","","","","","","","","tax","refund","+946","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2011_tax_IP-IC-3","462","","3%","3% IC IS","3.0","percent","purchase","tax_group_3","False","base","invoice","+431","",""
"","","","","","","","","","","tax","invoice","-432","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-431","",""
"","","","","","","","","","","tax","refund","+432","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2015_tax_IP-IC-8","468","","8%","8% IC IS","8.0","percent","purchase","tax_group_8","False","base","invoice","+745","",""
"","","","","","","","","","","tax","invoice","-746","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","invoice","+461","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","-745","",""
"","","","","","","","","","","tax","refund","+746","-100","lu_2020_account_461411"
"","","","","","","","","","","tax","refund","-461","","lu_2020_account_421611"
"lu_2011_tax_IP-PA-0","471","","0%","0% IS","0.0","percent","purchase","tax_group_0","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_IP-PA-14","473","","14%","14% IS","14.0","percent","purchase","tax_group_14","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IP-PA-17","475","","17%","17% IS","17.0","percent","purchase","tax_group_17","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IP-PA-16","476","","16%","16% IS","16.0","percent","purchase","tax_group_16","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IP-PA-13","477","","13%","13% IS","13.0","percent","purchase","tax_group_13","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IP-PA-7","478","","7%","7% IS","7.0","percent","purchase","tax_group_7","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2011_tax_IP-PA-3","479","","3%","3% IS","3.0","percent","purchase","tax_group_3","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_IP-PA-8","480","","8%","8% IS","8.0","percent","purchase","tax_group_8","False","base","invoice","","",""
"","","","","","","","","","","tax","invoice","+458","","lu_2020_account_421611"
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","-458","","lu_2020_account_421611"
"lu_2015_tax_V-ART-43_60b","489","0-E-Art.43&60b","0%","0% E 43&60b","0.0","percent","sale","tax_group_0","False","base","invoice","+015","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-015","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_V-ART-44_56q","480","0-E-Art.43&60b","0%","0% E 44&56q","0.0","percent","sale","tax_group_0","False","base","invoice","+016","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-016","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VB-EC-0","481","","0%","0% EC S G","0.0","percent","sale","tax_group_0","False","base","invoice","+014","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-014","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VB-EC-Tab","482","","0%","0% EC ST G","0.0","percent","sale","tax_group_0","False","base","invoice","+017","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-017","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VB-IC-0","483","","0%","0% IC S G","0.0","percent","sale","tax_group_0","","base","invoice","+457","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-457","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VB-IC-Tab","484","","0%","0% IC ST G","0.0","percent","sale","tax_group_0","False","base","invoice","+017","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-017","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VB-PA-0","485","","0%","0% G","0.0","percent","sale","tax_group_0","False","base","invoice","+033","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-033","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_VB-PA-14","487","","14%","14% G","14.0","percent","sale","tax_group_14","False","base","invoice","+703","",""
"","","","","","","","","","","tax","invoice","+704","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-703","",""
"","","","","","","","","","","tax","refund","-704","","lu_2020_account_461411"
"lu_2015_tax_VB-PA-17","502","","17%","17% G","17.0","percent","sale","tax_group_17","False","base","invoice","+701","",""
"","","","","","","","","","","tax","invoice","+702","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-701","",""
"","","","","","","","","","","tax","refund","-702","","lu_2020_account_461411"
"lu_2015_tax_VB-PA-16","503","","16%","16% G","16.0","percent","sale","tax_group_16","False","base","invoice","+901","",""
"","","","","","","","","","","tax","invoice","+902","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-901","",""
"","","","","","","","","","","tax","refund","-902","","lu_2020_account_461411"
"lu_2015_tax_VB-PA-13","504","","13%","13% G","13.0","percent","sale","tax_group_13","False","base","invoice","+903","",""
"","","","","","","","","","","tax","invoice","+904","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-903","",""
"","","","","","","","","","","tax","refund","-904","","lu_2020_account_461411"
"lu_2015_tax_VB-PA-7","505","","7%","7% G","7.0","percent","sale","tax_group_7","False","base","invoice","+905","",""
"","","","","","","","","","","tax","invoice","+906","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-905","",""
"","","","","","","","","","","tax","refund","-906","","lu_2020_account_461411"
"lu_2011_tax_VB-PA-3","490","","3%","3% G","3.0","percent","sale","tax_group_3","False","base","invoice","+031","",""
"","","","","","","","","","","tax","invoice","+040","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-031","",""
"","","","","","","","","","","tax","refund","-040","","lu_2020_account_461411"
"lu_2015_tax_VB-PA-8","492","","8%","8% G","8.0","percent","sale","tax_group_8","False","base","invoice","+705","",""
"","","","","","","","","","","tax","invoice","+706","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-705","",""
"","","","","","","","","","","tax","refund","-706","","lu_2020_account_461411"
"lu_2011_tax_VB-PA-Tab","493","","0%","0% ST G","0.0","percent","sale","tax_group_0","False","base","invoice","+017","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-017","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_VB-TR-0","494","","0%","0% ICT G","0.0","percent","sale","tax_group_0","","base","invoice","+018","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-018","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VP-EC-0","495","","0%","0% EC S","0.0","percent","sale","tax_group_0","","base","invoice","+019","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-019","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VP-IC-0","496","","0%","0% IC S","0.0","percent","sale","tax_group_0","","base","invoice","+423","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-423","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VP-IC-EX","497","","0%","0% EX IC S","0.0","percent","sale","tax_group_0","","base","invoice","+424","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-424","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2011_tax_VP-PA-0","498","","0%","0% S","0.0","percent","sale","tax_group_0","","base","invoice","+033","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-033","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_VP-PA-14","500","","14%","14% S","14.0","percent","sale","tax_group_14","","base","invoice","+703","",""
"","","","","","","","","","","tax","invoice","+704","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-703","",""
"","","","","","","","","","","tax","refund","-704","","lu_2020_account_461411"
"lu_2015_tax_VP-PA-17","479","","17%","17% S","17.0","percent","sale","tax_group_17","","base","invoice","+701","",""
"","","","","","","","","","","tax","invoice","+702","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-701","",""
"","","","","","","","","","","tax","refund","-702","","lu_2020_account_461411"
"lu_2015_tax_VP-PA-16","480","","16%","16% S","16.0","percent","sale","tax_group_16","False","base","invoice","+901","",""
"","","","","","","","","","","tax","invoice","+902","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-901","",""
"","","","","","","","","","","tax","refund","-902","","lu_2020_account_461411"
"lu_2015_tax_VP-PA-13","481","","13%","13% S","13.0","percent","sale","tax_group_13","False","base","invoice","+903","",""
"","","","","","","","","","","tax","invoice","+904","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-903","",""
"","","","","","","","","","","tax","refund","-904","","lu_2020_account_461411"
"lu_2015_tax_VP-PA-7","482","","7%","7% S","7.0","percent","sale","tax_group_7","False","base","invoice","+905","",""
"","","","","","","","","","","tax","invoice","+906","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-905","",""
"","","","","","","","","","","tax","refund","-906","","lu_2020_account_461411"
"lu_2011_tax_VP-PA-3","503","","3%","3% S","3.0","percent","sale","tax_group_3","","base","invoice","+031","",""
"","","","","","","","","","","tax","invoice","+040","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-031","",""
"","","","","","","","","","","tax","refund","-040","","lu_2020_account_461411"
"lu_2015_tax_VP-PA-8","505","","8%","8% S","8.0","percent","sale","tax_group_8","","base","invoice","+705","",""
"","","","","","","","","","","tax","invoice","+706","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-705","",""
"","","","","","","","","","","tax","refund","-706","","lu_2020_account_461411"
"lu_2015_tax_SANS","506","Tax free","0%","0%","0.0","percent","purchase","tax_group_0","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_SANS_sale","507","","0%","0% S S","0.0","percent","sale","tax_group_0","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","",""
"lu_2015_tax_ATN_sale","510","","17%","17% ATN","17.0","percent","sale","tax_group_17","","base","invoice","+456||+701","",""
"","","","","","","","","","","tax","invoice","+702","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-456||-701","",""
"","","","","","","","","","","tax","refund","-702","","lu_2020_account_461411"
"lu_2015_tax_ATN_sale_16","511","","16%","16% ATN","16.0","percent","sale","tax_group_16","False","base","invoice","+456||+901","",""
"","","","","","","","","","","tax","invoice","+902","","lu_2020_account_461411"
"","","","","","","","","","","base","refund","-456||-901","",""
"","","","","","","","","","","tax","refund","-902","","lu_2020_account_461411"

```

## File: data\template\account.tax.group-lu.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","advance_tax_payment_account_id","name@fr","name@de"
"tax_group_0","0% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 0%","MwSt. 0%"
"tax_group_3","3% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 3%","MwSt. 3%"
"tax_group_6","6% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 6%","MwSt. 6%"
"tax_group_7","7% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 7%","MwSt. 7%"
"tax_group_8","8% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 8%","MwSt. 8%"
"tax_group_10","10% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 10%","MwSt. 10%"
"tax_group_12","12% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 12%","MwSt. 12%"
"tax_group_13","13% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 13%","MwSt. 13%"
"tax_group_14","14% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 14%","MwSt. 14%"
"tax_group_15","15% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 15%","MwSt. 15%"
"tax_group_16","16% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 16%","MwSt. 16%"
"tax_group_17","17% VAT","base.lu","lu_2011_account_421612","lu_2011_account_461412","lu_2011_account_421613","TVA 17%","MwSt. 17%"

```

## File: i18n_extra\l10n_lu.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_lu
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server saas~13.3\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2020-06-19 13:53+0000\n"
"PO-Revision-Date: 2020-06-19 13:53+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1a_overall_turnover
msgid "012 - Overall turnover"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_2_export
msgid "014 - Exports"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_3_other_exemptions_art_43
msgid "015 - Other exemptions"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater
msgid "016 - Other exemptions"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_5_manufactured_tobacco_vat_collected
msgid ""
"017 - Manufactured tobacco whose VAT was collected at the source or at the "
"exit of the tax..."
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_6_a_subsequent_to_intra_community
msgid ""
"018 - Supply, subsequent to intra-Community acquisitions of goods, in the "
"context of triangular transactions, when the customer identified,..."
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_6_d_supplies_other_referred
msgid ""
"019 - Other supplies carried out (for which the place of supply is) abroad"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_exemptions_deductible_amounts
msgid "021 - Exemptions and deductible amounts"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1c_taxable_turnover
msgid "022 - Taxable turnover"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_3
msgid "031 - base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_0
msgid "033 - base 0%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_breakdown_taxable_turnover_base
msgid "037 - Breakdown of taxable turnover – base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_tax_3
msgid "040 - tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_breakdown_taxable_turnover_tax
msgid "046 - Breakdown of taxable turnover – tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_3
msgid "049 - base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_intra_community_acqui_of_goods_base
msgid "051 - Intra-Community acquisitions of goods – base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_tax_3
msgid "054 - tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_intra_community_acquisitions_goods_tax
msgid "056 - Intra-Community acquisitions of goods – tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_3
msgid "059 - for business purposes: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_3
msgid "063 - for non-business purposes: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_importation_of_goods_base
msgid "065 - Importation of goods – base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_3
msgid "068 - for business purposes: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_3
msgid "073 - for non-business purposes: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2h_total_tax_due
msgid "076 - Total tax due"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_4_due_respect_application_goods
msgid "090 - Due in respect of the application of goods for business purposes"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_6_paid_joint_several_guarantee
msgid "092 - Paid as joint and several guarantee"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_total_input_tax
msgid "093 - Total input tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3b1_rel_trans
msgid ""
"094 - relating to transactions which are exempt pursuant to articles 44 and "
"56quater"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3b2_ded_prop
msgid ""
"095 - where the deductible proportion determined in accordance to article 50"
" is applied"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3b2_input_tax_margin
msgid ""
"096 - Non recoverable input tax in accordance with Art. 56ter-1(7) and "
"56ter-2(7) (when applying the margin scheme)"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3b_total_input_tax_nd
msgid "097 - Total input tax non-deductible"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3c_total_input_tax_deductible
msgid "102 - Total input tax deductible"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_4a_total_tax_due
msgid "103 - Total tax due"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_4a_total_input_tax_deductible
msgid "104 - Total input tax deductible"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_4c_exceeding_amount
msgid "105 - Exceeding amount"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2c_acquisitions_triangular_transactions_base
msgid "152 - Acquisitions, in the context of triangular transactions – base"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_188
msgid ""
"188 - Appendix A - Expenses for other work carried out by third parties"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_190
msgid "190 - Appendix A - Car expenses"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_exempt
msgid "194 - base exempt"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_exempt
msgid "195 - for business purposes: base exempt"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_exempt
msgid "196 - for non-business purposes: base exempt"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_6_c_supplies_scope_special_arrangement
msgid ""
"226 - Supplies carried out within the scope of the special arrangement of "
"art. 56sexies"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2g_special_arrangement
msgid "227 - Special arrangement for tax suspension: adjustment"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_7_adjusted_tax_special_arrangement
msgid "228 - Adjusted tax - special arrangement for tax suspension"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_239
msgid "239 - Appendix A - Gross salaries"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_244
msgid "244 - Appendix A - Gross wages"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_247
msgid "247 - Appendix A - Occasional salaries"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_250
msgid ""
"250 - Appendix A - Compulsory social security contributions (employer's "
"share)"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_253
msgid "253 - Appendix A - Accident insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_260
msgid "260 - Appendix A - Staff travel and representation expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_269
msgid "269 - Appendix A - Accounting and bookkeeping fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_283
msgid "283 - Appendix A - Employer's travel and representation expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_285
msgid "285 - Appendix A - Electricity"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_289
msgid "289 - Appendix A - Gas"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_293
msgid "293 - Appendix A - Employer's travel and representation expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_301
msgid "301 - Appendix A - Telecommunications"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_305
msgid ""
"305 - Appendix A - Renting/leasing of immovable property with application of"
" VAT"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_307
msgid ""
"307 - Appendix A - Renting/leasing of immovable property with no application"
" of VAT"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_310
msgid ""
"310 - Appendix A - Renting/leasing of permanently installed equipment and "
"machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_316
msgid "316 - Appendix A - Property tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_324
msgid "324 - Appendix A - Business tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_326
msgid "326 - Appendix A - Interest paid for long-term debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_327
msgid "327 - Appendix A - Interest paid for short-term debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_328
msgid "328 - Appendix A - Other financial costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_330
msgid "330 - Appendix A - Stock and business equipment insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_331
msgid ""
"331 - Appendix A - Public and professional third party liability insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_332
msgid "332 - Appendix A - Office expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_336
msgid ""
"336 - Appendix A - Fees and subscriptions paid to professional associations "
"and learned societies"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_337
msgid "337 - Appendix A - Papers and periodicals for business purposes"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_343
msgid "343 - Appendix A - Shipping and transport expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_345
msgid "345 - Appendix A - Work clothes"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_347
msgid "347 - Appendix A - Advertising and publicity"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_349
msgid "349 - Appendix A - Packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_351
msgid "351 - Appendix A - Repair and maintenance of equipment and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_353
msgid "353 - Appendix A - Other repairs"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_355
msgid ""
"355 - Appendix A - New acquisitions (tools and equipment) if their cost can "
"be fully allocated to the year of acquisition or creation"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_358
msgid "358 - Appendix A - Custom (value)"
msgstr ""

#. module: l10n_lu
#: model:account.account.tag,name:l10n_lu.account_tag_appendix_361
msgid "361 - Appendix A - Total 'Appendix to Operational expenditures'"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_importation_of_goods_tax
msgid "407 - Importation of goods – tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_supply_of_service_for_customer
msgid ""
"409 - Supply of services for which the customer is liable for the payment of"
" VAT – base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax
msgid ""
"410 - Supply of services for which the customer is liable for the payment of"
" VAT – tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_7_inland_supplies_for_customer
msgid ""
"419 - Inland supplies for which the customer is liable for the payment of "
"VAT"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_6_b1_non_exempt_customer_vat
msgid ""
"423 - not exempt in the MS where the customer is liable for payment of VAT"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_6_b2_exempt_ms_customer
msgid "424 - exempt in the MS where the customer is identified"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_3
msgid "431 - not exempt within the territory: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_3
msgid "432 - not exempt within the territory: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_b_exempt
msgid "435 - exempt within the territory: exempt"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_base
msgid "436 - base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_3
msgid "441 - not established or residing within the Community: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_3
msgid "442 - not established or residing within the Community: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_exempt
msgid "445 - not established or residing within the Community: exempt"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1a_total_sale
msgid "454 - Total Sales / Receipts"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1a_app_goods_non_bus
msgid ""
"455 - Application of goods for non-business use and for business purposes"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1a_non_bus_gs
msgid "456 - Non-business use of goods and supply of services free of charge"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_1_intra_community_goods_pi_vat
msgid ""
"457 - Intra-Community supply of goods to persons identified for VAT purposes"
" in another Member State (MS)"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_1_invoiced_by_other_taxable_person
msgid "458 - Invoiced by other taxable persons for goods or services supplied"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_2_due_respect_intra_comm_goods
msgid "459 - Due in respect of intra-Community acquisitions of goods"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_3_due_paid_respect_importation_goods
msgid "460 - Due or paid in respect of importation of goods"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3a_5_due_under_reverse_charge
msgid "461 - Due under the reverse charge (see points II.E and F)"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax
msgid "462 - tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base
msgid "463 - base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax
msgid "464 - tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1a_telecom_service
msgid ""
"471 - Telecommunications services, radio and television broadcasting "
"services..."
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1a_other_sales
msgid "472 - Other sales / receipts"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_8_supplies_carried_out_domestic
msgid ""
"481 - Supplies carried out within the scope of the domestic SME scheme of "
"article 57bis (7)"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_1b_9_supplies_carried_out_cross_border
msgid ""
"482 - Supplies carried out within the scope of the cross-border SME scheme "
"of article 57quater "
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_17
msgid "701 - base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_tax_17
msgid "702 - tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_14
msgid "703 - base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_tax_14
msgid "704 - tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_8
msgid "705 - base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_tax_8
msgid "706 - tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_17
msgid "711 - base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_tax_17
msgid "712 - tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_14
msgid "713 - base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_tax_14
msgid "714 - tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_8
msgid "715 - base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_tax_8
msgid "716 - tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_manufactured_tobacco
msgid ""
"719 - of manufactured tobacco (VAT is collected at the exit of the tax "
"warehouse with excise duties)"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_17
msgid "721 - for business purposes: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_17
msgid "722 - for business purposes: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_14
msgid "723 - for business purposes: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_14
msgid "724 - for business purposes: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_8
msgid "725 - for business purposes: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_8
msgid "726 - for business purposes: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_manufactured_tobacco
msgid ""
"729 - of manufactured tobacco (VAT is collected at the exit of the tax "
"warehouse with excise duties)"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_17
msgid "731 - for non-business purposes: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_17
msgid "732 - for non-business purposes: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_14
msgid "733 - for non-business purposes: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_14
msgid "734 - for non-business purposes: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_8
msgid "735 - for non-business purposes: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_8
msgid "736 - for non-business purposes: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_17
msgid "741 - not exempt within the territory: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_17
msgid "742 - not exempt within the territory: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_14
msgid "743 - not exempt within the territory: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_14
msgid "744 - not exempt within the territory: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_8
msgid "745 - not exempt within the territory: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_8
msgid "746 - not exempt within the territory: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_17
msgid "751 - not established or residing within the Community: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_17
msgid "752 - not established or residing within the Community: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_14
msgid "753 - not established or residing within the Community: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_14
msgid "754 - not established or residing within the Community: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_8
msgid "755 - not established or residing within the Community: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_8
msgid "756 - not established or residing within the Community: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_3_base_17
msgid "761 - suppliers established within the territory: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax_17
msgid "762 - suppliers established within the territory: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base_8
msgid "763 - base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_8
msgid "764 - tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_3_base
msgid "765 - base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax
msgid "766 - tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base
msgid ""
"767 - Supply of goods for which the purchaser is liable for the payment of "
"VAT - base"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax
msgid ""
"768 - Supply of goods for which the purchaser is liable for the payment of "
"VAT - tax"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base_17
msgid "769 - base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_17
msgid "770 - tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_16
msgid "901 - base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_tax_16
msgid "902 - tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_13
msgid "903 - base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_tax_13
msgid "904 - tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_base_7
msgid "905 - base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2a_tax_7
msgid "906 - tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_16
msgid "911 - base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_tax_16
msgid "912 - tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_13
msgid "913 - base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_tax_13
msgid "914 - tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_base_7
msgid "915 - base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2b_tax_7
msgid "916 - tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_16
msgid "921 - for business purposes: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_16
msgid "922 - for business purposes: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_13
msgid "923 - for business purposes: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_13
msgid "924 - for business purposes: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_7
msgid "925 - for business purposes: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_7
msgid "926 - for business purposes: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_16
msgid "931 - for non-business purposes: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_16
msgid "932 - for non-business purposes: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_13
msgid "933 - for non-business purposes: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_13
msgid "934 - for non-business purposes: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_7
msgid "935 - for non-business purposes: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_7
msgid "936 - for non-business purposes: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_16
msgid "941 - not exempt within the territory: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_16
msgid "942 - not exempt within the territory: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_13
msgid "943 - not exempt within the territory: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_13
msgid "944 - not exempt within the territory: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_7
msgid "945 - not exempt within the territory: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_7
msgid "946 - not exempt within the territory: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_16
msgid "951 - not established or residing within the Community: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_16
msgid "952 - not established or residing within the Community: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_13
msgid "953 - not established or residing within the Community: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_13
msgid "954 - not established or residing within the Community: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_7
msgid "955 - not established or residing within the Community: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_7
msgid "956 - not established or residing within the Community: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_3_base_16
msgid "961 - suppliers established within the territory: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax_16
msgid "962 - suppliers established within the territory: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base_7
msgid "963 - base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_7
msgid "964 - tax 7%"
msgstr ""

#. module: l10n_lu
#: model:ir.model,name:l10n_lu.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_lu
#: model:account.report.column,name:l10n_lu.tax_report_section_1_balance
#: model:account.report.column,name:l10n_lu.tax_report_section_2_balance
#: model:account.report.column,name:l10n_lu.tax_report_sections_3_4_balance
msgid "Balance"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.l10n_lu_tax_report_assessment_turnover
msgid "I. ASSESSMENT OF TAXABLE TURNOVER"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.l10n_lu_tax_report_assessment_tax_due
msgid "II. ASSESSMENT OF TAX DUE"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_3_assessment_deducible_tax
msgid "III. ASSESSMENT OF DEDUCTIBLE TAX (input tax)"
msgstr ""

#. module: l10n_lu
#: model:account.report.line,name:l10n_lu.account_tax_report_line_4_tax_tobe_paid_or_reclaimed
msgid "IV. TAX TO BE PAID OR TO BE RECLAIMED"
msgstr ""

#. module: l10n_lu
#: model:ir.ui.menu,name:l10n_lu.account_reports_lu_statements_menu
msgid "Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.report,name:l10n_lu.l10n_lu_tax_report_section_1
msgid "Section I"
msgstr ""

#. module: l10n_lu
#: model:account.report,name:l10n_lu.l10n_lu_tax_report_section_2
msgid "Section II"
msgstr ""

#. module: l10n_lu
#: model:account.report,name:l10n_lu.l10n_lu_tax_report_sections_3_4
msgid "Sections III, IV"
msgstr ""

#. module: l10n_lu
#: model:account.report,name:l10n_lu.tax_report
msgid "Tax Report"
msgstr ""

```

## File: migrations\2.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID

def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'lu')], order="parent_path"):
        env['account.chart.template'].try_loading('lu', company)

```

## File: migrations\2.2\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID

def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'lu')], order="parent_path"):
        env['account.chart.template'].try_loading('lu', company)

```

## File: models\template_lu.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, Command
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('lu')
    def _get_lu_template_data(self):
        return {
            'property_account_receivable_id': 'lu_2011_account_4011',
            'property_account_payable_id': 'lu_2011_account_44111',
            'property_account_expense_categ_id': 'lu_2011_account_6061',
            'property_account_income_categ_id': 'lu_2020_account_703001',
            'property_stock_account_input_categ_id': 'lu_2011_account_321',
            'property_stock_account_output_categ_id': 'lu_2011_account_321',
            'property_stock_valuation_account_id': 'lu_2020_account_60761',
            'code_digits': '6',
        }

    @template('lu', 'res.company')
    def _get_lu_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.lu',
                'bank_account_code_prefix': '513',
                'cash_account_code_prefix': '516',
                'transfer_account_code_prefix': '517',
                'account_default_pos_receivable_account_id': 'lu_2011_account_40111',
                'income_currency_exchange_account_id': 'lu_2020_account_7561',
                'expense_currency_exchange_account_id': 'lu_2020_account_6561',
                'account_journal_suspense_account_id': 'lu_2011_account_485',
                'account_journal_early_pay_discount_loss_account_id': 'lu_2020_account_65562',
                'account_journal_early_pay_discount_gain_account_id': 'lu_2020_account_75562',
                'account_sale_tax_id': 'lu_2015_tax_VP-PA-17',
                'account_purchase_tax_id': 'lu_2015_tax_AP-PA-17',
            },
        }

    @template('lu', 'account.journal')
    def _get_lu_account_journal(self):
        return {
            'sale': {'refund_sequence': True},
            'purchase': {'refund_sequence': True},
        }

    @template('lu', 'account.reconcile.model')
    def _get_lu_reconcile_model(self):
        return {
            'bank_fees_template': {
                'name': 'Bank Fees',
                'rule_type': 'writeoff_button',
                'line_ids': [
                    Command.create({
                        'account_id': 'lu_2011_account_61333',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Bank Fees',
                    }),
                ],
            },
            'cash_discount_template': {
                'name': 'Cash Discount',
                'rule_type': 'writeoff_button',
                'line_ids': [
                    Command.create({
                        'account_id': 'lu_2020_account_65562',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Cash Discount',
                    }),
                ],
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_lu

```

