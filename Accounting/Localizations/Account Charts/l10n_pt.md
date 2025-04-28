# Odoo Module: l10n_pt

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Portugal - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['pt'],
    'version': '1.0',
    'author': 'Odoo',
    'category': 'Accounting/Localizations/Account Charts',
    'description': 'Portugal - Accounting',
    'depends': [
        'base',
        'account',
        'base_vat',
    ],
    'data': [
        'data/account_tax_report.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report_pt" model="account.report">
        <field name="name">Tax report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.pt"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="trp_base" model="account.report.line">
                <field name="name">Base</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="trp_base_section_1" model="account.report.line">
                        <field name="name">1 - Transmissions of goods and services on which you paid tax</field>
                        <field name="children_ids">
                            <record id="trp_base_1" model="account.report.line">
                                <field name="name">[1] Reduced rate</field>
                                <field name="code">trp_base_1</field>
                                <field name="expression_ids">
                                    <record id="trp_base_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_base_5" model="account.report.line">
                                <field name="name">[5] Intermediate rate</field>
                                <field name="code">trp_base_5</field>
                                <field name="expression_ids">
                                    <record id="trp_base_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_base_3" model="account.report.line">
                                <field name="name">[3] Normal rate</field>
                                <field name="code">trp_base_3</field>
                                <field name="expression_ids">
                                    <record id="trp_base_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_base_7" model="account.report.line">
                                <field name="name">[7] Exempt or non-taxed - Intra-community transfers of goods and supplies of services mentioned in the recapitulative statements</field>
                                <field name="code">trp_base_7</field>
                                <field name="expression_ids">
                                    <record id="trp_base_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_7</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_base_8" model="account.report.line">
                                <field name="name">[8] Exempt or non-taxed - Entitled to deduction</field>
                                <field name="code">trp_base_8</field>
                                <field name="expression_ids">
                                    <record id="trp_base_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_8</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_base_9" model="account.report.line">
                                <field name="name">[9] Exempt or non-taxed - Not entitled to deduction</field>
                                <field name="code">trp_base_9</field>
                                <field name="expression_ids">
                                    <record id="trp_base_9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_9</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_base_10" model="account.report.line">
                        <field name="name">2 - [10] Intra-community acquisitions of goods and assimilated operations</field>
                        <field name="code">trp_base_10</field>
                        <field name="aggregation_formula">trp_base_12.balance + trp_base_14.balance + trp_base_15.balance</field>
                        <field name="children_ids">
                            <record id="trp_base_12" model="account.report.line">
                                <field name="name">[12] Whose tax was assessed by the declarant</field>
                                <field name="code">trp_base_12</field>
                                <field name="expression_ids">
                                    <record id="trp_base_12_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_12</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_base_14" model="account.report.line">
                                <field name="name">[14] Covered by the articles 15.° of the CIVA or the RITI</field>
                                <field name="code">trp_base_14</field>
                                <field name="expression_ids">
                                    <record id="trp_base_14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_14</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_base_15" model="account.report.line">
                                <field name="name">[15] Covered by the n.°s 3, 4 e 5 of the article 22.° of the RITI</field>
                                <field name="code">trp_base_15</field>
                                <field name="expression_ids">
                                    <record id="trp_base_15_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_base_15</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_base_16" model="account.report.line">
                        <field name="name">3 - [16] Services rendered by taxable persons from other Member States, for which the tax was paid by the declarant</field>
                        <field name="code">trp_base_16</field>
                        <field name="expression_ids">
                            <record id="trp_base_16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_base_16</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_base_18" model="account.report.line">
                        <field name="name">4 - [18] Imports of goods for which the tax was assessed by the declarant (No. 8 of Article 27 of CIVA)</field>
                        <field name="code">trp_base_18</field>
                        <field name="expression_ids">
                            <record id="trp_base_18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_base_18</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_base_total_90" model="account.report.line">
                        <field name="name">[90] BASE TOTAL</field>
                        <field name="aggregation_formula">trp_base_1.balance + trp_base_5.balance + trp_base_3.balance + trp_base_7.balance + trp_base_8.balance + trp_base_9.balance + trp_base_10.balance + trp_base_16.balance + trp_base_18.balance</field>
                    </record>
                </field>
            </record>
            <record id="trp_tax_favor_company" model="account.report.line">
                <field name="name">Tax in favor of the taxable entity</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="trp_tax_favor_company5" model="account.report.line">
                        <field name="name">5 - Tax deductible</field>
                        <field name="children_ids">
                            <record id="trp_tax_favor_company_20" model="account.report.line">
                                <field name="name">[20] Non-current assets (fixed assets)</field>
                                <field name="code">trp_tax_favor_company_20</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_company_20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_company_20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_company_21" model="account.report.line">
                                <field name="name">[21] Inventories at reduced rate</field>
                                <field name="code">trp_tax_favor_company_21</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_company_21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_company_21</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_company_23" model="account.report.line">
                                <field name="name">[23] Inventories at the intermediate rate</field>
                                <field name="code">trp_tax_favor_company_23</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_company_23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_company_23</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_company_22" model="account.report.line">
                                <field name="name">[22] Inventories at the normal rate</field>
                                <field name="code">trp_tax_favor_company_22</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_company_22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_company_22</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_company_24" model="account.report.line">
                                <field name="name">[24] Other goods and services</field>
                                <field name="code">trp_tax_favor_company_24</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_company_24_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_company_24</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_company_40" model="account.report.line">
                        <field name="name">6 - [40] Monthly, quarterly and yearly regularizations</field>
                        <field name="code">trp_tax_favor_company_40</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_company_40_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_company_40</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_company_61" model="account.report.line">
                        <field name="name">7 - [61] Excess to carry forward from the previous period (Field 96 dof previous declaration - n.°4 of article 22.°)</field>
                        <field name="code">trp_tax_favor_company_61</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_company_61_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_company_61</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_company_65" model="account.report.line">
                        <field name="name">8 - [65] Attachment - (see Table 03)</field>
                        <field name="code">trp_tax_favor_company_65</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_company_65_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_company_65</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_company_67" model="account.report.line">
                        <field name="name">9 - [67] Attachment - (see Table 03)</field>
                        <field name="code">trp_tax_favor_company_67</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_company_67_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_company_67</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_company_total_91" model="account.report.line">
                        <field name="name">[91] TOTAL TAX IN FAVOR OF THE TAXABLE ENTITY</field>
                        <field name="code">trp_tax_favor_company_total_91</field>
                        <field name="aggregation_formula">trp_tax_favor_company_20.balance + trp_tax_favor_company_21.balance + trp_tax_favor_company_23.balance + trp_tax_favor_company_22.balance + trp_tax_favor_company_24.balance + trp_tax_favor_company_40.balance + trp_tax_favor_company_61.balance + trp_tax_favor_company_65.balance + trp_tax_favor_company_67.balance</field>
                    </record>
                </field>
            </record>
            <record id="trp_tax_favor_state" model="account.report.line">
                <field name="name">Tax in favor of the State</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="trp_tax_favor_state_1" model="account.report.line">
                        <field name="name">1 - Transmissions of goods and services on which you paid tax</field>
                        <field name="children_ids">
                            <record id="trp_tax_favor_state_2" model="account.report.line">
                                <field name="name">[2] Reduced rate</field>
                                <field name="code">trp_tax_favor_state_2</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_6" model="account.report.line">
                                <field name="name">[6] Intermediate rate</field>
                                <field name="code">trp_tax_favor_state_6</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_4" model="account.report.line">
                                <field name="name">[4] Normal rate</field>
                                <field name="code">trp_tax_favor_state_4</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_4</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_11" model="account.report.line">
                        <field name="name">2 - [11] Intra-community acquisitions of goods and assimilated operations</field>
                        <field name="code">trp_tax_favor_state_11</field>
                        <field name="aggregation_formula">trp_tax_favor_state_13.balance</field>
                        <field name="children_ids">
                            <record id="trp_tax_favor_state_13" model="account.report.line">
                                <field name="name">[13] Whose tax was assessed by the declarant</field>
                                <field name="code">trp_tax_favor_state_13</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_13</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_17" model="account.report.line">
                        <field name="name">3 - [17] Services rendered by taxable persons from other Member States, for which the tax was paid by the declarant</field>
                        <field name="code">trp_tax_favor_state_17</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_state_17</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_19" model="account.report.line">
                        <field name="name">4 - [19] Imports of goods on which tax has been assessed by the declarant (n°8 of article 27.° of the CIVA)</field>
                        <field name="code">trp_tax_favor_state_19</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_19_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_state_19</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_41" model="account.report.line">
                        <field name="name">6 - [41] Monthly, quarterly and yearly regularizations</field>
                        <field name="code">trp_tax_favor_state_41</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_41_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_state_41</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_66" model="account.report.line">
                        <field name="name">8 - [66] Attachment - (see Table 03)</field>
                        <field name="code">trp_tax_favor_state_66</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_66_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_state_66</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_68" model="account.report.line">
                        <field name="name">9 - [68] Attachment - (see Table 03)</field>
                        <field name="code">trp_tax_favor_state_68</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_68_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_state_68</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_total_92" model="account.report.line">
                        <field name="name">[92] TOTAL TAX TO THE STATE</field>
                        <field name="code">trp_tax_favor_state_total_92</field>
                        <field name="aggregation_formula">trp_tax_favor_state_2.balance + trp_tax_favor_state_6.balance + trp_tax_favor_state_4.balance + trp_tax_favor_state_11.balance + trp_tax_favor_state_17.balance + trp_tax_favor_state_19.balance + trp_tax_favor_state_41.balance + trp_tax_favor_state_66.balance + trp_tax_favor_state_68.balance</field>
                    </record>
                </field>
            </record>
            <record id="trp_totais" model="account.report.line">
                <field name="name">Totals</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="trp_tax_favor_state_total_93" model="account.report.line">
                        <field name="name">[93] Tax to be paid to the State</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_total_93_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">trp_tax_favor_state_total_92.balance - trp_tax_favor_company_total_91.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_total_94" model="account.report.line">
                        <field name="name">[94] Credit to be recovered</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_total_94_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">trp_tax_favor_company_total_91.balance - trp_tax_favor_state_total_92.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="trp_quadro_06A" model="account.report.line">
                <field name="name">Table 06A</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="trp_quadro_06A_A" model="account.report.line">
                        <field name="name">A - Operations located in Portugal where, as the acquirer, you have paid the VAT due</field>
                        <field name="children_ids">
                            <record id="trp_tax_favor_state_97" model="account.report.line">
                                <field name="name">[97] Made by entities resident in EU countries (does not include the operations mentioned in field 16)</field>
                                <field name="code">trp_tax_favor_state_97</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_97_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_97</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_98" model="account.report.line">
                                <field name="name">[98] Made by entities resident in third countries or territories</field>
                                <field name="code">trp_tax_favor_state_98</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_98_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_98</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_quadro_06A_B" model="account.report.line">
                        <field name="name">B - Transactions in which you assessed the VAT due by applying the reverse charge rule</field>
                        <field name="children_ids">
                            <record id="trp_tax_favor_state_99" model="account.report.line">
                                <field name="name">[99] Gold (Decree-Law 362/99)</field>
                                <field name="code">trp_tax_favor_state_99</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_99_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_99</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_100" model="account.report.line">
                                <field name="name">[100] Property acquisition with waiver of the exemption (Decree-Law 21/2007)</field>
                                <field name="code">trp_tax_favor_state_100</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_100_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_100</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_101" model="account.report.line">
                                <field name="name">[101] Metal [Subparagraph i) of n.°1 of the article 2.° of the CIVA]</field>
                                <field name="code">trp_tax_favor_state_101</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_101_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_101</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_102" model="account.report.line">
                                <field name="name">[102] Construction services[Subparagraph j) of n.°1 of the article 2.° of the CIVA]</field>
                                <field name="code">trp_tax_favor_state_102</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_102_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_102</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_105" model="account.report.line">
                                <field name="name">[105] Greenhouse gas emissions [Subparagraph l) of n.°1 of the article 2.° of the CIVA]</field>
                                <field name="code">trp_tax_favor_state_105</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_105_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_105</field>
                                    </record>
                                </field>
                            </record>
                            <record id="trp_tax_favor_state_107" model="account.report.line">
                                <field name="name">[107] Acquisition of cork and other forestry products [Subparagraph m) of n.°1 of the article 2.° of the CIVA</field>
                                <field name="code">trp_tax_favor_state_107</field>
                                <field name="expression_ids">
                                    <record id="trp_tax_favor_state_107_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">trp_tax_favor_state_107</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_103" model="account.report.line">
                        <field name="name">C - [103] Operations referred to in subparagraphs f) and g) of N.°3 of the article 3.°E subparagraph a) and b) of N.°2 of the article 4.° of the CIVA</field>
                        <field name="code">trp_tax_favor_state_103</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_103_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_state_103</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_104" model="account.report.line">
                        <field name="name">D - [104] Operations referred to in subparagraphs a), b) and c) of the article 42.° of the CIVA</field>
                        <field name="code">trp_tax_favor_state_104</field>
                        <field name="expression_ids">
                            <record id="trp_tax_favor_state_104_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">trp_tax_favor_state_104</field>
                            </record>
                        </field>
                    </record>
                    <record id="trp_tax_favor_state_total_106" model="account.report.line">
                        <field name="name">[106] Sum of the table 06A</field>
                        <field name="aggregation_formula">trp_tax_favor_state_97.balance + trp_tax_favor_state_98.balance + trp_tax_favor_state_99.balance + trp_tax_favor_state_100.balance + trp_tax_favor_state_101.balance + trp_tax_favor_state_102.balance + trp_tax_favor_state_105.balance + trp_tax_favor_state_107.balance + trp_tax_favor_state_103.balance + trp_tax_favor_state_104.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-pt.csv

```csv
"id","code","l10n_pt_taxonomy_code","name","account_type","reconcile","name@pt"
"chart_13","13","3","Other bank deposits","asset_cash","False","Outros depósitos bancários"
"chart_1411","1411","4","Potentially favorable","asset_current","False","Potencialmente favoráveis"
"chart_1412","1412","5","Potentially unfavorable","liability_current","False","Potencialmente desfavoráveis"
"chart_1421","1421","6","Financial assets","asset_current","False","Ativos financeiros"
"chart_1422","1422","7","Financial liabilities","asset_current","False","Passivos financeiros"
"chart_1431","1431","8","Other financial assets","asset_current","False","Outros ativos financeiros"
"chart_1432","1432","9","Other financial liabilities","asset_current","False","Outros passivos financeiros"
"chart_2111","2111","10","General customers","asset_receivable","True","Clientes gerais"
"chart_2112","2112","11","Customers - Mother company","asset_receivable","True","Clientes - empresa-mãe"
"chart_2113","2113","12","Customers - Subsidiary companies","asset_receivable","True","Clientes - empresas subsidiárias"
"chart_2114","2114","13","Customers - Associated companies","asset_receivable","True","Clientes - empresas associadas"
"chart_2115","2115","14","Customers - Joint enterprises","asset_receivable","True","Clientes - empreendimentos conjuntos"
"chart_2116","2116","15","Customers - Other related parties","asset_receivable","True","Clientes - outras partes relacionadas"
"chart_2117","2117","10","General customers (POS)","asset_receivable","True","Clientes gerais (PoS)"
"chart_2121","2121","16","General customers","asset_receivable","True","Clientes gerais"
"chart_2122","2122","17","Customers - Mother company","asset_receivable","True","Clientes - empresa-mãe"
"chart_2123","2123","18","Customers - Subsidiary companies","asset_receivable","True","Clientes - empresas subsidiárias"
"chart_2124","2124","19","Customers - Associated companies","asset_receivable","True","Clientes - empresas associadas"
"chart_2125","2125","20","Customers - Joint enterprises","asset_receivable","True","Clientes - empreendimentos conjuntos"
"chart_2126","2126","21","Customers - Other related parties","asset_receivable","True","Clientes - outras partes relacionadas"
"chart_213","213","22","Customers - Other Customers","asset_receivable","True","Clientes - Outros clientes"
"chart_218","218","23","Customer advances","asset_receivable","True","Adiantamentos de clientes"
"chart_21911","21911","24","General customers","asset_receivable","True","Clientes gerais"
"chart_21912","21912","25","Customers - Mother company","asset_receivable","True","Clientes - empresa-mãe"
"chart_21913","21913","26","Customers - Subsidiary companies","asset_receivable","True","Clientes - empresas subsidiárias"
"chart_21914","21914","27","Customers - Associated companies","asset_receivable","True","Clientes - empresas associadas"
"chart_21915","21915","28","Customers - Joint enterprises","asset_receivable","True","Clientes - empreendimentos conjuntos"
"chart_21916","21916","29","Customers - Other related parties","asset_receivable","True","Clientes - outras partes relacionadas"
"chart_21921","21921","30","General customers","asset_receivable","True","Clientes gerais"
"chart_21922","21922","31","Customers - Mother company","asset_receivable","True","Clientes - empresa-mãe"
"chart_21923","21923","32","Customers - Subsidiary companies","asset_receivable","True","Clientes - empresas subsidiárias"
"chart_21924","21924","33","Customers - Associated companies","asset_receivable","True","Clientes - empresas associadas"
"chart_21925","21925","34","Customers - Joint enterprises","asset_receivable","True","Clientes - empreendimentos conjuntos"
"chart_21926","21926","35","Customers - Other related parties","asset_receivable","True","Clientes - outras partes relacionadas"
"chart_2193","2193","36","Other customers","asset_receivable","True","Outros Clientes"
"chart_2211","2211","37","General suppliers","liability_payable","True","Fornecedores gerais"
"chart_2212","2212","38","Suppliers - Mother company","liability_payable","True","Fornecedores - empresa-mãe"
"chart_2213","2213","39","Suppliers - Subsidiary companies","liability_payable","True","Fornecedores - empresas subsidiárias"
"chart_2214","2214","40","Suppliers - Associated companies","liability_payable","True","Fornecedores - empresas associadas"
"chart_2215","2215","41","Suppliers - Joint enterprises","liability_payable","True","Fornecedores - empreendimentos conjuntos"
"chart_2216","2216","42","Suppliers - Other related parties","liability_payable","True","Fornecedores - outras partes relacionadas"
"chart_2221","2221","43","General suppliers","liability_payable","True","Fornecedores gerais"
"chart_2222","2222","44","Suppliers - Mother company","liability_payable","True","Fornecedores - empresa-mãe"
"chart_2223","2223","45","Suppliers - Subsidiary companies","liability_payable","True","Fornecedores - empresas subsidiárias"
"chart_2224","2224","46","Suppliers - Associated companies","liability_payable","True","Fornecedores - empresas associadas"
"chart_2225","2225","47","Suppliers - Joint enterprises","liability_payable","True","Fornecedores - empreendimentos conjuntos"
"chart_2226","2226","48","Suppliers - Other related parties","liability_payable","True","Fornecedores - outras partes relacionadas"
"chart_223","223","49","Other suppliers","liability_payable","True","Outros fornecedores"
"chart_225","225","50","Invoices received and checked","liability_payable","True","Faturas em receção e conferência"
"chart_228","228","51","Advances to suppliers","liability_payable","True","Adiantamentos a fornecedores"
"chart_229","229","52","Accumulated impairment losses","liability_payable","True","Perdas por imparidade acumuladas"
"chart_2311","2311","53","To corporate bodies","liability_non_current","False","Aos órgãos sociais"
"chart_2312","2312","54","Staff","liability_current","False","Ao pessoal"
"chart_2321","2321","55","To corporate bodies","liability_current","False","Aos órgãos sociais"
"chart_2322","2322","56","Staff","liability_current","False","Ao pessoal"
"chart_23311","23311","61","With the corporate bodies (current)","liability_current","False","Com os órgãos sociais (CORRENTE)"
"chart_23312","23312","62","With the corporate bodies (non-current)","liability_non_current","False","Com os órgãos sociais (NÃO CORRENTE)"
"chart_23321","23321","63","With the staff (current)","liability_current","False","Com o pessoal (CORRENTE)"
"chart_23322","23322","64","With the staff (non-current)","liability_non_current","False","Com o pessoal (NÃO CORRENTE)"
"chart_23711","23711","57","Of corporate bodies (current)","liability_current","False","Dos órgãos sociais (CORRENTE)"
"chart_23712","23712","58","Of corporate bodies (non-current)","liability_non_current","False","Dos órgãos sociais (NÃOCORRENTE)"
"chart_23721","23721","59","Of staff (current)","liability_current","False","Do pessoal (CORRENTE)"
"chart_23722","23722","60","Of staff (non-current)","liability_non_current","False","Do pessoal (NÃO CORRENTE)"
"chart_23911","23911","65","To corporate bodies","liability_current","False","Aos órgãos sociais"
"chart_23912","23912","66","Staff","liability_current","False","Ao pessoal"
"chart_239211","239211","67","To corporate bodies (current)","liability_current","False","Aos órgãos sociais (CORRENTE)"
"chart_239212","239212","68","To corporate bodies (non-current)","liability_non_current","False","Ao órgãos sociais (NÃO CORRENTE)"
"chart_239221","239221","69","To staff (current)","liability_current","False","Ao pessoal (CORRENTE)"
"chart_239222","239222","70","To staff (non-current)","liability_non_current","False","Ao pessoal (NÃO CORRENTE)"
"chart_241","241","71","Income tax","liability_current","False","Imposto sobre o rendimento"
"chart_242","242","72","Retention retention on income","asset_current","False","Retenção de impostos sobre rendimentos"
"chart_2431","2431","73","VAT - Supported","asset_current","False","IVA - Suportado"
"chart_2432","2432","74","VAT - Deductible","liability_current","False","IVA - Dedutível"
"chart_2433","2433","75","VAT - Liquidated","liability_current","False","IVA - Liquidado"
"chart_2434","2434","76","VAT - Regularization","asset_current","False","IVA - Regularizações"
"chart_2435","2435","77","VAT - Clearance","liability_current","False","IVA - Apuramento"
"chart_2436","2436","78","VAT - Payable","liability_current","False","IVA - A pagar"
"chart_2437","2437","79","VAT - Recovering","asset_current","False","IVA - A recuperar"
"chart_2438","2438","80","VAT - Refaults orders","asset_current","False","IVA - Reembolsos pedidos"
"chart_2439","2439","81","VAT - Official settlements","asset_current","False","IVA - Liquidações oficiosas"
"chart_244","244","82","Other taxes","liability_current","False","Outros impostos"
"chart_245","245","83","Contributions to Social Security","liability_current","False","Contribuições para a Segurança Social"
"chart_246","246","84","Taxes of local authorities","liability_current","False","Tributos das autarquias locais"
"chart_248","248","85","Other taxes","liability_current","False","Outros impostos"
"chart_25111","25111","86","Bank loans (current)","liability_current","False","Empréstimos bancários (CORRENTE)"
"chart_25112","25112","87","Bank loans (non-current)","liability_non_current","False","Empréstimos bancários (NÃO CORRENTE)"
"chart_25121","25121","88","Bank discoveries (current)","liability_current","False","Descobertos bancários (CORRENTE)"
"chart_25122","25122","89","Bank discoveries (non-current)","liability_non_current","False","Descobertos bancários (NÃO CORRENTE)"
"chart_25131","25131","90","Financial Locations (current)","liability_current","False","Locações financeiras(CORRENTE)"
"chart_25132","25132","91","Financial locations (non-current)","liability_non_current","False","Locações financeiras(NÃO CORRENTE)"
"chart_25141","25141","92","Other financing - (current)","liability_current","False","Outros financiamentos - (CORRENTE)"
"chart_25142","25142","93","Other financing - (non-current)","liability_current","False","Outros financiamentos - (NÃO CORRENTE)"
"chart_25211","25211","94","Loans by obligations (current)","liability_current","False","Empréstimos por obrigações (CORRENTE)"
"chart_25212","25212","95","Loans by obligations (non-current)","liability_non_current","False","Empréstimos por obrigações (NÃO CORRENTE)"
"chart_25221","25221","96","Other financing - (current)","liability_current","False","Outros financiamentos - (CORRENTE)"
"chart_25311","25311","98","Mother company - Supplies and other mutual (current)","liability_current","False","Empresa-mãe - Suprimentos e outros mútuos (CORRENTE)"
"chart_25312","25312","99","Mother company - Supplies and other mutual (non-current)","liability_non_current","False","Empresa-mãe - Suprimentos e outros mútuos (NÃO CORRENTE)"
"chart_25321","25321","100","Other participants - Supplies and other mutual (current)","liability_current","False","Outros participantes - Suprimentos e outros mútuos (CORRENTE)"
"chart_25322","25322","101","Other participants - Supplies and other mutual (non-current)","liability_non_current","False","Outros participantes - Suprimentos e outros mútuos (NÃO CORRENTE)"
"chart_2541","2541","102","Subsidiaries, associates and joint enterprises (current)","liability_current","False","Subsidiárias, associadas e empr. Conjuntos (CORRENTE)"
"chart_2542","2542","103","Subsidiaries, associates and joint enterprises (non-current)","liability_non_current","False","Subsidiárias, associadas e empr. Conjuntos (NÃO CORRENTE)"
"chart_2581","2581","104","Other financiers (current)","liability_current","False","Outros financiadores (CORRENTE)"
"chart_2582","2582","105","Other financiers (non-current)","liability_non_current","False","Outros financiadores (NÃO CORRENTE)"
"chart_261","261","106","Shareholders with subscription","asset_current","False","Acionistas c/ subscrição"
"chart_262","262","107","Unreleased quotas","asset_current","False","Quotas não liberadas"
"chart_263","263","108","Advances due to profits","asset_current","False","Adiantamentos por conta de lucros"
"chart_264","264","109","Attributed results","asset_current","False","Resultados atribuídos"
"chart_265","265","110","Available profits","asset_current","False","Lucros disponíveis"
"chart_2661","2661","111","Loans granted - Mother company (current)","asset_current","False","Empréstimos concedidos - empresa-mãe (CORRENTE)"
"chart_2662","2662","112","Loans granted - Mother company (non-current)","asset_non_current","False","Empréstimos concedidos - empresa-mãe (NÃO CORRENTE)"
"chart_2681","2681","113","Other operations (current)","asset_current","False","Outras operações (CORRENTE)"
"chart_2682","2682","114","Other operations (non-current)","asset_non_current","False","Outras operações (NÃO CORRENTE)"
"chart_2691","2691","115","Shareholders with subscription","asset_current","False","Acionistas c/ subscrição"
"chart_2692","2692","116","Unplorer Quotas","asset_current","False","Quotas não liberadas"
"chart_2693","2693","117","Advances due to profits","asset_current","False","Adiantamentos por conta de lucros"
"chart_2694","2694","118","Attributed results","asset_current","False","Resultados atribuídos"
"chart_2695","2695","119","Available profits","asset_current","False","Lucros disponíveis"
"chart_26961","26961","120","Loans granted - Mother company (current)","asset_current","False","Empréstimos concedidos - empresa-mãe (CORRENTE)"
"chart_26962","26962","121","Loans granted - Mother company (non-current)","asset_non_current","False","Empréstimos concedidos - empresa-mãe (NÃO CORRENTE)"
"chart_26971","26971","122","Other operations (current)","asset_current","False","Outras operações (CORRENTE)"
"chart_26972","26972","123","Other operations (non-current)","asset_non_current","False","Outras operações (NÃO CORRENTE)"
"chart_27111","27111","124","Investment suppliers - General accounts (current)","asset_current","False","Fornecedores de investimentos - contas gerais (CORRENTE)"
"chart_27112","27112","125","Investment suppliers - General accounts (non-current)","asset_non_current","False","Fornecedores de investimentos - contas gerais (NÃO CORRENTE)"
"chart_27121","27121","126","Invoices received and checked (current)","asset_current","False","Faturas em receção e conferência (CORRENTE)"
"chart_27122","27122","127","Invoices received and checked (non-current)","asset_non_current","False","Faturas em receção e conferência (NÃO CORRENTE)"
"chart_27131","27131","128","Advances to investment suppliers (current)","asset_current","False","Adiantamentos a fornecedores de investimentos (CORRENTE)"
"chart_27132","27132","129","Advances to investment suppliers (non-current)","asset_non_current","False","Adiantamentos a fornecedores de investimentos (NÃO CORRENTE)"
"chart_2721","2721","130","Debtors for income additions","asset_current","False","Devedores por acréscimos de rendimentos"
"chart_2722","2722","131","Creditors by expenditure","liability_current","False","Credores por acréscimos de gastos"
"chart_273","273","132","Post-employment benefits","liability_non_current","False","Benefícios pós-emprego"
"chart_2741","2741","133","Deferred taxes","asset_non_current","False","Ativos por impostos diferidos"
"chart_2742","2742","134","Passive by deferred taxes","liability_non_current","False","Passivos por impostos diferidos"
"chart_2751","2751","135","Creditors for unreleased subscriptions (current)","liability_current","False","Credores por subscrições não liberadas (CORRENTE)"
"chart_2752","2752","136","Creditors for unreleased subscriptions (non-current)","liability_non_current","False","Credores por subscrições não liberadas (NÃO CORRENTE)"
"chart_276","276","137","Advances due to sales","liability_current","False","Adiantamentos por conta de vendas"
"chart_2781","2781","138","Other debtors and creditors (current)","liability_current","False","Outros devedores e credores (CORRENTE)"
"chart_2782","2782","139","Other debtors and creditors (non-current)","liability_non_current","False","Outros devedores e credores (NÃO CORRENTE)"
"chart_27911","27911","140","Advances to investment suppliers (current)","asset_current","False","Adiantamentos a fornecedores de investimentos (CORRENTE)"
"chart_27912","27912","141","Advances to investment suppliers (non-current)","asset_non_current","False","Adiantamentos a fornecedores de investimentos (NÃO CORRENTE)"
"chart_2792","2792","142","Debtors and creditors for additions (economic periodization) - Debtors due to income accruals","asset_current","False","Devedores e credores por acréscimos (periodização económica) - Dev por acréscimos de rendimentos"
"chart_2793","2793","143","Deferred taxes","asset_non_current","False","Ativos por impostos diferidos"
"chart_27941","27941","144","Other debtors and creditors (current)","asset_current","False","Outros devedores e credores (CORRENTE)"
"chart_27942","27942","145","Other debtors and creditors (non-current)","asset_non_current","False","Outros devedores e credores (NÃO CORRENTE)"
"chart_281","281","146","Spending to recognize","asset_current","False","Gastos a reconhecer"
"chart_282","282","147","Income to recognize","liability_current","False","Rendimentos a reconhecer"
"chart_291","291","148","Taxes","liability_non_current","False","Impostos"
"chart_292","292","149","Guarantees to customers","liability_non_current","False","Garantias a clientes"
"chart_293","293","150","Ongoing judicial proceedings","liability_non_current","False","Processos judiciais em curso"
"chart_294","294","151","Occupational accidents and professional diseases","liability_non_current","False","Acidentes de trabalho e doenças profissionais"
"chart_295","295","152","Environmental materials","liability_non_current","False","Matérias ambientais"
"chart_296","296","153","Costly","liability_non_current","False","Contratos onerosos"
"chart_297","297","154","Restructuring","liability_non_current","False","Reestruturação"
"chart_298","298","155","Other provisions","liability_non_current","False","Outras provisões"
"chart_311","311","156","Goods","asset_current","False","Mercadorias"
"chart_312","312","157","Raw materials, subsidiaries and consumption","asset_current","False","Matérias-primas, subsidiárias e de consumo"
"chart_313","313","158","Biological assets","asset_current","False","Ativos biológicos"
"chart_3171","3171","159","Goods","asset_current","False","Mercadorias"
"chart_3172","3172","160","Raw materials, subsidiaries and consumption","asset_current","False","Matérias-primas, subsidiárias e de consumo"
"chart_3173","3173","161","Biological assets","asset_current","False","Ativos biológicos"
"chart_3181","3181","162","Goods","asset_current","False","Mercadorias"
"chart_3182","3182","163","Raw materials, subsidiaries and consumption","asset_current","False","Matérias-primas, subsidiárias e de consumo"
"chart_3183","3183","164","Biological assets","asset_current","False","Ativos biológicos"
"chart_321","321","165","Goods","asset_current","False","Mercadorias"
"chart_325","325","166","Transit goods","asset_current","False","Mercadorias em trânsito"
"chart_326","326","167","Third party goods","asset_current","False","Mercadorias em poder de terceiros"
"chart_3291","3291","168","Goods","asset_current","False","Mercadorias"
"chart_3292","3292","169","Transit goods","asset_current","False","Mercadorias em trânsito"
"chart_3293","3293","170","Third party goods","asset_current","False","Mercadorias em poder de terceiros"
"chart_331","331","171","Raw material","asset_current","False","Matérias-primas"
"chart_332","332","172","Subsidiary","asset_current","False","Matérias subsidiárias"
"chart_333","333","173","Packaging","asset_current","False","Embalagens"
"chart_334","334","174","Miscellaneous materials","asset_current","False","Materiais diversos"
"chart_335","335","175","Materials in transit","asset_current","False","Matérias em trânsito"
"chart_336","336","176","Other subjects - raws, subsidiaries and consumption","asset_current","False","Outras matérias - primas, subsidiárias e de consumo"
"chart_3391","3391","177","Raw material","asset_current","False","Matérias-primas"
"chart_3392","3392","178","Subsidiary","asset_current","False","Matérias subsidiárias"
"chart_3393","3393","179","Packaging","asset_current","False","Embalagens"
"chart_3394","3394","180","Miscellaneous materials","asset_current","False","Materiais diversos"
"chart_3395","3395","181","Materials in transit","asset_current","False","Matérias em trânsito"
"chart_3396","3396","182","Other raw materials, subsidiaries and consumption","asset_current","False","Outras matérias-primas, subsidiárias e de consumo"
"chart_341","341","183","Finished and intermediate products","asset_current","False","Produtos acabados e intermédios"
"chart_346","346","184","Products in the possession of third parties","asset_current","False","Produtos em poder de terceiros"
"chart_3491","3491","185","Finished and intermediate products","asset_current","False","Produtos acabados e intermédios"
"chart_3492","3492","186","Products in the possession of third parties","asset_current","False","Produtos em poder de terceiros"
"chart_351","351","187","Byproducts","asset_current","False","Subprodutos"
"chart_352","352","188","Waste, residues and refuses","asset_current","False","Desperdícios, resíduos e refugos"
"chart_353","353","189","Other byproducts, waste, residues and refuses","asset_current","False","Outros subprodutos, desperdícios, resíduos e refugos"
"chart_3591","3591","190","Byproducts","asset_current","False","Subprodutos"
"chart_3592","3592","191","Waste, residues and refuses","asset_current","False","Desperdícios, resíduos e refugos"
"chart_3593","3593","192","Other waste, residues and refuses","asset_current","False","Outros desperdícios, resíduos e refugos"
"chart_361","361","193","Products and work in progress","asset_current","False","Produtos e trabalhos em curso"
"chart_369","369","194","Accumulated impairment losses","asset_current","False","Perdas por imparidade acumuladas"
"chart_3711","3711","195","Animals","asset_current","False","Animais"
"chart_3712","3712","196","Plant","asset_current","False","Plantas"
"chart_3721","3721","197","Animals","asset_non_current","False","Animais"
"chart_3722","3722","198","Plant","asset_non_current","False","Plantas"
"chart_3781","3781","199","Consumables","asset_current","False","Consumíveis"
"chart_3782","3782","200","Of production","asset_non_current","False","De produção"
"chart_3791","3791","201","Consumables","asset_current","False","Consumíveis"
"chart_3792","3792","202","Of production","asset_non_current","False","De produção"
"chart_382","382","203","Goods","asset_current","False","Mercadorias"
"chart_383","383","204","Raw materials, subsidiaries and consumption","asset_current","False","Matérias-primas, subsidiárias e de consumo"
"chart_384","384","205","Finished and intermediate products","asset_current","False","Produtos acabados e intermédios"
"chart_385","385","206","Byproducts, waste, residues and refuses","asset_current","False","Subprodutos, desperdícios, resíduos e refugos"
"chart_386","386","207","Products and work in progress","asset_current","False","Produtos e trabalhos em curso"
"chart_387","387","208","Biological assets","asset_current","False","Ativos biológicos"
"chart_391","391","209","Goods","asset_current","False","Mercadorias"
"chart_392","392","210","Raw materials, subsidiaries and consumption","asset_current","False","Matérias-primas, subsidiárias e de consumo"
"chart_393","393","211","Finished and intermediate products","asset_current","False","Produtos acabados e intermédios"
"chart_394","394","212","Byproducts, waste, residues and refuses","asset_current","False","Subprodutos, desperdícios, resíduos e refugos"
"chart_395","395","213","Products and work in progress","asset_current","False","Produtos e trabalhos em curso"
"chart_396","396","214","Consumable biological assets","asset_current","False","Ativos biológicos consumíveis"
"chart_397","397","215","Biological production assets","asset_current","False","Ativos biológicos de produção"
"chart_41111","41111","216","Capital participation","asset_non_current","False","Participação de capital"
"chart_41112","41112","217","Goodwill","asset_non_current","False","Goodwill"
"chart_4112","4112","218","Capital Participations - Other methods","asset_non_current","False","Participações de capital- outros métodos"
"chart_4113","4113","219","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_41211","41211","221","Capital participation","asset_non_current","False","Participação de capital"
"chart_41212","41212","222","Goodwill","asset_non_current","False","Goodwill"
"chart_4122","4122","223","Capital Participations - Other methods","asset_non_current","False","Participações de capital- outros métodos"
"chart_4123","4123","224","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_41311","41311","226","Capital participation","asset_non_current","False","Participação de capital"
"chart_41312","41312","227","Goodwill","asset_non_current","False","Goodwill"
"chart_4132","4132","228","Capital Participations - Other methods","asset_non_current","False","Participações de capital- outros métodos"
"chart_4133","4133","229","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_4141","4141","231","Capital Participations","asset_non_current","False","Participações de capital"
"chart_4142","4142","232","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_4151","4151","234","Detained to maturity","asset_non_current","False","Detidos até à maturidade"
"chart_4158","4158","235","Others","asset_non_current","False","Outros"
"chart_41611","41611","236","Subsidiary investments - Goodwill","asset_non_current","False","Investimentos em subsidiárias - Goodwill"
"chart_41612","41612","237","Investments in associates - Goodwill","asset_non_current","False","Investimentos em associadas - Goodwill"
"chart_41613","41613","238","Investments in jointly controlled entities - Goodwill","asset_non_current","False","Investimentos em associadas - Goodwill"
"chart_419111","419111","239","Capital participation","asset_non_current","False","Participação de capital"
"chart_419112","419112","240","Goodwill","asset_non_current","False","Goodwill"
"chart_41912","41912","241","Capital Participations - Other methods","asset_non_current","False","Participações de capital- outros métodos"
"chart_41913","41913","242","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_41914","41914","243","Other financial investments","asset_non_current","False","Outros investimentos financeiros"
"chart_419211","419211","244","Capital participation","asset_non_current","False","Participação de capital"
"chart_419212","419212","245","Goodwill","asset_non_current","False","Goodwill"
"chart_41922","41922","246","Capital Participations - Other methods","asset_non_current","False","Participações de capital- outros métodos"
"chart_41923","41923","247","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_41924","41924","248","Other financial investments","asset_non_current","False","Outros investimentos financeiros"
"chart_419311","419311","249","Capital participation","asset_non_current","False","Participação de capital"
"chart_419312","419312","250","Goodwill","asset_non_current","False","Goodwill"
"chart_41932","41932","251","Capital Participations - Other methods","asset_non_current","False","Participações de capital- outros métodos"
"chart_41933","41933","252","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_41934","41934","253","Other financial investments","asset_non_current","False","Outros investimentos financeiros"
"chart_41941","41941","254","Capital Participations","asset_non_current","False","Participações de capital"
"chart_41942","41942","255","Loans granted","asset_non_current","False","Empréstimos concedidos"
"chart_41943","41943","256","Other financial investments","asset_non_current","False","Outros investimentos financeiros"
"chart_41951","41951","257","Detained to maturity","asset_non_current","False","Detidos até à maturidade"
"chart_41952","41952","258","Others (*)","asset_non_current","False","Outros (*)"
"chart_421","421","259","Land and natural resources","asset_non_current","False","Terrenos e recursos naturais"
"chart_422","422","260","Buildings and other constructions","asset_non_current","False","Edifícios e outras construções"
"chart_426","426","261","Other investment properties","asset_non_current","False","Outras propriedades de investimento"
"chart_4281","4281","262","Land and natural resources","asset_non_current","False","Terrenos e recursos naturais"
"chart_4282","4282","263","Buildings and other constructions","asset_non_current","False","Edifícios e outras construções"
"chart_4283","4283","264","Other investment properties","asset_non_current","False","Outras propriedades de investimento"
"chart_4291","4291","265","Land and natural resources","asset_non_current","False","Terrenos e recursos naturais"
"chart_4292","4292","266","Buildings and other constructions","asset_non_current","False","Edifícios e outras construções"
"chart_4293","4293","267","Other investment properties","asset_non_current","False","Outras propriedades de investimento"
"chart_431","431","268","Land and natural resources","asset_non_current","False","Terrenos e recursos naturais"
"chart_432","432","269","Buildings and other constructions","asset_non_current","False","Edifícios e outras construções"
"chart_433","433","270","Basic equipment","asset_non_current","False","Equipamento básico"
"chart_434","434","271","Transport equipment","asset_non_current","False","Equipamento de transporte"
"chart_435","435","272","Office equipment","asset_non_current","False","Equipamento administrativo"
"chart_436","436","273","Biological equipment","asset_non_current","False","Equipamentos biológicos"
"chart_437","437","274","Other tangible fixed assets","asset_non_current","False","Outros ativos fixos tangíveis"
"chart_4381","4381","275","Land and natural resources","asset_non_current","False","Terrenos e recursos naturais"
"chart_4382","4382","276","Buildings and other constructions","asset_non_current","False","Edifícios e outras construções"
"chart_4383","4383","277","Basic equipment","asset_non_current","False","Equipamento básico"
"chart_4384","4384","278","Transport equipment","asset_non_current","False","Equipamento de transporte"
"chart_4385","4385","279","Office equipment","asset_non_current","False","Equipamento administrativo"
"chart_4386","4386","280","Biological equipment","asset_non_current","False","Equipamentos biológicos"
"chart_4387","4387","281","Other tangible fixed assets","asset_non_current","False","Outros ativos fixos tangíveis"
"chart_4391","4391","282","Land and natural resources","asset_non_current","False","Terrenos e recursos naturais"
"chart_4392","4392","283","Buildings and other constructions","asset_non_current","False","Edifícios e outras construções"
"chart_4393","4393","284","Basic equipment","asset_non_current","False","Equipamento básico"
"chart_4394","4394","285","Transport equipment","asset_non_current","False","Equipamento de transporte"
"chart_4395","4395","286","Office equipment","asset_non_current","False","Equipamento administrativo"
"chart_4396","4396","287","Biological equipment","asset_non_current","False","Equipamentos biológicos"
"chart_4397","4397","288","Other tangible fixed assets","asset_non_current","False","Outros ativos fixos tangíveis"
"chart_441","441","289","Goodwill","asset_non_current","False","Goodwill"
"chart_442","442","290","Development projects","asset_non_current","False","Projetos de desenvolvimento"
"chart_443","443","291","Computer programs","asset_non_current","False","Programas de computador"
"chart_444","444","292","Industrial property","asset_non_current","False","Propriedade industrial"
"chart_446","446","293","Other intangible assets","asset_non_current","False","Outros ativos intangíveis"
"chart_4481","4481","294","Goodwill","asset_non_current","False","Goodwill"
"chart_4482","4482","295","Development projects","asset_non_current","False","Projetos de desenvolvimento"
"chart_4483","4483","296","Computer programs","asset_non_current","False","Programas de computador"
"chart_4484","4484","297","Industrial property","asset_non_current","False","Propriedade industrial"
"chart_4485","4485","298","Other intangible assets","asset_non_current","False","Outros ativos intangíveis"
"chart_4491","4491","299","Goodwill","asset_non_current","False","Goodwill"
"chart_4492","4492","300","Development projects","asset_non_current","False","Projetos de desenvolvimento"
"chart_4493","4493","301","Computer programs","asset_non_current","False","Programas de computador"
"chart_4494","4494","302","Industrial property","asset_non_current","False","Propriedade industrial"
"chart_4495","4495","303","Other intangible assets","asset_non_current","False","Outros ativos intangíveis"
"chart_451","451","304","Financial investments ongoing","asset_non_current","False","Investimentos financeiros em curso"
"chart_452","452","305","Investment properties in progress","asset_non_current","False","Propriedades de investimento em curso"
"chart_453","453","306","Tangible fixed assets ongoing","asset_non_current","False","Ativos fixos tangíveis em curso"
"chart_454","454","307","Intangible assets ongoing","asset_non_current","False","Ativos intangíveis em curso"
"chart_4551","4551","308","Financial investments ongoing","asset_non_current","False","Investimentos financeiros em curso"
"chart_4552","4552","309","Investment properties in progress","asset_non_current","False","Propriedades de investimento em curso"
"chart_4553","4553","310","Tangible fixed assets ongoing","asset_non_current","False","Ativos fixos tangíveis em curso"
"chart_4554","4554","311","Intangible assets ongoing","asset_non_current","False","Ativos intangíveis em curso"
"chart_4591","4591","312","Financial investments ongoing","asset_non_current","False","Investimentos financeiros em curso"
"chart_4592","4592","313","Investment properties in progress","asset_non_current","False","Propriedades de investimento em curso"
"chart_4593","4593","314","Tangible fixed assets ongoing","asset_non_current","False","Ativos fixos tangíveis em curso"
"chart_4594","4594","315","Intangible assets ongoing","asset_non_current","False","Ativos intangíveis em curso"
"chart_45951","45951","316","Financial investments ongoing","asset_non_current","False","Investimentos financeiros em curso"
"chart_45952","45952","317","Investment properties in progress","asset_non_current","False","Propriedades de investimento em curso"
"chart_45953","45953","318","Tangible fixed assets ongoing","asset_non_current","False","Ativos fixos tangíveis em curso"
"chart_45954","45954","319","Intangible assets ongoing","asset_non_current","False","Ativos intangíveis em curso"
"chart_461","461","320","Financial investments","asset_current","False","Investimentos Financeiros"
"chart_462","462","321","Investment properties","asset_current","False","Propriedades de investimento"
"chart_463","463","322","Tangible fixed assets","asset_current","False","Ativos fixos tangíveis"
"chart_464","464","323","Intangible assets","asset_current","False","Ativos intangíveis"
"chart_465","465","324","Others","asset_current","False","Outros"
"chart_466","466","325","Non-current passive held for sale","liability_current","False","Passivos não correntes detidos para venda"
"chart_4691","4691","326","Financial investments","asset_current","False","Investimentos Financeiros"
"chart_4692","4692","327","Investment properties","asset_current","False","Propriedades de investimento"
"chart_4693","4693","328","Tangible fixed assets","asset_current","False","Ativos fixos tangíveis"
"chart_4694","4694","329","Intangible assets","asset_current","False","Ativos intangíveis"
"chart_4695","4695","330","Others","asset_current","False","Outros"
"chart_51","51","331","Subscribed capital","equity","False","Capital subscrito"
"chart_521","521","332","Nominal value","equity","False","Valor nominal"
"chart_522","522","333","Discounts and bonuses","equity","False","Descontos e prémios"
"chart_53","53","334","Other instruments of equity","equity","False","Outros instrumentos de capital próprio"
"chart_54","54","335","Emission awards","equity","False","Prémios de emissão"
"chart_551","551","336","Legal reserves","equity","False","Reservas legais"
"chart_552","552","337","Other reserves","equity","False","Outras reservas"
"chart_56","56","338","Transited results","equity","False","Resultados transitados"
"chart_5711","5711","339","Transition adjustments","equity","False","Ajustamentos de transição"
"chart_5712","5712","340","Unlavoded profits","equity","False","Lucros não atribuídos"
"chart_5713","5713","341","Arising from other variations in the capitals of the participated","equity","False","Decorrentes de outras variações nos capitais próprios das participadas"
"chart_579","579","342","Others","equity","False","Outros"
"chart_5811","5811","343","Before income tax","equity","False","Antes de imposto sobre o rendimento"
"chart_5812","5812","344","Deferred taxes","equity","False","Ativos por impostos diferidos"
"chart_5891","5891","345","Before income tax","equity","False","Antes de imposto sobre o rendimento"
"chart_5892","5892","346","Deferred taxes","equity","False","Ativos por impostos diferidos"
"chart_591","591","347","Differences in conversion of financial statements","equity","False","Diferenças de conversão de demonstrações financeiras"
"chart_592","592","348","Deferred tax adjustments","equity","False","Ajustamentos por impostos diferidos"
"chart_5931","5931","349","Assigned subsidies","equity","False","Subsídios atribuídos"
"chart_5932","5932","350","Subsidies adjustments","equity","False","Ajustamentos em subsídios"
"chart_594","594","351","Donations","equity","False","Doações"
"chart_599","599","352","Others","equity","False","Outros"
"chart_611","611","353","Goods","expense","False","Mercadorias"
"chart_612","612","354","Raw materials, subsidiaries and consumption","expense","False","Matérias-primas, subsidiárias e de consumo"
"chart_613","613","355","Biological assets (Purchases)","expense","False","Ativos biológicos (compras)"
"chart_621","621","356","Subcontracts","expense","False","Subcontratos"
"chart_6221","6221","357","Specialized jobs","expense","False","Trabalhos especializados"
"chart_6222","6222","358","Advertising and marketing","expense","False","Publicidade e propaganda"
"chart_6223","6223","359","Surveillance and security","expense","False","Vigilância e segurança"
"chart_6224","6224","360","Fees","expense","False","Honorários"
"chart_6225","6225","361","Commissions","expense","False","Comissões"
"chart_6226","6226","362","Conservation and repair","expense","False","Conservação e reparação"
"chart_6228","6228","363","Others","expense","False","Outros"
"chart_6231","6231","364","Quick wear tools and utensils","expense","False","Ferramentas e utensílios de desgaste rápido"
"chart_6232","6232","365","Books and technical documentation","expense","False","Livros e documentação técnica"
"chart_6233","6233","366","Office supplies","expense","False","Material de escritório"
"chart_6234","6234","367","Things on sale","expense","False","Artigos para oferta"
"chart_6238","6238","368","Others","expense","False","Outros"
"chart_6241","6241","369","Electricity","expense","False","Eletricidade"
"chart_6242","6242","370","Fuels","expense","False","Combustíveis"
"chart_6243","6243","371","Water","expense","False","Água"
"chart_6248","6248","372","Others","expense","False","Outros"
"chart_6251","6251","373","Travel and stays","expense","False","Deslocações e estadas"
"chart_6252","6252","374","Staff transport","expense","False","Transportes de pessoal"
"chart_6253","6253","375","Transport of goods","expense","False","Transportes de mercadorias"
"chart_6258","6258","376","Others","expense","False","Outros"
"chart_6261","6261","377","Incomes and rents","expense","False","Rendas e alugueres"
"chart_6262","6262","378","Communication","expense","False","Comunicação"
"chart_6263","6263","379","Insurance","expense","False","Seguros"
"chart_6264","6264","380","Royalties","expense","False","Royalties"
"chart_6265","6265","381","Litigation and notary","expense","False","Contencioso e notariado"
"chart_6266","6266","382","Representation expenses","expense","False","Despesas de representação"
"chart_6267","6267","383","Cleaning, hygiene and confort","expense","False","Limpeza, higiene e conforto"
"chart_6268","6268","384","Other services","expense","False","Outros serviços"
"chart_631","631","385","Corporate bodies remuneration","expense","False","Remunerações dos órgãos sociais"
"chart_632","632","386","Staff remuneration","expense","False","Remunerações do pessoal"
"chart_6331","6331","387","Pension allowances","expense","False","Prémios para pensões"
"chart_6332","6332","388","Other benefits","expense","False","Outros benefícios"
"chart_634","634","389","Compensation","expense","False","Indemnizações"
"chart_635","635","390","Remuneration charges","expense","False","Encargos sobre remunerações"
"chart_636","636","391","Accidents insurance at work and professional illnesses","expense","False","Seguros de acidentes no trabalho e doenças profissionais"
"chart_637","637","392","Social action expenses","expense","False","Gastos de ação social"
"chart_638","638","393","Other spending on staff","expense","False","Outros gastos com o pessoal"
"chart_6411","6411","394","Land and natural resources","expense_depreciation","False","Terrenos e recursos naturais"
"chart_6412","6412","395","Buildings and other constructions","expense_depreciation","False","Edifícios e outras construções"
"chart_6413","6413","396","Other investment properties","expense_depreciation","False","Outras propriedades de investimento"
"chart_6421","6421","397","Land and natural resources","expense_depreciation","False","Terrenos e recursos naturais"
"chart_6422","6422","398","Buildings and other constructions","expense_depreciation","False","Edifícios e outras construções"
"chart_6423","6423","399","Basic equipment","expense_depreciation","False","Equipamento básico"
"chart_6424","6424","400","Transport equipment","expense_depreciation","False","Equipamento de transporte"
"chart_6425","6425","401","Office equipment","expense_depreciation","False","Equipamento administrativo"
"chart_6426","6426","402","Biological equipment","expense_depreciation","False","Equipamentos biológicos"
"chart_6427","6427","403","Other tangible fixed assets","expense_depreciation","False","Outros ativos fixos tangíveis"
"chart_6431","6431","404","Goodwill","expense_depreciation","False","Goodwill"
"chart_6432","6432","405","Development projects","expense_depreciation","False","Projetos de desenvolvimento"
"chart_6433","6433","406","Computer programs","expense_depreciation","False","Programas de computador"
"chart_6434","6434","407","Industrial property","expense_depreciation","False","Propriedade industrial"
"chart_6435","6435","408","Other intangible assets","expense_depreciation","False","Outros ativos intangíveis"
"chart_6428","6428","409","Consumables","expense_depreciation","False","Consumíveis"
"chart_6429","6429","410","Of production","expense_depreciation","False","De produção"
"chart_6414","6414","411","Goodwill","expense_depreciation","False","Goodwill"
"chart_6511","6511","413","Customers","expense","False","Clientes"
"chart_6512","6512","414","Other debtors","expense","False","Outros devedores"
"chart_6521","6521","415","Goods","expense","False","Mercadorias"
"chart_6522","6522","416","Raw materials, subsidiaries and consumption","expense","False","Matérias-primas, subsidiárias e de consumo"
"chart_6523","6523","417","Finished and intermediate products","expense","False","Produtos acabados e intermédios"
"chart_6524","6524","418","Byproducts, waste, residues and refuses","expense","False","Subprodutos, desperdícios, resíduos e refugos"
"chart_6525","6525","419","Products and work in progress","expense","False","Produtos e trabalhos em curso"
"chart_6526","6526","420","Consumable biological assets","expense","False","Ativos biológicos consumíveis"
"chart_6527","6527","421","Biological production assets","expense","False","Ativos biológicos de produção"
"chart_6531","6531","422","Capital Participations","expense","False","Participações de capital"
"chart_6532","6532","423","Loans granted","expense","False","Empréstimos concedidos"
"chart_6533","6533","424","Other financial investments","expense","False","Outros investimentos financeiros"
"chart_6534","6534","425","Goodwill","expense","False","Goodwill"
"chart_6541","6541","426","Land and natural resources","expense","False","Terrenos e recursos naturais"
"chart_6542","6542","427","Buildings and other constructions","expense","False","Edifícios e outras construções"
"chart_6543","6543","428","Other investment properties","expense","False","Outras propriedades de investimento"
"chart_6551","6551","429","Land and natural resources","expense","False","Terrenos e recursos naturais"
"chart_6552","6552","430","Buildings and other constructions","expense","False","Edifícios e outras construções"
"chart_6553","6553","431","Basic equipment","expense","False","Equipamento básico"
"chart_6554","6554","432","Transport equipment","expense","False","Equipamento de transporte"
"chart_6555","6555","433","Office equipment","expense","False","Equipamento administrativo"
"chart_6556","6556","434","Biological equipment","expense","False","Equipamentos biológicos"
"chart_6557","6557","435","Other tangible fixed assets","expense","False","Outros ativos fixos tangíveis"
"chart_6561","6561","436","Goodwill","expense","False","Goodwill"
"chart_6562","6562","437","Development projects","expense","False","Projetos de desenvolvimento"
"chart_6563","6563","438","Computer programs","expense","False","Programas de computador"
"chart_6564","6564","439","Industrial property","expense","False","Propriedade industrial"
"chart_6565","6565","440","Other intangible assets","expense","False","Outros ativos intangíveis"
"chart_6571","6571","441","Financial investments ongoing","expense","False","Investimentos financeiros em curso"
"chart_6572","6572","442","Investment properties in progress","expense","False","Propriedades de investimento em curso"
"chart_6573","6573","443","Tangible fixed assets ongoing","expense","False","Ativos fixos tangíveis em curso"
"chart_6574","6574","444","Intangible assets ongoing","expense","False","Ativos intangíveis em curso"
"chart_65751","65751","445","Financial investments ongoing","expense","False","Investimentos financeiros em curso"
"chart_65752","65752","446","Investment properties in progress","expense","False","Propriedades de investimento em curso"
"chart_65753","65753","447","Tangible fixed assets ongoing","expense","False","Ativos fixos tangíveis em curso"
"chart_65754","65754","448","Intangible assets ongoing","expense","False","Ativos intangíveis em curso"
"chart_6581","6581","449","Financial investments","expense","False","Investimentos Financeiros"
"chart_6582","6582","450","Investment properties","expense","False","Propriedades de investimento"
"chart_6583","6583","451","Tangible fixed assets","expense","False","Ativos fixos tangíveis"
"chart_6584","6584","452","Intangible assets","expense","False","Ativos intangíveis"
"chart_6585","6585","453","Others","expense","False","Outros"
"chart_661","661","454","In financial instruments","expense","False","Em instrumentos financeiros"
"chart_6621","6621","455","Capital Participations","expense","False","Participações de capital"
"chart_6622","6622","456","Other financial investments","expense","False","Outros investimentos financeiros"
"chart_6631","6631","457","Land and natural resources","expense","False","Terrenos e recursos naturais"
"chart_6632","6632","458","Buildings and other constructions","expense","False","Edifícios e outras construções"
"chart_6633","6633","459","Other investment properties","expense","False","Outras propriedades de investimento"
"chart_6634","6634","460","Investment properties in progress","expense","False","Propriedades de investimento em curso"
"chart_6641","6641","461","Consumables","expense","False","Consumíveis"
"chart_6642","6642","462","Of production","expense","False","De produção"
"chart_671","671","463","Taxes","expense","False","Impostos"
"chart_672","672","464","Guarantees to customers","expense","False","Garantias a clientes"
"chart_673","673","465","Ongoing judicial proceedings","expense","False","Processos judiciais em curso"
"chart_674","674","466","Accidents at work and professional diseases","expense","False","Acidentes no trabalho e doenças profissionais"
"chart_675","675","467","Environmental materials","expense","False","Matérias ambientais"
"chart_676","676","468","Costly","expense","False","Contratos onerosos"
"chart_677","677","469","Restructuring","expense","False","Reestruturação"
"chart_678","678","470","Other provisions","expense","False","Outras provisões"
"chart_6811","6811","471","Direct taxes","expense","False","Impostos diretos"
"chart_6812","6812","472","Indirect taxes","expense","False","Impostos indiretos"
"chart_6813","6813","473","Rates","expense","False","Taxas"
"chart_682","682","474","Discounts granted","expense","False","Descontos de pronto pagamento concedidos"
"chart_683","683","475","Irrecoverable debts","expense","False","Dívidas incobráveis"
"chart_6841","6841","476","Sinister","expense","False","Sinistros"
"chart_6842","6842","477","Breakdown","expense","False","Quebras"
"chart_6848","6848","478","Other losses","expense","False","Outras perdas"
"chart_6851","6851","479","Damage","expense","False","Cobertura de prejuízos"
"chart_6852","6852","480","Application of the equity method","expense","False","Aplicação do método da equivalência patrimonial"
"chart_6853","6853","481","Alienations","expense","False","Alienações"
"chart_6858","6858","482","Other expenses","expense","False","Outros gastos"
"chart_6861","6861","483","Damage","expense","False","Cobertura de prejuízos"
"chart_6862","6862","484","Alienations","expense","False","Alienações"
"chart_6863","6863","485","Unfavorable exchange rate differences","expense","False","Diferenças de câmbio desfavoráveis"
"chart_6868","6868","486","Other expenses","expense","False","Outros gastos"
"chart_6871","6871","487","Alienations","expense","False","Alienações"
"chart_6872","6872","488","Sinister","expense","False","Sinistros"
"chart_6873","6873","489","Slaughter","expense","False","Abates"
"chart_6874","6874","490","Spending on investment properties","expense","False","Gastos em propriedades de investimento"
"chart_6878","6878","491","Other expenses","expense","False","Outros gastos"
"chart_6881","6881","492","Corrections related to previous periods","expense","False","Correções relativas a períodos anteriores"
"chart_6882","6882","493","Donations","expense","False","Doações"
"chart_6883","6883","494","Quotanizations","expense","False","Quotizações"
"chart_6884","6884","495","Inventory offers and samples","expense","False","Ofertas e amostras de inventários"
"chart_6885","6885","496","Tax estimation insufficiency","expense","False","Insuficiência da estimativa para impostos"
"chart_6886","6886","497","Losses in financial instruments","expense","False","Perdas em instrumentos financeiros"
"chart_6887","6887","498","Unfavorable exchange rate differences","expense","False","Diferenças de câmbio desfavoráveis"
"chart_6888","6888","499","Others not specified","expense","False","Outros não especificados"
"chart_6911","6911","500","Interest of financing obtained","expense","False","Juros de financiamentos obtidos"
"chart_6918","6918","501","Other interest","expense","False","Outros juros"
"chart_6921","6921","502","Relating to financing obtained","expense","False","Relativas a financiamentos obtidos"
"chart_6928","6928","503","Others","expense","False","Outros"
"chart_6981","6981","504","Relating to financing obtained","expense","False","Relativas a financiamentos obtidos"
"chart_6988","6988","505","Others","expense","False","Outros"
"chart_711","711","506","Goods","income","False","Mercadorias"
"chart_712","712","507","Finished and intermediate products","income","False","Produtos acabados e intermédios"
"chart_713","713","508","Byproducts, waste, residues and refuses","income","False","Subprodutos, desperdícios, resíduos e refugos"
"chart_714","714","509","Biological assets","income","False","Ativos biológicos"
"chart_716","716","510","VAT of tax sales included","income","False","IVA das vendas com imposto incluído"
"chart_717","717","511","Sales returns","income","False","Devoluções de vendas"
"chart_718","718","512","Sales discounts and slaughter","income","False","Descontos e abatimentos em vendas"
"chart_721","721","513","Service to","income","False","Serviço A"
"chart_722","722","514","Service b","income","False","Serviço B"
"chart_723","723","515","Other services","income","False","Outros serviços"
"chart_725","725","516","Secondary services","income","False","Serviços secundários"
"chart_726","726","517","VAT of services with tax included","income","False","IVA dos serviços com imposto incluído"
"chart_728","728","518","Discounts and rebates","income","False","Descontos e abatimentos"
"chart_731","731","519","Finished and intermediate products","income","False","Produtos acabados e intermédios"
"chart_732","732","520","Byproducts, waste, residues and refuses","income","False","Subprodutos, desperdícios, resíduos e refugos"
"chart_733","733","521","Products and work in progress","income","False","Produtos e trabalhos em curso"
"chart_734","734","522","Biological assets","income","False","Ativos biológicos"
"chart_741","741","523","Tangible fixed assets","income","False","Ativos fixos tangíveis"
"chart_742","742","524","Intangible assets","income","False","Ativos intangíveis"
"chart_743","743","525","Investment properties","income","False","Propriedades de investimento"
"chart_744","744","526","Active for deferred spending","income","False","Ativos por gastos diferidos"
"chart_751","751","527","Subsidies of public entities","income","False","Subsídios das entidades públicas"
"chart_752","752","528","Subsidies of other entities","income","False","Subsídios de outras entidades"
"chart_76111","76111","529","Land and natural resources","income","False","Terrenos e recursos naturais"
"chart_76112","76112","530","Buildings and other constructions","income","False","Edifícios e outras construções"
"chart_76113","76113","531","Other investment properties","income","False","Outras propriedades de investimento"
"chart_76121","76121","532","Land and natural resources","income","False","Terrenos e recursos naturais"
"chart_76122","76122","533","Buildings and other constructions","income","False","Edifícios e outras construções"
"chart_76123","76123","534","Basic equipment","income","False","Equipamento básico"
"chart_76124","76124","535","Transport equipment","income","False","Equipamento de transporte"
"chart_76125","76125","536","Office equipment","income","False","Equipamento administrativo"
"chart_76126","76126","537","Biological equipment","income","False","Equipamentos biológicos"
"chart_76127","76127","538","Other tangible fixed assets","income","False","Outros ativos fixos tangíveis"
"chart_76131","76131","539","Goodwill","income","False","Goodwill"
"chart_76132","76132","540","Development projects","income","False","Projetos de desenvolvimento"
"chart_76133","76133","541","Computer programs","income","False","Programas de computador"
"chart_76134","76134","542","Industrial property","income","False","Propriedade industrial"
"chart_76135","76135","543","Other intangible assets","income","False","Outros ativos intangíveis"
"chart_76141","76141","544","Consumables","income","False","Consumíveis"
"chart_76142","76142","545","Of production","income","False","De produção"
"chart_76151","76151","546","Goodwill","income","False","Goodwill"
"chart_76211","76211","547","Customers","income","False","Clientes"
"chart_76212","76212","548","Other debtors","income","False","Outros devedores"
"chart_76221","76221","549","Goods","income","False","Mercadorias"
"chart_76222","76222","550","Raw materials, subsidiaries and consumption","income","False","Matérias-primas, subsidiárias e de consumo"
"chart_76223","76223","551","Finished and intermediate products","income","False","Produtos acabados e intermédios"
"chart_76224","76224","552","Byproducts, waste, residues and refuses","income","False","Subprodutos, desperdícios, resíduos e refugos"
"chart_76225","76225","553","Products and work in progress","income","False","Produtos e trabalhos em curso"
"chart_76226","76226","554","Consumable biological assets","income","False","Ativos biológicos consumíveis"
"chart_76227","76227","555","Biological production assets","income","False","Ativos biológicos de produção"
"chart_76231","76231","556","Capital Participations","income","False","Participações de capital"
"chart_76232","76232","557","Loans granted","income","False","Empréstimos concedidos"
"chart_76233","76233","558","Other financial investments","income","False","Outros investimentos financeiros"
"chart_76241","76241","559","Land and natural resources","income","False","Terrenos e recursos naturais"
"chart_76242","76242","560","Buildings and other constructions","income","False","Edifícios e outras construções"
"chart_76243","76243","561","Other investment properties","income","False","Outras propriedades de investimento"
"chart_76251","76251","562","Land and natural resources","income","False","Terrenos e recursos naturais"
"chart_76252","76252","563","Buildings and other constructions","income","False","Edifícios e outras construções"
"chart_76253","76253","564","Basic equipment","income","False","Equipamento básico"
"chart_76254","76254","565","Transport equipment","income","False","Equipamento de transporte"
"chart_76255","76255","566","Office equipment","income","False","Equipamento administrativo"
"chart_76256","76256","567","Biological equipment","income","False","Equipamentos biológicos"
"chart_76257","76257","568","Other tangible fixed assets","income","False","Outros ativos fixos tangíveis"
"chart_76261","76261","569","Development projects","income","False","Projetos de desenvolvimento"
"chart_726262","726262","570","Computer programs","income","False","Programas de computador"
"chart_76263","76263","571","Industrial property","income","False","Propriedade industrial"
"chart_76264","76264","572","Other intangible assets","income","False","Outros ativos intangíveis"
"chart_76271","76271","573","Financial investments ongoing","income","False","Investimentos financeiros em curso"
"chart_76272","76272","574","Investment properties in progress","income","False","Propriedades de investimento em curso"
"chart_76273","76273","575","Tangible fixed assets ongoing","income","False","Ativos fixos tangíveis em curso"
"chart_76274","76274","576","Intangible assets ongoing","income","False","Ativos intangíveis em curso"
"chart_762751","762751","577","Financial investments ongoing","income","False","Investimentos financeiros em curso"
"chart_762752","762752","578","Investment properties in progress","income","False","Propriedades de investimento em curso"
"chart_762753","762753","579","Tangible fixed assets ongoing","income","False","Ativos fixos tangíveis em curso"
"chart_762754","762754","580","Intangible assets ongoing","income","False","Ativos intangíveis em curso"
"chart_76281","76281","581","Financial investments","income","False","Investimentos Financeiros"
"chart_76282","76282","582","Investment properties","income","False","Propriedades de investimento"
"chart_76283","76283","583","Tangible fixed assets","income","False","Ativos fixos tangíveis"
"chart_76284","76284","584","Intangible assets","income","False","Ativos intangíveis"
"chart_76285","76285","585","Others","income","False","Outros"
"chart_7631","7631","586","Taxes","income","False","Impostos"
"chart_7632","7632","587","Guarantees to customers","income","False","Garantias a clientes"
"chart_7633","7633","588","Ongoing judicial proceedings","income","False","Processos judiciais em curso"
"chart_7634","7634","589","Accidents at work and professional diseases","income","False","Acidentes no trabalho e doenças profissionais"
"chart_7635","7635","590","Environmental materials","income","False","Matérias ambientais"
"chart_7636","7636","591","Costly","income","False","Contratos onerosos"
"chart_7637","7637","592","Restructuring","income","False","Reestruturação"
"chart_7638","7638","593","Other provisions","income","False","Outras provisões"
"chart_771","771","594","In financial instruments","income","False","Em instrumentos financeiros"
"chart_7721","7721","595","Capital Participations","income","False","Participações de capital"
"chart_7722","7722","596","Other financial investments","income","False","Outros investimentos financeiros"
"chart_7731","7731","597","Land and natural resources","income","False","Terrenos e recursos naturais"
"chart_7732","7732","598","Buildings and other constructions","income","False","Edifícios e outras construções"
"chart_7733","7733","599","Other investment properties","income","False","Outras propriedades de investimento"
"chart_7734","7734","600","Investment properties in progress","income","False","Propriedades de investimento em curso"
"chart_7741","7741","601","Consumables","income","False","Consumíveis"
"chart_7742","7742","602","Of production","income","False","De produção"
"chart_7811","7811","603","Social services","income","False","Serviços sociais"
"chart_7812","7812","604","Equipment rental","income","False","Aluguer de equipamento"
"chart_7813","7813","605","Studies, projects and technological assistance","income","False","Estudos, projetos e assistência tecnológica"
"chart_7814","7814","606","Royalties","income","False","Royalties"
"chart_7815","7815","607","Performance of social positions in other companies","income","False","Desempenho de cargos sociais noutras empresas"
"chart_7816","7816","608","Other supplementary income","income","False","Outros rendimentos suplementares"
"chart_782","782","609","Discounts obtained","income","False","Descontos de pronto pagamento obtidos"
"chart_783","783","610","Debt recovery receivable","income","False","Recuperação de dívidas a receber"
"chart_7841","7841","611","Sinister","income","False","Sinistros"
"chart_7842","7842","612","Leftover","income","False","Sobras"
"chart_7848","7848","613","Other gains","income","False","Outros ganhos"
"chart_7851","7851","614","Application of the equity method","income","False","Aplicação do método da equivalência patrimonial"
"chart_7852","7852","615","Alienations","income","False","Alienações"
"chart_7858","7858","616","Other income","income","False","Outros rendimentos"
"chart_7861","7861","617","Favorable exchange rate differences","income","False","Diferenças de câmbio favoráveis"
"chart_7862","7862","618","Alienations","income","False","Alienações"
"chart_7868","7868","619","Other income","income","False","Outros rendimentos"
"chart_7871","7871","620","Alienations","income","False","Alienações"
"chart_7872","7872","621","Sinister","income","False","Sinistros"
"chart_7873","7873","622","Income and other income in investment properties","income","False","Rendas e outros rendimentos em propriedades de investimento"
"chart_7878","7878","623","Other income","income","False","Outros rendimentos"
"chart_7881","7881","624","Corrections related to previous periods","income","False","Correções relativas a períodos anteriores"
"chart_7882","7882","625","Excessive estimate for taxes","income","False","Excesso da estimativa para impostos"
"chart_7883","7883","626","Imputation of subsidies for investments","income","False","Imputação de subsídios para investimentos"
"chart_7884","7884","627","Gains in other financial instruments","income","False","Ganhos em outros instrumentos financeiros"
"chart_7885","7885","628","Tax Refund","income","False","Restituição de impostos"
"chart_7887","7887","629","Favorable exchange rate differences","income","False","Diferenças de câmbio favoráveis"
"chart_7888","7888","630","Others not specified","income","False","Outros não especificados"
"chart_7911","7911","631","Deposits","income","False","De depósitos"
"chart_7912","7912","632","Other applications of net financial means","income","False","De outras aplicações de meios financeiros líquidos"
"chart_7913","7913","633","Financing granted to associates and joint enterprises","income","False","De financiamentos concedidos a associadas e empreendimentos conjuntos"
"chart_7914","7914","634","Of financing granted to subsidiaries","income","False","De financiamentos concedidos a subsidiárias"
"chart_7915","7915","635","Of financing obtained","income","False","De financiamentos obtidos"
"chart_7918","7918","636","Other financing granted","income","False","De outros financiamentos concedidos"
"chart_7921","7921","637","Of applications of net financial means","income","False","De aplicações de meios financeiros líquidos"
"chart_7922","7922","638","Of associates and joint enterprises","income","False","De associadas e empreendimentos conjuntos"
"chart_7923","7923","639","Of subsidiaries","income","False","Matérias subsidiárias"
"chart_7928","7928","640","Others","income","False","Outros"
"chart_793","793","641","Favorable exchange rate differences","income","False","Diferenças de câmbio favoráveis"
"chart_798","798","642","Other similar income","income","False","Outros rendimentos similares"
"chart_811","811","643","Result before taxes","liability_current","False","Resultado antes de impostos"
"chart_8121","8121","644","Estimated tax for the period","liability_current","False","Imposto estimado para o período"
"chart_8122","8122","645","Deferred tax","liability_current","False","Imposto diferido"
"chart_818","818","646","Net income","equity_unaffected","False","Resultado líquido"
"chart_89","89","647","Early dividends","equity","False","Dividendos antecipados"

```

## File: data\template\account.fiscal.position-pt.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@pt"
"fiscal_position_national_customers","1","Portugal","1","1","base.pt","","","","Portugal"
"fiscal_position_foreign_eu_private","2","Private EU","1","","","base.europe","","","Intra-comunitário privado (UE)"
"fiscal_position_foreign_eu","3","Inside EU","1","1","","base.europe","iva_pt_sale_normal","iva_pt_sale_eu_isenta","Intra-comunitário (UE)"
"","","","","","","","iva_pt_sale_intermedia","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_sale_reduzida","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_sale_isenta","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_ma_sale_normal","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_ma_sale_intermedia","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_ma_sale_reduzida","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_ac_sale_normal","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_ac_sale_intermedia","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_ac_sale_reduzida","iva_pt_sale_eu_isenta",""
"","","","","","","","iva_pt_purchase_normal","iva_pt_purchase_eu_normal_bens",""
"","","","","","","","iva_pt_purchase_intermedia","iva_pt_purchase_eu_intermedia_bens",""
"","","","","","","","iva_pt_purchase_reduzida","iva_pt_purchase_eu_reduzida_bens",""
"","","","","","","","iva_pt_purchase_isenta","iva_pt_purchase_eu_isenta",""
"","","","","","","","iva_pt_ma_purchase_normal","iva_pt_ma_purchase_eu_normal_bens",""
"","","","","","","","iva_pt_ma_purchase_intermedia","iva_pt_ma_purchase_eu_intermedia_bens",""
"","","","","","","","iva_pt_ma_purchase_reduzida","iva_pt_ma_purchase_eu_reduzida_bens",""
"","","","","","","","iva_pt_ac_purchase_normal","iva_pt_ac_purchase_eu_normal_bens",""
"","","","","","","","iva_pt_ac_purchase_intermedia","iva_pt_ac_purchase_eu_intermedia_bens",""
"","","","","","","","iva_pt_ac_purchase_reduzida","iva_pt_ac_purchase_eu_reduzida_bens",""
"fiscal_position_foreign_other","4","Outside EU","1","","","","iva_pt_sale_normal","iva_pt_sale_non_eu_isenta","Extra-comunitário (Não UE)"
"","","","","","","","iva_pt_sale_intermedia","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_sale_reduzida","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_sale_isenta","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_ma_sale_normal","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_ma_sale_intermedia","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_ma_sale_reduzida","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_ac_sale_normal","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_ac_sale_intermedia","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_ac_sale_reduzida","iva_pt_sale_non_eu_isenta",""
"","","","","","","","iva_pt_purchase_normal","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_purchase_intermedia","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_purchase_reduzida","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_purchase_isenta","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_ma_purchase_normal","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_ma_purchase_intermedia","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_ma_purchase_reduzida","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_ac_purchase_normal","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_ac_purchase_intermedia","iva_pt_purchase_non_eu_isenta",""
"","","","","","","","iva_pt_ac_purchase_reduzida","iva_pt_purchase_non_eu_isenta",""

```

## File: data\template\account.group-pt.csv

```csv
"id","code_prefix_start","name","name@pt"
"chart_group_1","1","Liquid financial means","Meios financeiros líquidos"
"chart_group_14","14","Other Financial Instruments","Outros instrumentos financeiros"
"chart_group_141","141","Byproducts","Subprodutos"
"chart_group_142","142","Financial instruments held for negotiation","Instrumentos financeiros detidos para negociação"
"chart_group_143","143","Other assets and financial liabilities","Outros ativos e passivos financeiros"
"chart_group_2","2","Accounts receivable and payable","Contas a receber e a pagar"
"chart_group_21","21","Customers","Clientes"
"chart_group_211","211","Customers c/c","Clientes c/c"
"chart_group_212","212","Customers - Titles Receivable","Clientes - títulos a receber"
"chart_group_219","219","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_2191","2191","Customers c/c","Clientes c/c"
"chart_group_2192","2192","Customers - Titles Receivable","Clientes - títulos a receber"
"chart_group_22","22","Suppliers","Fornecedores"
"chart_group_221","221","Suppliers c/c","Fornecedores c/c"
"chart_group_222","222","Suppliers - Titles payable","Fornecedores - títulos a pagar"
"chart_group_23","23","Staff","Ao pessoal"
"chart_group_231","231","Remunerations to be paid","Remunerações a pagar"
"chart_group_232","232","Advance","Adiantamentos"
"chart_group_2331","2331","With the corporate bodies","Com os órgãos sociais"
"chart_group_2332","2332","With the staff","Com o pessoal"
"chart_group_2341","2341","With the corporate bodies","Com os órgãos sociais"
"chart_group_2342","2342","With the staff","Com o pessoal"
"chart_group_2351","2351","With the corporate bodies","Com os órgãos sociais"
"chart_group_2352","2352","With the staff","Com o pessoal"
"chart_group_2361","2361","With the corporate bodies","Com os órgãos sociais"
"chart_group_2362","2362","With the staff","Com o pessoal"
"chart_group_237","237","Guarantees","Cauções"
"chart_group_2371","2371","Of corporate bodies","Dos órgãos sociais"
"chart_group_2372","2372","Of staff","Do pessoal"
"chart_group_238","238","Other operations","Outras operações"
"chart_group_2381","2381","With the corporate bodies","Com os órgãos sociais"
"chart_group_2382","2382","With the staff","Com o pessoal"
"chart_group_239","239","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_2391","2391","Advance","Adiantamentos"
"chart_group_2392","2392","Other operations","Outras operações"
"chart_group_23921","23921","To corporate bodies","Aos órgãos sociais"
"chart_group_23922","23922","To staff","Ao pessoal"
"chart_group_24","24","State and other public entities","Estado e outros entes públicos"
"chart_group_243","243","Value added tax (VAT)","Imposto sobre o valor acrescentado (IVA)"
"chart_group_25","25","Financing obtained","Financiamentos obtidos"
"chart_group_251","251","Credit institutions and financial companies","Instituições de crédito e sociedades financeiras"
"chart_group_2511","2511","Bank loans","Empréstimos bancários"
"chart_group_252","252","Securities market","Mercado de valores mobiliários"
"chart_group_2521","2521","Loans by obligations","Empréstimos por obrigações"
"chart_group_253","253","Capital participants","Participantes de capital"
"chart_group_2531","2531","Mother company - Supplies and other mutual","Empresa-mãe - Suprimentos e outros mútuos"
"chart_group_2532","2532","Other participants - Supplies and other mutual","Outros participantes - Suprimentos e outros mútuos"
"chart_group_254","254","Subsidiaries, associates and joint enterprises","Subsidiárias, associadas e empreendimentos conjuntos"
"chart_group_258","258","Other funders","Outros financiadores"
"chart_group_26","26","Shareholders/partners","Acionistas/sócios"
"chart_group_266","266","Loans granted - Mother company","Empréstimos concedidos - empresa-mãe"
"chart_group_268","268","Other operations","Outras operações"
"chart_group_269","269","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_2696","2696","Loans granted - Mother company","Empréstimos concedidos - empresa-mãe"
"chart_group_2697","2697","Other operations","Outras operações"
"chart_group_27","27","Other receivable and payable accounts","Outras contas a receber e a pagar"
"chart_group_271","271","Investment suppliers","Fornecedores de investimentos"
"chart_group_2711","2711","Investment suppliers - General accounts","Fornecedores de investimentos - contas gerais"
"chart_group_2712","2712","Invoices received and checked","Faturas em receção e conferência"
"chart_group_2713","2713","Advances to investment suppliers","Adiantamentos a fornecedores de investimentos"
"chart_group_272","272","Debtors and creditors by additions (economic periodization)","Devedores e credores por acréscimos (periodização económica)"
"chart_group_274","274","Deferred taxes","Ativos por impostos diferidos"
"chart_group_275","275","Creditors for unreleased subscriptions","Credores por subscrições não liberadas"
"chart_group_278","278","Other debtors and creditors","Outros devedores e credores"
"chart_group_279","279","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_2791","2791","Advances to investment suppliers","Adiantamentos a fornecedores de investimentos"
"chart_group_2794","2794","Other debtors and creditors","Outros devedores e credores"
"chart_group_28","28","Deferments","Diferimentos"
"chart_group_29","29","Provisions","Provisões"
"chart_group_3","3","Inventories and biological assets","Inventários e ativos biológicos"
"chart_group_31","31","Purchases","Compras"
"chart_group_317","317","Purchase returns","Devoluções de compras"
"chart_group_318","318","Discounts and rebates on purchases","Descontos e abatimentos em compras"
"chart_group_329","329","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_33","33","Raw materials, subsidiaries and consumption","Matérias-primas, subsidiárias e de consumo"
"chart_group_339","339","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_34","34","Finished and intermediate products","Produtos acabados e intermédios"
"chart_group_349","349","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_35","35","Byproducts, waste, residues and refuses","Subprodutos, desperdícios, resíduos e refugos"
"chart_group_359","359","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_36","36","Products and work in progress","Produtos e trabalhos em curso"
"chart_group_37","37","Biological assets","Ativos biológicos"
"chart_group_371","371","Consumables","Consumíveis"
"chart_group_372","372","Of production","De produção"
"chart_group_378","378","Accumulated depreciation","Depreciações acumuladas"
"chart_group_379","379","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_39","39","Down payments on account of purchases","Adiantamentos por conta de compras"
"chart_group_4","4","Investment","Investimentos"
"chart_group_41","41","Financial investments","Investimentos Financeiros"
"chart_group_411","411","Investments in subsidiaries","Investimentos em subsidiárias"
"chart_group_4111","4111","Capital participations - Equivalence method","Participações de capital - método da equivalência patrimonial"
"chart_group_412","412","Investments in associates","Investimentos em associadas"
"chart_group_4121","4121","Capital participations - Equivalence method","Participações de capital - método da equivalência patrimonial"
"chart_group_413","413","Investments in jointly controlled entities","Investimentos em entidades conjuntamente controladas"
"chart_group_414","414","Investments in other companies","Investimentos noutras empresas"
"chart_group_415","415","Other financial investments","Outros investimentos financeiros"
"chart_group_418","418","Accumulated amortization","Amortizações acumuladas"
"chart_group_4181","4181","Capital participations - Equivalence method","Participações de capital - método da equivalência patrimonial"
"chart_group_41813","41813","Investments in jointly controlled entities","Investimentos em entidades conjuntamente controladas"
"chart_group_419","419","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_4191","4191","Investments in subsidiaries","Investimentos em subsidiárias"
"chart_group_41911","41911","Capital participations - Equivalence method","Participações de capital - método da equivalência patrimonial"
"chart_group_4192","4192","Investments in associates","Investimentos em associadas"
"chart_group_41921","41921","Capital participations - Equivalence method","Participações de capital - método da equivalência patrimonial"
"chart_group_4193","4193","Investments in jointly controlled entities","Investimentos em entidades conjuntamente controladas"
"chart_group_41931","41931","Capital participations - Equivalence method","Participações de capital - método da equivalência patrimonial"
"chart_group_4194","4194","Investments in other companies","Investimentos noutras empresas"
"chart_group_4195","4195","Other financial investments","Outros investimentos financeiros"
"chart_group_42","42","Investment properties","Propriedades de investimento"
"chart_group_428","428","Accumulated depreciation","Depreciações acumuladas"
"chart_group_429","429","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_43","43","Tangible fixed assets","Ativos fixos tangíveis"
"chart_group_438","438","Accumulated depreciation","Depreciações acumuladas"
"chart_group_439","439","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_44","44","Intangible assets","Ativos intangíveis"
"chart_group_448","448","Accumulated amortization","Amortizações acumuladas"
"chart_group_449","449","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_45","45","Ongoing investments","Investimentos em curso"
"chart_group_455","455","Advances on account of investments","Adiantamentos por conta de investimentos"
"chart_group_459","459","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_4595","4595","Advances on account of investments","Adiantamentos por conta de investimentos"
"chart_group_46","46","Non-current assets held for sale","Ativos não correntes detidos para venda"
"chart_group_469","469","Accumulated impairment losses","Perdas por imparidade acumuladas"
"chart_group_5","5","Capital, reserves and retained earnings","Capital, Reservas e Resultados Transitados"
"chart_group_52","52","Own shares","Ações (quotas) próprias"
"chart_group_55","55","Reserves","Reservas"
"chart_group_57","57","Adjustments in financial assets","Ajustamentos em ativos financeiros"
"chart_group_571","571","Related to the equivalence method","Relacionados com o método da equivalência patrimonial"
"chart_group_581","581","Reevaluations resulting from legal diplomas","Reavaliações decorrentes de diplomas legais"
"chart_group_589","589","Other surplus","Outros excedentes"
"chart_group_59","59","Other variations in equity","Outras variações no capital próprio"
"chart_group_593","593","Subsidies","Subsídios"
"chart_group_6","6","Expenses","Gastos"
"chart_group_61","61","Cost of goods sold and materials consumed","Custo das mercadorias vendidas e das matérias consumidas"
"chart_group_62","62","Supplies and external services","Fornecimentos e serviços externos"
"chart_group_622","622","Specialized services","Serviços especializados"
"chart_group_623","623","Materials","Materiais"
"chart_group_624","624","Energy and fluids","Energia e fluidos"
"chart_group_625","625","Travel, accommodation and transportation","Deslocações, estadas e transportes"
"chart_group_626","626","Miscellaneous services","Serviços diversos"
"chart_group_63","63","Staff spending","Gastos com o pessoal"
"chart_group_633","633","Post-employment benefits","Benefícios pós-emprego"
"chart_group_64","64","Depreciation and amortization expenses","Gastos de depreciação e de amortização"
"chart_group_641","641","Investment properties","Propriedades de investimento"
"chart_group_642","642","Tangible fixed assets","Ativos fixos tangíveis"
"chart_group_643","643","Intangible assets","Ativos intangíveis"
"chart_group_65","65","Impairment losses","Perdas por imparidade"
"chart_group_651","651","In debt receivable","Em dívidas a receber"
"chart_group_652","652","In inventory","Em inventários"
"chart_group_653","653","In financial investments","Em investimentos financeiros"
"chart_group_654","654","In investment properties","Em propriedades de investimento"
"chart_group_655","655","In tangible fixed assets","Em ativos fixos tangíveis"
"chart_group_656","656","In intangible assets","Em ativos intangíveis"
"chart_group_657","657","In investments in progress","Em investimentos em curso"
"chart_group_6575","6575","Advances on account of investments","Adiantamentos por conta de investimentos"
"chart_group_658","658","In non-current assets held for sale","Em ativos não correntes detidos para venda"
"chart_group_66","66","Losses due to reductions in fair value","Perdas por reduções de justo valor"
"chart_group_663","663","In investment properties","Em propriedades de investimento"
"chart_group_664","664","In biological assets","Em ativos biológicos"
"chart_group_67","67","Provisions of the period","Provisões do período"
"chart_group_68","68","Other expenses","Outros gastos"
"chart_group_681","681","Taxes","Impostos"
"chart_group_684","684","Inventory losses","Perdas em inventários"
"chart_group_686","686","Expenses on remaining financial investments","Gastos nos restantes investimentos financeiros"
"chart_group_687","687","Expenses on non-financial investments","Gastos em investimentos não financeiros"
"chart_group_688","688","Others","Outros"
"chart_group_69","69","Financing expenses","Gastos de financiamento"
"chart_group_691","691","Supported interests","Juros suportados"
"chart_group_692","692","Unfavorable exchange rate differences","Diferenças de câmbio desfavoráveis"
"chart_group_698","698","Other financing expenses","Outros gastos de financiamento"
"chart_group_7","7","Income","Rendimentos"
"chart_group_71","71","Sales","Vendas"
"chart_group_72","72","Services","Prestações de serviços"
"chart_group_73","73","Variations in production inventories","Variações nos inventários da produção"
"chart_group_74","74","Work for the entity itself","Trabalhos para a própria entidade"
"chart_group_75","75","Exploration subsidies","Subsídios à exploração"
"chart_group_76","76","Reversals","Reversões"
"chart_group_761","761","Depreciation and amortization","De depreciações e de amortizações"
"chart_group_7611","7611","Investment properties","Propriedades de investimento"
"chart_group_7612","7612","Tangible fixed assets","Ativos fixos tangíveis"
"chart_group_7613","7613","Intangible assets","Ativos intangíveis"
"chart_group_7614","7614","Biological assets","Ativos biológicos"
"chart_group_7615","7615","Financial investments","Investimentos Financeiros"
"chart_group_762","762","Impairment losses","Perdas por imparidade"
"chart_group_7621","7621","In debt receivable","Em dívidas a receber"
"chart_group_7622","7622","In inventory","Em inventários"
"chart_group_7623","7623","In financial investments","Em investimentos financeiros"
"chart_group_7624","7624","In investment properties","Em propriedades de investimento"
"chart_group_7625","7625","In tangible fixed assets","Em ativos fixos tangíveis"
"chart_group_7626","7626","In intangible assets","Em ativos intangíveis"
"chart_group_7627","7627","In investements in progress","Em investimentos em curso"
"chart_group_76275","76275","Advances on account of investments","Adiantamentos por conta de investimentos"
"chart_group_7628","7628","In non-current assets held for sale","Em ativos não correntes detidos para venda"
"chart_group_763","763","Provisions","Provisões"
"chart_group_77","77","Gains for fair value increases","Ganhos por aumentos de justo valor"
"chart_group_772","772","In financial investments","Em investimentos financeiros"
"chart_group_773","773","In investment properties","Em propriedades de investimento"
"chart_group_774","774","In biological assets","Em ativos biológicos"
"chart_group_78","78","Other income","Outros rendimentos"
"chart_group_781","781","Supplementary income","Rendimentos suplementares"
"chart_group_784","784","Inventory gains","Ganhos em inventários"
"chart_group_786","786","Income in the remaining financial assets","Rendimentos nos restantes ativos financeiros"
"chart_group_787","787","Income in non-financial investments","Rendimentos em investimentos não financeiros"
"chart_group_788","788","Others","Outros"
"chart_group_79","79","Interest, dividends and other similar income","Juros, dividendos e outros rendimentos similares"
"chart_group_791","791","Interest obtained","Juros obtidos"
"chart_group_792","792","Dividends obtained","Dividendos obtidos"
"chart_group_8","8","Results","Resultados"
"chart_group_81","81","Net result of the period","Resultado líquido do período"
"chart_group_812","812","Tax over the period's income","Imposto sobre o rendimento do período"
"chart_group_89","89","Early dividends","Dividendos antecipados"

```

## File: data\template\account.tax-pt.csv

```csv
"id","name","description","invoice_label","sequence","amount","amount_type","type_tax_use","tax_group_id","active","tax_scope","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@pt"
"iva_pt_sale_normal","23%","","VAT 23%","10","23.0","percent","sale","tax_group_iva_23","","","100","base","invoice","+trp_base_3","",""
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_4","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_3","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_4","chart_2433",""
"iva_pt_sale_intermedia","13%","","VAT 13%","20","13.0","percent","sale","tax_group_iva_13","","","100","base","invoice","+trp_base_5","",""
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_6","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_5","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_6","chart_2433",""
"iva_pt_sale_reduzida","6%","","VAT 6%","30","6.0","percent","sale","tax_group_iva_6","","","100","base","invoice","+trp_base_1","",""
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_2","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_1","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_2","chart_2433",""
"iva_pt_sale_isenta","0%","","VAT 0%","40","0.0","percent","sale","tax_group_iva_0","","","100","base","invoice","+trp_base_8","",""
"","","","","","","","","","","","100","tax","invoice","","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_8","",""
"","","","","","","","","","","","100","tax","refund","","chart_2433",""
"iva_pt_purchase_normal","23%","","VAT 23%","70","23.0","percent","purchase","tax_group_iva_23","","","100","base","invoice","","",""
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_22","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_22","chart_2432",""
"iva_pt_purchase_intermedia","13%","","VAT 13%","80","13.0","percent","purchase","tax_group_iva_13","","","100","base","invoice","","",""
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_23","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_23","chart_2432",""
"iva_pt_purchase_reduzida","6%","","VAT 6%","90","6.0","percent","purchase","tax_group_iva_6","","","100","base","invoice","","",""
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_21","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_21","chart_2432",""
"iva_pt_purchase_isenta","0%","","VAT 0%","100","0.0","percent","purchase","tax_group_iva_0","","","100","base","invoice","","",""
"","","","","","","","","","","","100","tax","invoice","","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","","chart_2432",""
"iva_pt_sale_eu_isenta","0% EU","0% EU","VAT 0%","103","0.0","percent","sale","tax_group_iva_0","True","","100","base","invoice","+trp_base_7","","0% UE"
"","","","","","","","","","","","100","tax","invoice","","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_7","",""
"","","","","","","","","","","","100","tax","refund","","chart_2433",""
"iva_pt_sale_non_eu_isenta","0% EX","0% No EU","VAT 0%","105","0.0","percent","sale","tax_group_iva_0","True","","100","base","invoice","","","0% Não UE"
"","","","","","","","","","","","100","tax","invoice","","chart_2433",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","","chart_2433",""
"iva_pt_purchase_eu_isenta","0% EU","0% EU","VAT 0%","107","0.0","percent","purchase","tax_group_iva_0","","","100","base","invoice","","","0% UE"
"","","","","","","","","","","","100","tax","invoice","","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","","chart_2432",""
"iva_pt_purchase_non_eu_isenta","0% EX","0% No EU","VAT 0%","108","0.0","percent","purchase","tax_group_iva_0","","","100","base","invoice","","","0% Não UE"
"","","","","","","","","","","","100","tax","invoice","","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","","chart_2432",""
"iva_pt_purchase_eu_normal_bens","23% EU G","23% EU G","VAT 23%","110","23.0","percent","purchase","tax_group_iva_23","True","consu","100","base","invoice","+trp_base_12","","23% UE B"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_22","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_22","chart_2433",""
"iva_pt_purchase_eu_normal_servicos","23% EU S","23% EU S","VAT 23%","120","23.0","percent","purchase","tax_group_iva_23","True","service","100","base","invoice","+trp_base_16","","23% UE S"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_22","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_22","chart_2433",""
"iva_pt_purchase_eu_intermedia_bens","13% EU G","13% EU G","VAT 13%","130","13.0","percent","purchase","tax_group_iva_13","","consu","100","base","invoice","+trp_base_12","","13% UE B"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_23","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_23","chart_2433",""
"iva_pt_purchase_eu_intermedia_servicos","13% EU S","13% EU S","VAT 13%","140","13.0","percent","purchase","tax_group_iva_13","","service","100","base","invoice","+trp_base_16","","13% UE S"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_23","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_23","chart_2433",""
"iva_pt_purchase_eu_reduzida_bens","6% EU G","6% EU G","VAT 6%","150","6.0","percent","purchase","tax_group_iva_6","","consu","100","base","invoice","+trp_base_12","","6% UE B."
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_21","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_21","chart_2433",""
"iva_pt_purchase_eu_reduzida_servicos","6% EU S","6% EU S","VAT 6%","160","6.0","percent","purchase","tax_group_iva_6","","service","100","base","invoice","+trp_base_16","","6% UE S"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_21","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_21","chart_2433",""
"iva_pt_ma_sale_normal","22% MA","22% (MA)","VAT 22%","200","22.0","percent","sale","tax_group_iva_22","","","100","base","invoice","+trp_base_3","","22% (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_4","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_3","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_4","chart_2433",""
"iva_pt_ma_sale_intermedia","12% MA","12% (MA)","VAT 12%","210","12.0","percent","sale","tax_group_iva_12","","","100","base","invoice","+trp_base_5","","12% (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_6","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_5","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_6","chart_2433",""
"iva_pt_ma_sale_reduzida","5% MA","5% (MA)","VAT 5%","220","5.0","percent","sale","tax_group_iva_5","","","100","base","invoice","+trp_base_1","","5% (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_2","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_1","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_2","chart_2433",""
"iva_pt_ma_purchase_normal","22% MA","22% (MA)","VAT 22%","230","22.0","percent","purchase","tax_group_iva_22","","","100","base","invoice","","","22% (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_22","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_22","chart_2432",""
"iva_pt_ma_purchase_intermedia","12% MA","12% (MA)","VAT 12%","240","12.0","percent","purchase","tax_group_iva_12","","","100","base","invoice","","","12% (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_23","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_23","chart_2432",""
"iva_pt_ma_purchase_reduzida","5% MA","5% (MA)","VAT 5%","250","5.0","percent","purchase","tax_group_iva_5","","","100","base","invoice","","","5% (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_21","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_21","chart_2432",""
"iva_pt_ma_purchase_eu_normal_bens","22% EU G MA","22% EU G (MA)","VAT 22%","260","22.0","percent","purchase","tax_group_iva_22","","consu","100","base","invoice","+trp_base_12","","22% UE B (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_22","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_22","chart_2433",""
"iva_pt_ma_purchase_eu_normal_servicos","22% EU S MA","22% EU S (MA)","VAT 22%","270","22.0","percent","purchase","tax_group_iva_22","","service","100","base","invoice","+trp_base_16","","22% UE S (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_22","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_22","chart_2433",""
"iva_pt_ma_purchase_eu_intermedia_bens","12% EU G MA","12% EU G (MA)","VAT 12%","280","12.0","percent","purchase","tax_group_iva_12","","consu","100","base","invoice","+trp_base_12","","12% UE B (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_23","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_23","chart_2433",""
"iva_pt_ma_purchase_eu_intermedia_servicos","12% EU S MA","12% EU S (MA)","VAT 12%","290","12.0","percent","purchase","tax_group_iva_12","","service","100","base","invoice","+trp_base_16","","12% UE S (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_23","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_23","chart_2433",""
"iva_pt_ma_purchase_eu_reduzida_bens","5% EU G MA","5% EU G (MA)","VAT 5%","300","5.0","percent","purchase","tax_group_iva_5","","consu","100","base","invoice","+trp_base_12","","5% UE B (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_21","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_21","chart_2433",""
"iva_pt_ma_purchase_eu_reduzida_servicos","5% EU S MA","5% EU S (MA)","VAT 5%","310","5.0","percent","purchase","tax_group_iva_5","","service","100","base","invoice","+trp_base_16","","5% UE S (MA)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_21","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_21","chart_2433",""
"iva_pt_ac_sale_normal","16% AZ","16% (AZ)","VAT 16%","400","16.0","percent","sale","tax_group_iva_16","","","100","base","invoice","+trp_base_3","","16% (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_4","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_3","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_4","chart_2433",""
"iva_pt_ac_sale_intermedia","9% AZ","9% (AZ)","VAT 9%","410","9.0","percent","sale","tax_group_iva_9","","","100","base","invoice","+trp_base_5","","9% (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_6","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_5","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_6","chart_2433",""
"iva_pt_ac_sale_reduzida","4% AZ","4% (AZ)","VAT 4%","420","4.0","percent","sale","tax_group_iva_4","","","100","base","invoice","+trp_base_1","","4% (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_2","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_1","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_2","chart_2433",""
"iva_pt_ac_purchase_normal","16% AZ","16% (AZ)","VAT 16%","430","16.0","percent","purchase","tax_group_iva_16","","","100","base","invoice","","","16% (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_22","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_22","chart_2432",""
"iva_pt_ac_purchase_intermedia","9% AZ","9% (AZ)","VAT 9%","440","9.0","percent","purchase","tax_group_iva_9","","","100","base","invoice","","","9% (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_23","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_23","chart_2432",""
"iva_pt_ac_purchase_reduzida","4% AZ","4% (AZ)","VAT 4%","450","4.0","percent","purchase","tax_group_iva_4","","","100","base","invoice","","","4% (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_company_21","chart_2432",""
"","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_company_21","chart_2432",""
"iva_pt_ac_purchase_eu_normal_bens","16% EU G AZ","16% EU G (AZ)","VAT 16%","460","16.0","percent","purchase","tax_group_iva_16","","consu","100","base","invoice","+trp_base_12","","16% UE B (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_22","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_22","chart_2433",""
"iva_pt_ac_purchase_eu_normal_servicos","16% EU S AZ","16% EU S (AZ)","VAT 16%","470","16.0","percent","purchase","tax_group_iva_16","","service","100","base","invoice","+trp_base_16","","16% UE S. (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_22","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_22","chart_2433",""
"iva_pt_ac_purchase_eu_intermedia_bens","9% EU G AZ","9% EU G (AZ)","VAT 9%","480","9.0","percent","purchase","tax_group_iva_9","","consu","100","base","invoice","+trp_base_12","","9% UE B (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_23","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_23","chart_2433",""
"iva_pt_ac_purchase_eu_intermedia_servicos","9% EU S AZ","9% EU S (AZ)","VAT 9%","490","9.0","percent","purchase","tax_group_iva_9","","service","100","base","invoice","+trp_base_16","","9% UE S (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_23","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_23","chart_2433",""
"iva_pt_ac_purchase_eu_reduzida_bens","4% EU G AZ","4% EU G (AZ)","VAT 4%","500","4.0","percent","purchase","tax_group_iva_4","","consu","100","base","invoice","+trp_base_12","","4% UE B (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_21","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_12","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_13","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","+trp_tax_favor_company_21","chart_2433",""
"iva_pt_ac_purchase_eu_reduzida_servicos","4% EU S AZ","4% EU S (AZ)","VAT 4%","510","4.0","percent","purchase","tax_group_iva_4","","service","100","base","invoice","+trp_base_16","","4% UE S (AÇ)"
"","","","","","","","","","","","100","tax","invoice","+trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","invoice","-trp_tax_favor_company_21","chart_2433",""
"","","","","","","","","","","","100","base","refund","-trp_base_16","",""
"","","","","","","","","","","","100","tax","refund","-trp_tax_favor_state_17","chart_2432",""
"","","","","","","","","","","","-100","tax","refund","-trp_tax_favor_company_21","chart_2433",""

```

## File: data\template\account.tax.group-pt.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@pt"
"tax_group_iva_0","VAT 0%","base.pt","chart_2436","chart_2437","IVA 0%"
"tax_group_iva_4","VAT 4%","","chart_2436","chart_2437","IVA 4%"
"tax_group_iva_5","VAT 5%","","chart_2436","chart_2437","IVA 5%"
"tax_group_iva_6","VAT 6%","base.pt","chart_2436","chart_2437","IVA 6%"
"tax_group_iva_9","VAT 9%","","chart_2436","chart_2437","IVA 9%"
"tax_group_iva_12","VAT 12%","","chart_2436","chart_2437","IVA 12%"
"tax_group_iva_13","VAT 13%","base.pt","chart_2436","chart_2437","IVA 13%"
"tax_group_iva_16","VAT 16%","","chart_2436","chart_2437","IVA 16%"
"tax_group_iva_22","VAT 22%","","chart_2436","chart_2437","IVA 22%"
"tax_group_iva_23","VAT 23%","base.pt","chart_2436","chart_2437","IVA 23%"

```

## File: models\account_account.py

```python
from odoo import models, fields


class AccountAccount(models.Model):
    _inherit = "account.account"

    l10n_pt_taxonomy_code = fields.Integer(string="Taxonomy code")

```

## File: models\account_journal.py

```python
from odoo import models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    def _prepare_liquidity_account_vals(self, company, code, vals):
        account_vals = super()._prepare_liquidity_account_vals(company, code, vals)
        if company.account_fiscal_country_id.code == 'PT':
            if vals.get('type') == 'cash':
                account_vals['l10n_pt_taxonomy_code'] = 1
            elif vals.get('type') == 'bank':
                account_vals['l10n_pt_taxonomy_code'] = 2
        return account_vals

```

## File: models\template_pt.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('pt')
    def _get_pt_template_data(self):
        return {
            'property_account_receivable_id': 'chart_2111',
            'property_account_payable_id': 'chart_2211',
            'property_account_income_categ_id': 'chart_711',
            'property_account_expense_categ_id': 'chart_311',
        }

    @template('pt', 'res.company')
    def _get_pt_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.pt',
                'bank_account_code_prefix': '12',
                'cash_account_code_prefix': '11',
                'transfer_account_code_prefix': '1431',
                'account_default_pos_receivable_account_id': 'chart_2117',
                'income_currency_exchange_account_id': 'chart_7861',
                'expense_currency_exchange_account_id': 'chart_6863',
                'account_journal_early_pay_discount_loss_account_id': 'chart_682',
                'account_journal_early_pay_discount_gain_account_id': 'chart_728',
                'account_sale_tax_id': 'iva_pt_sale_normal',
                'account_purchase_tax_id': 'iva_pt_purchase_normal',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_pt
from . import account_account
from . import account_journal

```

