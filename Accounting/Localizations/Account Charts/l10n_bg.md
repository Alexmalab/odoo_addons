# Odoo Module: l10n_bg

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
    'name': 'Bulgaria - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['bg'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Chart accounting and taxes for Bulgaria
    """,
    'depends': [
        'account',
        'base_vat',
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
<odoo auto_sequence="1">
    <record id="l10n_bg_tax_report" model="account.report">
        <field name="name">Tax report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.bg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_bg_tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_bg_tax_report_a" model="account.report.line">
                <field name="name">Section A: Data on value added tax charged</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_bg_tax_report_01" model="account.report.line">
                        <field name="name">[01] Total amount of the tax bases for VAT taxation (amount from class 11 to class 16)</field>
                        <field name="code">BG_TR_01</field>
                        <field name="aggregation_formula">BG_TR_11.balance + BG_TR_12.balance + BG_TR_13.balance + BG_TR_14.balance + BG_TR_15.balance + BG_TR_16.balance</field>
                        <field name="children_ids">
                            <record id="l10n_bg_tax_report_a_1" model="account.report.line">
                                <field name="name">Tax base subject to taxation at a rate of 20%</field>
                                <field name="children_ids">
                                    <record id="l10n_bg_tax_report_11" model="account.report.line">
                                        <field name="name">[11] - tax base of taxable supplies, incl. deliveries under the conditions of distance sales with a place of performance on the territory of the country</field>
                                        <field name="code">BG_TR_11</field>
                                        <field name="expression_ids">
                                            <record id="l10n_bg_tax_report_11_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">11</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_bg_tax_report_12" model="account.report.line">
                                        <field name="name">[12] - tax base of VAT and tax base of received supplies under Article 82, paragraphs 2-6 of the VAT Act</field>
                                        <field name="code">BG_TR_12</field>
                                        <field name="aggregation_formula">BG_TR_12_1.balance + BG_TR_12_2.balance</field>
                                        <field name="children_ids">
                                            <record id="l10n_bg_tax_report_12_1" model="account.report.line">
                                                <field name="name">Intra-community acquisitions</field>
                                                <field name="code">BG_TR_12_1</field>
                                                <field name="expression_ids">
                                                    <record id="l10n_bg_tax_report_12_1_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">12_1</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="l10n_bg_tax_report_12_2" model="account.report.line">
                                                <field name="name">Deliveries under Art. 82, para. 2-6</field>
                                                <field name="code">BG_TR_12_2</field>
                                                <field name="expression_ids">
                                                    <record id="l10n_bg_tax_report_12_2_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">12_2</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bg_tax_report_13" model="account.report.line">
                                <field name="name">[13] Tax base of taxable supplies at a rate of 9%</field>
                                <field name="code">BG_TR_13</field>
                                <field name="expression_ids">
                                    <record id="l10n_bg_tax_report_13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">13</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bg_tax_report_a_2" model="account.report.line">
                                <field name="name">Tax base subject to 0% tax</field>
                                <field name="children_ids">
                                    <record id="l10n_bg_tax_report_14" model="account.report.line">
                                        <field name="name">[14] Tax base for supplies under Chapter Three of the VAT Act</field>
                                        <field name="code">BG_TR_14</field>
                                        <field name="expression_ids">
                                            <record id="l10n_bg_tax_report_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">14</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_bg_tax_report_15" model="account.report.line">
                                        <field name="name">[15] Tax base of AEO of goods</field>
                                        <field name="code">BG_TR_15</field>
                                        <field name="expression_ids">
                                            <record id="l10n_bg_tax_report_15_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">15</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="l10n_bg_tax_report_16" model="account.report.line">
                                        <field name="name">[16] Tax base of supplies under Articles 140, 146 and 173 of the VAT Act </field>
                                        <field name="code">BG_TR_16</field>
                                        <field name="expression_ids">
                                            <record id="l10n_bg_tax_report_16_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">16</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_17" model="account.report.line">
                        <field name="name">[17] Tax base for supplies of services under Article 21, paragraph 2 with a place of performance on the territory of another member state</field>
                        <field name="code">BG_TR_17</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">17</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_18" model="account.report.line">
                        <field name="name">[18] Tax base of supplies under Article 69, paragraph 2 of the VAT Act, incl. deliveries on the basis of distance selling with a place of performance in the territory of another Member State, as well as deliveries as an intermediary in a tripartite</field>
                        <field name="code">BG_TR_18</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">18</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_19" model="account.report.line">
                        <field name="name">[19] Tax base of exempt supplies and exempt VOP</field>
                        <field name="code">BG_TR_19</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_19_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">19</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_20" model="account.report.line">
                        <field name="name">[20] All VAT charged (amount from class 21 to class 24)</field>
                        <field name="code">BG_TR_20</field>
                        <field name="aggregation_formula">BG_TR_21.balance + BG_TR_22.balance + BG_TR_23.balance + BG_TR_24.balance</field>
                        <field name="children_ids">
                            <record id="l10n_bg_tax_report_21" model="account.report.line">
                                <field name="name">[21] VAT charged</field>
                                <field name="code">BG_TR_21</field>
                                <field name="expression_ids">
                                    <record id="l10n_bg_tax_report_21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">21</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bg_tax_report_22" model="account.report.line">
                                <field name="name">[22] VAT charged for VAT and for received deliveries under Art. 82, para 2-6</field>
                                <field name="code">BG_TR_22</field>
                                <field name="expression_ids">
                                    <record id="l10n_bg_tax_report_22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bg_tax_report_23" model="account.report.line">
                                <field name="name">[23] Tax charged on supplies of goods and services for personal use</field>
                                <field name="code">BG_TR_23</field>
                                <field name="expression_ids">
                                    <record id="l10n_bg_tax_report_23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">23</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bg_tax_report_24" model="account.report.line">
                                <field name="name">[24] VAT charged (9%)</field>
                                <field name="code">BG_TR_24</field>
                                <field name="expression_ids">
                                    <record id="l10n_bg_tax_report_24_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bg_tax_report_b" model="account.report.line">
                <field name="name">Section B: Data on the exercised right to a tax credit</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_bg_tax_report_30" model="account.report.line">
                        <field name="name">[30] Tax base and tax on the received deliveries, VAT, the received deliveries under art. 82, para. 2-6 of the VAT Act and imports without the right to a tax credit or without tax</field>
                        <field name="code">BG_TR_30</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_30_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">30</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_b_2" model="account.report.line">
                        <field name="name">Tax base of the received deliveries, VAT, the received deliveries under art. 82, para 2-6 of the VAT Act, the import, as well as the tax base of the received deliveries, used for making deliveries under art. 69, para 2 of the VAT Act</field>
                        <field name="children_ids">
                            <record id="l10n_bg_tax_report_31" model="account.report.line">
                                <field name="name">[31] - entitled to a full tax credit</field>
                                <field name="code">BG_TR_31</field>
                                <field name="expression_ids">
                                    <record id="l10n_bg_tax_report_31_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">31</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_bg_tax_report_32" model="account.report.line">
                                <field name="name">[32] - with the right to a partial tax credit</field>
                                <field name="code">BG_TR_32</field>
                                <field name="expression_ids">
                                    <record id="l10n_bg_tax_report_32_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">32</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_33" model="account.report.line">
                        <field name="name">[33] Coefficient under Article 73, paragraph 5 of the VAT Act</field>
                        <field name="code">BG_TR_33</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_33_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="figure_type" eval="False"/>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_41" model="account.report.line">
                        <field name="name">[41] VAT eligible for a full tax credit</field>
                        <field name="code">BG_TR_41</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_41_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">41</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_42" model="account.report.line">
                        <field name="name">[42] VAT with the right to a partial tax credit</field>
                        <field name="code">BG_TR_42</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_42_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_43" model="account.report.line">
                        <field name="name">[43] Annual adjustment under Article 73, paragraph 8 (+/-)</field>
                        <field name="code">BG_TR_43</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_43_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="figure_type">integer</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_40" model="account.report.line">
                        <field name="name">[40] Total tax credit (41 + 42 x class 33 + 43)</field>
                        <field name="code">BG_TR_40</field>
                        <field name="aggregation_formula">BG_TR_41.balance + (BG_TR_42.balance * BG_TR_33.balance) + BG_TR_43.balance</field>
                    </record>
                </field>
            </record>
            <record id="l10n_bg_tax_report_c" model="account.report.line">
                <field name="name">Section C: Result for the period</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_bg_tax_report_50" model="account.report.line">
                        <field name="name">[50] VAT to be paid (class 20 - class 40) >= 0</field>
                        <field name="code">BG_TR_50</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_50_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BG_TR_20.balance - BG_TR_40.balance</field>
                                <field name="subformula">if_above(BGN(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_60" model="account.report.line">
                        <field name="name">[60] VAT for refund (class 20 - class 40) &lt; 0</field>
                        <field name="code">BG_TR_60</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_60_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BG_TR_40.balance - BG_TR_20.balance</field>
                                <field name="subformula">if_above(BGN(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bg_tax_report_d" model="account.report.line">
                <field name="name">Section D. VAT for deposition</field>
                <field name="children_ids">
                    <record id="l10n_bg_tax_report_70" model="account.report.line">
                        <field name="name">[70] Tax for payment from Art. 50, deducted in accordance with Art. 92, para. 1 of the VAT Act</field>
                        <field name="code">BG_TR_70</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_70_refund" model="account.report.expression">
                                <field name="label">refund</field>
                                <field name="date_scope">from_beginning</field>
                                <field name="engine">account_codes</field>
                                <field name="formula">4531 + 4532 + 4534 + 4538 + 4539</field>
                            </record>
                            <record id="l10n_bg_tax_report_70_refund_remaining" model="account.report.expression">
                                <field name="label">refund_remaining</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BG_TR_70.refund</field>
                                <field name="subformula">if_below(BGN(0))</field>
                            </record>
                            <record id="l10n_bg_tax_report_70_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BG_TR_70.refund_remaining + BG_TR_50.balance</field>
                                <field name="subformula">if_above(BGN(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_71" model="account.report.line">
                        <field name="name">[71] Tax for payment from Art. 50, effectively paid</field>
                        <field name="code">BG_TR_71</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_71_refund" model="account.report.expression">
                                <field name="label">refund</field>
                                <field name="date_scope">from_beginning</field>
                                <field name="engine">account_codes</field>
                                <field name="formula">4531 + 4532 + 4534 + 4538 + 4539</field>
                            </record>
                            <record id="l10n_bg_tax_report_71_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BG_TR_50.balance - BG_TR_70.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bg_tax_report_e" model="account.report.line">
                <field name="name">Section E: Refundable VAT</field>
                <field name="children_ids">
                    <record id="l10n_bg_tax_report_80" model="account.report.line">
                        <field name="name">[80] According to Art. 92, para. 1 of the VAT Act within a 30-day period from the submission of this declaration</field>
                        <field name="code">BG_TR_80</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_80_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_81" model="account.report.line">
                        <field name="name">[81] According to Art. 92, para. 3 of the VAT Act within a 30-day period from the submission of this declaration</field>
                        <field name="code">BG_TR_81</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_81_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bg_tax_report_82" model="account.report.line">
                        <field name="name">[82] According to Art. 92, para. 4 of the VAT Act within a 30-day period from the submission of this declaration</field>
                        <field name="code">BG_TR_82</field>
                        <field name="expression_ids">
                            <record id="l10n_bg_tax_report_82_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-bg.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@bg"
"l10n_bg_101","Fixed capital required for registration","101","equity","","False","Основен капитал, изискващ регистрация"
"l10n_bg_102","Fixed capital that does not require registration","102","equity","","False","Основен капитал, неизискващ регистрация"
"l10n_bg_103","Liquidation capital in case of insolvency and liquidation","103","equity","","False","Ликвидационен капитал при несъстоятелност и ликвидация"
"l10n_bg_104","Capital of non-profit enterprises","104","equity","","False","Капитал на предприятия с нестопанска дейност"
"l10n_bg_105","Capital premiums","105","equity","","False","Премии, свързани с капитал"
"l10n_bg_106","Deductions (discount) related to capital","106","equity","","False","Отбиви (сконто), свързани с капитал"
"l10n_bg_111","General reserves","111","equity","","False","Общи резерви"
"l10n_bg_112","Reserves from subsequent valuation of fixed assets","112","equity","","False","Резерви от последваща оценка на дълготрайни активи"
"l10n_bg_113","Reserves from subsequent valuation of current assets","113","equity","","False","Резерви от последваща оценка на краткотрайни активи"
"l10n_bg_114","Reserves from subsequent valuation of financial instruments","114","equity","","False","Резерви от последваща оценка на финансови инструменти"
"l10n_bg_115","Reserves from share issue","115","equity","","False","Резерви от емисия на акции"
"l10n_bg_116","Reserve related to repurchased own shares","116","equity","","False","Резерв, свързан с изкупени собствени акции"
"l10n_bg_117","Reserve according to constitutive act","117","equity","","False","Резерв, свързан с изкупени собствени акции"
"l10n_bg_119","Other reserves","119","equity","","False","Други резерви"
"l10n_bg_121","Uncovered loss from previous years","121","equity","","False","Непокрита загуба от минали години"
"l10n_bg_122","Retained earnings from previous years","122","equity","","False","Неразпределена печалба от минали години"
"l10n_bg_123","Profit and loss from the current year","123","equity_unaffected","","False","Печалби и загуби от текущата година"
"l10n_bg_124","Result in bankruptcy and liquidation","124","equity","","False","Резултат при несъстоятелност и ликвидация"
"l10n_bg_125","Result of the activity of non-profit enterprises","125","equity","","False","Резултат от дейността на предприятия с нестопанска цел"
"l10n_bg_131","Life insurance reserves","131","equity","","False","Резерви за животозастраховане"
"l10n_bg_132","Transmission-premium reserves","132","equity","","False","Пренос-премийни резерви"
"l10n_bg_133","Reserves for forthcoming payments","133","equity","","False","Резерви на предстоящи плащания"
"l10n_bg_134","Reserves for reserve fund","134","equity","","False","Резерви за запасен фонд"
"l10n_bg_135","Reinsurance reserves","135","equity","","False","Пезерви за презастраховане"
"l10n_bg_136","Other reserves under insurance contracts","136","equity","","False","Други резерви по застрахователни договори"
"l10n_bg_137","Reserves of investment companies","137","equity","","False","Резерви на инвестиционни дружества"
"l10n_bg_138","Reserves of pension companies","138","equity","","False","Резерви на пенсионни дружества"
"l10n_bg_141","Banknotes for circulation","141","equity","","False","Банкноти за обращение"
"l10n_bg_142","Coins in circulation","142","equity","","False","Монети за обращение"
"l10n_bg_151","Received short-term loans","151","liability_current","","False","Получени краткосрочни заеми"
"l10n_bg_152","Long-term loans received","152","liability_current","","False","Получени дългосрочни заеми"
"l10n_bg_153","Debt instruments","153","liability_current","","False","Дългови инструменти"
"l10n_bg_159","Other loans and debts","159","liability_current","","False","Други заеми и дългове"
"l10n_bg_201","Lands (terrains)","201","asset_fixed","","False","Земи (терениà)"
"l10n_bg_202","Land improvements","202","asset_fixed","","False","Подобрения върху земите"
"l10n_bg_203","Buildings and structures","203","asset_fixed","","False","Сгради и конструкции"
"l10n_bg_204","Computer equipment","204","asset_fixed","","False","Компютърна техника"
"l10n_bg_205","Facilities","205","asset_fixed","","False","Съоръжения"
"l10n_bg_206","Machinery and equipment","206","asset_fixed","","False","Машини и оборудване"
"l10n_bg_207","Vehicles","207","asset_fixed","","False","Транспортни средства"
"l10n_bg_208","Office furniture","208","asset_fixed","","False","Офис обзавеждане"
"l10n_bg_209","Other tangible fixed assets","209","asset_fixed","","False","Други дълготрайни материални активи"
"l10n_bg_211","Development products","211","asset_fixed","","False","Продукти от развойна дейност"
"l10n_bg_212","Software products","212","asset_fixed","","False","Програмни продукти"
"l10n_bg_213","Intellectual property rights","213","asset_fixed","","False","Права върху интелектуална собственост"
"l10n_bg_214","Industrial property rights","214","asset_fixed","","False","Права върху индустриална собственост"
"l10n_bg_219","Other intangible fixed assets","219","asset_fixed","","False","Други дълготрайни нематериални активи"
"l10n_bg_221","Investments in subsidiaries","221","asset_non_current","","False","Инвестиции в дъщерни предприятия"
"l10n_bg_222","Investments in associates","222","asset_non_current","","False","Инвестиции в асоциирани предприятия"
"l10n_bg_223","Investments in joint ventures","223","asset_non_current","","False","Инвестиции в смесени предприятия"
"l10n_bg_224","Investment property","224","asset_non_current","","False","Инвестиционни имоти"
"l10n_bg_225","Financial assets held to maturity","225","asset_non_current","","False","Финансови активи, държани до настъпване на падеж"
"l10n_bg_226","Financial assets put up for sale","226","asset_non_current","","False","Финансови активи, обявени за продажба"
"l10n_bg_227","Financial assets pledged as compensation","227","asset_non_current","","False","Финансови активи, заложени като обезщетение"
"l10n_bg_228","Government securities","228","asset_non_current","","False","Държавни ценни книжа"
"l10n_bg_229","Other long-term financial assets","229","asset_non_current","","False","Други дългосрочни финансови активи"
"l10n_bg_231","Positive commercial reputation","231","asset_non_current","","False","Положителна търговска репутация"
"l10n_bg_232","Negative commercial reputation","232","asset_non_current","","False","Отрицателна търговска репутация"
"l10n_bg_241","Depreciation of tangible fixed assets","241","asset_fixed","","False","Амортизация на дълготрайни материални активи"
"l10n_bg_2413","Depreciation of buildings and structures","2413","asset_fixed","","False","Амортизация на сгради и конструкции"
"l10n_bg_2414","Depreciation of computer equipment","2414","asset_fixed","","False","Амортизация на компютърна техника"
"l10n_bg_2415","Depreciation of equipment","2415","asset_fixed","","False","Амортизация на сьоръжения"
"l10n_bg_2416","Depreciation of machinery and equipment","2416","asset_fixed","","False","Амортизация на машини и оборудване"
"l10n_bg_2417","Depreciation of vehicles","2417","asset_fixed","","False","Амортизация на транспортни средства"
"l10n_bg_2418","Depreciation of office furniture","2418","asset_fixed","","False","Амортизация на офис обзавеждане"
"l10n_bg_2419","Depreciation of other tangible assets","2419","asset_fixed","","False","Амортизация на други материални активи"
"l10n_bg_242","Amortization of intangible fixed assets","242","asset_fixed","","False","Амортизация на дълготрайни нематериални активи"
"l10n_bg_2421","Depreciation of development products","2421","asset_fixed","","False","Амортизация на продукти от развойна дейност"
"l10n_bg_2422","Depreciation of software products","2422","asset_fixed","","False","Амортизация на програмни продукти"
"l10n_bg_2423","Depreciation of intellectual property rights","2423","asset_fixed","","False","Амортизация на права върху интелектуална собственост"
"l10n_bg_2424","Depreciation of industrial property rights","2424","asset_fixed","","False","Амортизация на права върху индустриална собственост"
"l10n_bg_2429","Depreciation of other intangible fixed assets","2429","asset_fixed","","False","Амортизация на други дълготрайни нематериални активи"
"l10n_bg_261","Long-term receivables and loans to banks and other financial institutions","261","asset_non_current","","False","Дългосрочни вземания и предоставени заеми на банки и други финансови институции"
"l10n_bg_262","Long-term receivables and loans to non-financial corporations and other customers","262","asset_non_current","","False","Дългосрочни вземания и предоставени заеми на нефинансови предприятия и други клиенти"
"l10n_bg_263","Overdue long-term receivables and loans granted","263","asset_non_current","","False","Просрочени дългосрочни вземания и предоставени заеми"
"l10n_bg_264","Long-term receivables and loans granted as collateral","264","asset_non_current","","False","Дългосрочни вземания и предоставени заеми, заложени като обезпечение"
"l10n_bg_269","Other long-term receivables and loans","269","asset_non_current","","False","Други дългосрочни вземания и предоставени заеми"
"l10n_bg_271","Burning","271","asset_non_current","","False","Гори"
"l10n_bg_272","Permanent plantations - fruitful","272","asset_non_current","","False","Трайни насъждения-плододаващи"
"l10n_bg_273","Permanent plantations - infertile","273","asset_non_current","","False","Трайни насъждения-неплододаващи"
"l10n_bg_274","Animals in major herds","274","asset_non_current","","False","Животни в основни стада"
"l10n_bg_279","Other fixed biological assets","279","asset_non_current","","False","Други дълготрайни биологични активи"
"l10n_bg_301","Delivery","301","asset_current","","False","Доставки"
"l10n_bg_302","Materials","302","asset_current","","False","Материали"
"l10n_bg_303","Products","303","asset_current","","False","Продукция"
"l10n_bg_304","Goods","304","asset_current","","False","Стоки"
"l10n_bg_305","Work in progress","305","asset_current","","False","Незавършено производство"
"l10n_bg_311","Small productive animals","311","asset_current","","False","Дребни продуктивни животни"
"l10n_bg_312","Bird-main flocks","312","asset_current","","False","Пптици-основни стада"
"l10n_bg_313","Bee families","313","asset_current","","False","Пчелни семейства"
"l10n_bg_314","Young animals","314","asset_current","","False","Млади животни"
"l10n_bg_315","Animals for fattening","315","asset_current","","False","Животни за угояване"
"l10n_bg_316","Animals for experimental purposes","316","asset_current","","False","Животни за експериментални цели"
"l10n_bg_319","Other current biological assets","319","asset_current","","False","Други краткотрайни биологични активи"
"l10n_bg_401","Providers","401","liability_payable","","True","Доставчици"
"l10n_bg_402","Advance providers","402","liability_current","","False","Доставчици по аванси"
"l10n_bg_403","Trade credit providers","403","liability_current","","False","Доставчици по търговски кредити"
"l10n_bg_404","Suppliers of supplies under certain conditions","404","liability_current","","False","Доставчици по доставки при определени условия"
"l10n_bg_405","Suppliers of related party supplies","405","liability_current","","False","Доставчици по доставки със свързани лица"
"l10n_bg_406","Settlements with related parties for purchases","406","liability_current","","False","Разчети със свързани лица по покупки"
"l10n_bg_409","Other settlements with suppliers","409","liability_current","","False","Други разчети с доставчици"
"l10n_bg_411","Customers","411","asset_receivable","","True","Клиенти"
"l10n_bg_4111","Customers (PoS)","4111","asset_receivable","","True","Клиенти (PoS)"
"l10n_bg_412","Clients on advances","412","asset_current","","False","Клиенти по аванси"
"l10n_bg_413","Clients on trade credits","413","asset_current","","False","Клиенти по търговски кредити"
"l10n_bg_414","Sales customers under certain conditions","414","asset_current","","False","Клиенти по продажби при определени условия"
"l10n_bg_415","Sales customers with related parties","415","asset_current","","False","Клиенти по продажби със свързани лица"
"l10n_bg_416","Settlements with related parties for sales","416","asset_current","","False","Разчети със свързани лица по продажби"
"l10n_bg_419","Other settlements with clients","419","asset_current","","False","Други разчети с клиенти"
"l10n_bg_421","Staff","421","liability_payable","","True","Персонал"
"l10n_bg_422","Accountable persons","422","asset_current","","False","Подотчетни лица"
"l10n_bg_423","Liabilities for unused leave","423","liability_current","","False","Задължения по неизползвани отпуски"
"l10n_bg_424","Receivables from participations","424","asset_current","","False","Вземания от съучастия"
"l10n_bg_425","Obligations for participation","425","liability_current","","False","Задължения за съучастия"
"l10n_bg_426","Receivables from subscribed share contributions","426","asset_current","","False","Вземания по записани дялови вноски"
"l10n_bg_429","Other settlements with staff and partners","429","liability_current","","False","Други разчети с персонала и съдружниците"
"l10n_bg_430","Internal calculations","430","asset_current","","False","Вътрешни разчети"
"l10n_bg_431","Settlements by interbank operations","431","asset_current","","False","Разчети по междубанкови операции"
"l10n_bg_432","Internal settlements on interbank operations","432","asset_current","","False","Вътрешни разчети по междубанкови операции"
"l10n_bg_433","Internal settlements on intrabank operations","433","asset_current","","False","Вътрешни разчети по вътрешнобанкови операции"
"l10n_bg_439","Other internal estimates","439","asset_current","","False","Други вътрешни разчети"
"l10n_bg_441","Receivables on claims","441","asset_current","","False","Вземания по рекламации"
"l10n_bg_442","Receivables for losses and deductions","442","asset_current","","False","Вземания по липси и начети"
"l10n_bg_443","Price differences by shortages and readings","443","asset_current","","False","Ценови разлики по липси и начети"
"l10n_bg_444","Claims in litigation","444","asset_current","","False","Вземания по съдебни спорове"
"l10n_bg_445","Awarded receivables","445","asset_current","","False","Присъдени вземания"
"l10n_bg_451","Settlements with municipalities","451","liability_current","","False","Разчети с общините"
"l10n_bg_452","Profit tax estimates","452","liability_current","","False","Разчети за данък върху печалбата"
"l10n_bg_453","Estimates of value added tax","453","liability_current","","False","Разчети за данък върху добавената стойност"
"l10n_bg_4531","Vat purchases","4531","liability_current","","False","Ддс покупки"
"l10n_bg_4532","Vat sales","4532","liability_current","","False","Ддс продажби"
"l10n_bg_4534","Vat sales outside the country","4534","liability_current","","False","Ддс продажби извън страната"
"l10n_bg_4538","Vat recovery","4538","liability_current","","False","Ддс за възстановяване"
"l10n_bg_4539","Vat to pay","4539","liability_current","","False","Ддс за внасяне"
"l10n_bg_454","Estimates of personal income taxes","454","liability_current","","False","Разчети за данъци върху доходи на физически лица"
"l10n_bg_455","Settlements with ministries","455","liability_current","","False","Разчети с министерства"
"l10n_bg_456","Excise estimates","456","liability_current","","False","Разчети за акцизи"
"l10n_bg_457","Settlements with customs","457","liability_current","","False","Разчети с митници"
"l10n_bg_459","Other estimates with the budget and with departments","459","liability_current","","False","Други разчети с бюджета и с ведомства"
"l10n_bg_461","Settlements with the national social security institute","461","liability_current","","False","Разчети с националния осигурителен институт"
"l10n_bg_4611","Estimates for the social security fund","4611","liability_current","","False","Разчети за фонд доо"
"l10n_bg_4612","Estimates for smps fund","4612","liability_current","","False","Разчети за фонд дзпо"
"l10n_bg_4613","Estimates for the upf fund","4613","liability_current","","False","Разчети за фонд упф"
"l10n_bg_4614","Estimates for the gvrs fund","4614","liability_current","","False","Разчети за фонд гврс"
"l10n_bg_4615","Estimates for the tzpb fund","4615","liability_current","","False","Разчети за фонд тзпб"
"l10n_bg_462","Voluntary social security estimates","462","liability_current","","False","Разчети за доброволно социално осигуряване"
"l10n_bg_463","Health insurance estimates","463","liability_current","","False","Разчети за здравно осигуряване"
"l10n_bg_464","Estimates for one-time benefits and child allowances","464","liability_current","","False","Разчети за фонд дзпо"
"l10n_bg_469","Other settlements with insurers","469","liability_current","","False","Други разчети с осигурители"
"l10n_bg_471","Settlements with the international monetary fund","471","liability_current","","False","Разчети с международния жалутен фонд"
"l10n_bg_472","Settlements with the world bank","472","liability_current","","False","Разчети със световната банка"
"l10n_bg_473","Settlements with other international financial institutions","473","liability_current","","False","Разчети с други международни финансови институции"
"l10n_bg_474","Estimates under intergovernmental agreements","474","liability_current","","False","Разчети по междуправителствени спогодби"
"l10n_bg_481","Estimates of upcoming payments","481","liability_current","","False","Разчети по предсоящи плащания"
"l10n_bg_482","Reinsurance and co-insurance estimates","482","liability_current","","False","Разчети при презастраховане и съзастраховане"
"l10n_bg_483","Settlements with reinsurers","483","liability_current","","False","Разчети с презастрахователи"
"l10n_bg_484","Settlements with sedants","484","liability_current","","False","Разчети със седанти"
"l10n_bg_489","Other estimates","489","liability_current","","False","Други разчети"
"l10n_bg_491","Trustees","491","liability_non_current","","False","Доверители"
"l10n_bg_492","Guarantee estimates","492","liability_current","","False","Разчети за гаранции"
"l10n_bg_493","Settlements with owners","493","liability_current","","False","Разчети със собственици"
"l10n_bg_494","Insurance estimates","494","liability_current","","False",""
"l10n_bg_495","Interest calculations","495","liability_current","","False","Разчети по лихви"
"l10n_bg_496","Deferred tax estimates","496","liability_current","","False","Разчети по отсрочени данъци"
"l10n_bg_497","Provisions recognized as liabilities","497","liability_non_current","","False","Провизии, признати като пасиви"
"l10n_bg_4971","Provisions for pensions and other similar liabilities","4971","liability_non_current","","False","Провизии за пенсии и други подобни задължения"
"l10n_bg_4972","Other provisions and similar liabilities","4972","liability_non_current","","False","Други провизии и сходни задължения"
"l10n_bg_498","Other debtors","498","liability_payable","","True","Други дебитори"
"l10n_bg_499","Other creditors","499","asset_receivable","","True","Други кредитори"
"l10n_bg_501","Cash register in bgn","501","asset_cash","","False","Каса в левове"
"l10n_bg_502","Cash in foreign currency","502","asset_cash","","False","Каса във валута"
"l10n_bg_503","Current account in bgn","503","asset_cash","","False","Разплащателна сметка във левове"
"l10n_bg_504","Current account in foreign currency","504","asset_cash","","False","Разплащателна сметка във валута"
"l10n_bg_505","Letters of credit in bgn","505","asset_cash","","False","Акредитиви в левове"
"l10n_bg_506","Letters of credit in foreign currency","506","asset_cash","","False","Акредитиви във валута"
"l10n_bg_507","Deposits provided","507","asset_cash","","False","Предоставени депозити"
"l10n_bg_508","Payment checks","508","asset_cash","","False","Разплащателни чекове"
"l10n_bg_509","Other cash","509","asset_cash","","False","Други парични средства"
"l10n_bg_511","Financial assets held for trading","511","asset_current","","False","Финансови активи, държани за търгуване"
"l10n_bg_512","Repurchased own shares","512","asset_current","","False","Изкупени собствени акции"
"l10n_bg_513","Financial assets pledged as collateral","513","asset_current","","False","Финансови активи, заложени като обезпечение"
"l10n_bg_514","Repurchased own liabilities","514","asset_current","","False","Изкупени собствени облигации"
"l10n_bg_515","Government securities","515","asset_current","","False","Държавни ценни книжа"
"l10n_bg_516","Precious metals and precious stones","516","asset_current","","False","Благородни метали и акъпоценни камъни"
"l10n_bg_517","Shares and interests in group companies","517","asset_current","","False","Акции и дялове в предприятия от група"
"l10n_bg_518","Financing","518","asset_current","","False","Финансирания"
"l10n_bg_519","Other short-term financial assets","519","asset_current","","False","Други краткосрочни финансови активи"
"l10n_bg_521","Short-term receivables and loans from banks and other financial institutions","521","asset_current","","False","Краткосрочни вземания и заеми от банки и други финансови институции"
"l10n_bg_522","Short-term receivables and loans from non-financial corporations and other customers","522","asset_current","","False","Краткосрочни вземания и заеми от нефинансови предприятия и други клиенти"
"l10n_bg_523","Short-term receivables and loans pledged as collateral","523","asset_current","","False","Краткосрочни вземания и заеми заложени, като обезпечение"
"l10n_bg_524","Overdue short-term receivables and loans","524","asset_current","","False","Просрочени краткосрочни вземания и заеми"
"l10n_bg_529","Other current receivables and loans","529","asset_current","","False","Други краткосрочни вземания и заеми"
"l10n_bg_531","Prepaid expenses","531","asset_non_current","","False","Разходи за бъдещи периоди"
"l10n_bg_532","Income for future periods","532","liability_non_current","","False","Приходи за бъдещи периоди"
"l10n_bg_601","Material costs","601","expense","","False","Разходи за материали"
"l10n_bg_602","Costs for external services","602","expense","","False","Разходи за външни услуги"
"l10n_bg_603","Depreciation costs","603","expense_depreciation","","False","Разходи за амортизация"
"l10n_bg_604","Wage costs (remuneration)","604","expense","","False","Разходи за заплати (възнаграждения)"
"l10n_bg_6041","Compensation costs","6041","expense","","False","Разходи за възнаграждения"
"l10n_bg_6042","Expenses for additional remuneration","6042","expense","","False","Разходи за допълнителни възнаграждения"
"l10n_bg_6043","Paid leave expenses","6043","expense","","False","Разходи за платен отпуск"
"l10n_bg_6044","Hospital costs","6044","expense","","False","Разходи за болнични"
"l10n_bg_6045","Expenses for remuneration of self-insured persons","6045","expense","","False","Разходи за възнаграждения на самоосигуряващи се"
"l10n_bg_605","Insurance costs","605","expense","","False","Разходи за осигуровки"
"l10n_bg_6051","Expenditures for the social security fund","6051","expense","","False","Разходи за фонд доо"
"l10n_bg_6052","Expenditures for smps fund","6052","expense","","False","Разходи за фонд дзпо"
"l10n_bg_6053","Expenditures for the upf fund","6053","expense","","False","Разходи за фонд упф"
"l10n_bg_6054","Expenditures for the gvrs fund","6054","expense","","False","Разходи за фонд гврс"
"l10n_bg_6055","Expenditures for tzpb fund","6055","expense","","False","Разходи за фонд тзпб"
"l10n_bg_6056","Expenditures for the health fund","6056","expense","","False","Разходи за фонд зо"
"l10n_bg_606","Expenses for taxes, fees and other similar payments","606","expense","","False","Разходи за данъци, такси и други подобни плащания"
"l10n_bg_606451","Expenditures for settlements with municipalities","606451","expense","","False","Разходи за разчети с общините"
"l10n_bg_606452","Costs of income tax estimates","606452","expense","","False","Разходи за разчети за данък върху печалбата"
"l10n_bg_606454","Expenses for estimates of personal income taxes","606454","expense","","False","Разходи за разчети за данъци върху доходи на физически лица"
"l10n_bg_606459","Expenditures for other estimates with the budget and with departments","606459","expense","","False","Разходи за други разчети с бюджета и с ведомства"
"l10n_bg_607","Provision costs","607","expense","","False","Разходи за провизии"
"l10n_bg_608","Expenses from subsequent valuations of assets","608","expense","","False","Разходи от последващи оценки на активи"
"l10n_bg_609","Other expenses","609","expense","","False","Други разходи"
"l10n_bg_611","Operating expenses","611","expense","","False","Разходи за основна дейност"
"l10n_bg_612","Ancillary activity costs","612","expense","","False","Разходи за спомагателна дейност"
"l10n_bg_613","Expenses for acquisition of fixed assets","613","expense","","False","Разходи за придобиване на дълготрайни активи"
"l10n_bg_614","Administrative costs","614","expense","","False","Административни разходи"
"l10n_bg_615","Sales costs","615","expense","","False","Разходи за продажби"
"l10n_bg_616","Expenses for liquidation of tangible fixed assets","616","expense","","False","Разходи за ликвидация на дълготрайни материални активи"
"l10n_bg_617","Expenses on activities in non-profit enterprises","617","expense","","False","Разходи по дейности в предприятия с нестопанска дейност"
"l10n_bg_618","Liquidation and insolvency costs","618","expense","","False","Разходи при ликвидация и несъстоятелност"
"l10n_bg_621","Interest expenses","621","expense","","False","Разходи за лихви"
"l10n_bg_622","Provisioning costs for risky assets","622","expense","","False","Разходи за провизиране на рискови активи"
"l10n_bg_623","Expenses on operations with financial assets and instruments","623","expense","","False","Разходи по операции с финансови активи и инструменти"
"l10n_bg_624","Expenses on foreign exchange operations","624","expense","","False","Разходи по валутни операции"
"l10n_bg_625","Expenses from subsequent valuations of financial assets and instruments","625","expense","","False","Разходи от последващи оценки на финансови активи и инструменти"
"l10n_bg_629","Other financial expenses","629","expense","","False","Други финансови разходи"
"l10n_bg_631","Impairment loss (expense)","631","expense_depreciation","","False","Загуба (разходи) от обезценка"
"l10n_bg_632","Impairment losses on current assets","632","expense_depreciation","","False","Разходи от обезценка на текущи активи"
"l10n_bg_651","Non-financial expenses for future periods","651","expense","","False","Нефинансови разходи за бъдещи периоди"
"l10n_bg_652","Financial expenses for future periods","652","expense","","False","Финансови разходи за бъдещи периоди"
"l10n_bg_661","Expenses for insurance amounts","661","expense","","False","Разходи за суми по застраховането"
"l10n_bg_662","Costs for participation in the result","662","expense","","False","Разходи за участия в резултата"
"l10n_bg_663","Liquidation costs","663","expense","","False","Разходи за ликвидация"
"l10n_bg_664","Insurance commission expenses","664","expense","","False","Разходи за комисионни по застраховането"
"l10n_bg_665","Insurance reserve costs","665","expense","","False","Разходи за резерви по застраховането"
"l10n_bg_669","Other direct insurance costs","669","expense","","False","Други разходи по пряко застраховане"
"l10n_bg_671","Expenses for reinsurers' premiums","671","expense","","False","Разходи за отстъпени премии на презастрахователи"
"l10n_bg_672","Expenses for released reserves for passive reinsurance","672","expense","","False","Разходи за освободени резерви по пасивно презастраховане"
"l10n_bg_679","Other passive reinsurance costs","679","expense","","False","Други разходи по пасивно презастраховане"
"l10n_bg_681","Expenses for insurance benefits","681","expense","","False","Разходи за застрахователни обезщетения"
"l10n_bg_682","Costs for participation in the result","682","expense","","False","Разходи за участия в резултата"
"l10n_bg_683","Expenses for participation in the liquidation","683","expense","","False","Разходи за участие в ликвидацията"
"l10n_bg_684","Expenses for assigned reinsurance commissions and fees","684","expense","","False","Разходи за отстъпени презастраховтелни комисионни и такси"
"l10n_bg_685","Expenses for set aside reserves for active reinsurance","685","expense","","False","Разходи за заделени резерви по активно презастраховане"
"l10n_bg_689","Other active reinsurance costs","689","expense","","False","Други разходи по активно презастраховане"
"l10n_bg_691","Exceptional costs","691","expense","","False","Извънредни разходи"
"l10n_bg_701","Revenues from sales of products","701","income","","False","Приходи от продажба на продукция"
"l10n_bg_702","Revenues from sales of goods","702","income","","False","Приходи от продажби на материали"
"l10n_bg_703","Revenues from sales of services","703","income","","False","Приходи от продажби на услуги"
"l10n_bg_704","Revenues from financing","704","income","","False","Приходи от финансирание"
"l10n_bg_705","Income from sales of fixed assets","705","income","","False","Приходи от продажби на дълготрайни активи"
"l10n_bg_706","Revenues from sales of materials","706","income","","False",""
"l10n_bg_707","Proceeds from liquidation and insolvency","707","income","","False","Приходи от ликвидация и несъстоятелност"
"l10n_bg_709","Other operating income","709","income","","False","Други приходи от дейността"
"l10n_bg_711","Revenues from regulated activities","711","income_other","","False","Приходи от регламентирана дейност"
"l10n_bg_712","Revenues from membership fees","712","income_other","","False","Приходи от членски внос"
"l10n_bg_713","Revenues from program financing","713","income_other","","False","Приходи от финансирания на програми"
"l10n_bg_714","Revenues from government donations","714","income_other","","False","Приходи от правителствени дарения"
"l10n_bg_715","Revenues from other donations","715","income_other","","False","Приходи от други дарения"
"l10n_bg_719","Other incomes","719","income_other","","False","Други приходи"
"l10n_bg_721","Interest income","721","income_other","","False","Приходи от лихви"
"l10n_bg_722","Income from participations","722","income_other","","False","Приходи от съучастия"
"l10n_bg_723","Income from operations with financial assets and instruments","723","income_other","","False","Приходи от комисиони от презастрахователи"
"l10n_bg_724","Income from foreign exchange operations","724","income_other","","False","Приходи от валутни операции"
"l10n_bg_725","Income from subsequent valuations of financial assets and instruments","725","income_other","","False","Приходи от последващи оценки на финансови активи и инструменти"
"l10n_bg_726","Revenues from reintegrated provisions","726","income_other","","False","Приходи от реинтегрирани провизии"
"l10n_bg_729","Other financial income","729","income_other","","False","Други финансови приходи"
"l10n_bg_731","Income from recovered impairment losses","731","income_other","","False","Приходи от възстановени загуби от обезценка"
"l10n_bg_751","Non-financial income for future periods","751","income_other","","False","Нефинансови приходи за бъдещи периоди"
"l10n_bg_752","Financial income for future periods","752","income_other","","False","Финансови приходи за бъдещи периоди"
"l10n_bg_753","Financing for fixed assets","753","income_other","","False","Финансиране за дълготрайни активи"
"l10n_bg_754","Financing the current activity","754","income_other","","False","Финансиране на текущата дейност"
"l10n_bg_761","Income from insurance premiums","761","income_other","","False","Приходи от премии по застраховането"
"l10n_bg_762","Revenues from commissions and fees","762","income_other","","False","Приходи от комисионни и такси"
"l10n_bg_763","Insurance income from previous years","763","income_other","","False","Приходи по застраховане от минали години"
"l10n_bg_764","Revenue from recourses","764","income_other","","False","Приходи от регреси"
"l10n_bg_765","Revenues from released reserves","765","income_other","","False","Приходи от освободени резерви"
"l10n_bg_769","Other insurance income","769","income_other","","False","Други застрахователни приходи"
"l10n_bg_771","Income from indemnities received from reinsurers","771","income_other","","False","Приходи от получени обезщетения от презастрахователи"
"l10n_bg_772","Income from participation in the result of reinsurers","772","income_other","","False","Приходи от участие в резултата на презастрахователи"
"l10n_bg_773","Revenues from commissions from reinsurers","773","income_other","","False","Приходи от комисиони от презастрахователи"
"l10n_bg_774","Passive reinsurance income from previous years","774","income_other","","False","Приходи от пасивно презастраховане от минали години"
"l10n_bg_775","Income from reinsurance reserves","775","income_other","","False","Приходи от резерви за презастраховане"
"l10n_bg_779","Other income from passive reinsurance","779","income_other","","False","Други приходи по пасивно презастраховане"
"l10n_bg_781","Reinsurance premium income","781","income_other","","False","Приходи от премии по презастраховане"
"l10n_bg_782","Income from active reinsurance from previous years","782","income_other","","False","Приходи от активно презастраховане от минали години"
"l10n_bg_783","Income from active reinsurance recourses","783","income_other","","False","Приходи от регреси по активно презастраховане"
"l10n_bg_784","Income from active reinsurance reserves","784","income_other","","False","Приходи от резерви по активно презастраховане"
"l10n_bg_789","Other income from active reinsurance","789","income_other","","False","Други приходи от активно презастраховане"
"l10n_bg_791","Extraordinary income","791","income_other","","False","Извънредни приходи"
"l10n_bg_911","Foreign tangible and intangible assets provided as collateral","911","off_balance","","False","Чужди материални и нематериални активи, предоставени като обезпечение"
"l10n_bg_912","Leased foreign assets","912","off_balance","","False","Наети чужди активи"
"l10n_bg_913","Foreign tangible assets received under a consignment agreement","913","off_balance","","False","Чужди материални активи, получени по консигнационен договор"
"l10n_bg_914","Documents under special reporting","914","off_balance","","False","Документи под особена отчетност"
"l10n_bg_915","Non-financial assets accepted for safekeeping","915","off_balance","","False","Нефинансови ценности, приети за съхранение"
"l10n_bg_921","Foreign financial assets provided as collateral","921","off_balance","","False","Чужди финансови активи, предоставени като обезпечение"
"l10n_bg_922","Pledged policies","922","off_balance","","False","Заложени полици"
"l10n_bg_923","Miscellaneous emissions for circulation","923","off_balance","","False","Разни емисии за пускане в обращение"
"l10n_bg_924","Foreign financial assets held on behalf of customers","924","off_balance","","False","Чужди финансови активи, държани от името на клиенти"
"l10n_bg_925","Commitments under contracts","925","off_balance","","False","Поети задължения по договори"
"l10n_bg_931","Receivables from loans granted under interstate agreements","931","off_balance","","False","Вземания по предоставени кредити по междудържавни спогодби"
"l10n_bg_932","Outstanding receivables written off","932","off_balance","","False","Отписани непогасени вземания"
"l10n_bg_933","Debtors by collection operations","933","off_balance","","False","Дебитори по операции за инкасиране"
"l10n_bg_934","Receivables from derivative transactions","934","off_balance","","False","Вземания по дериватни сделки"
"l10n_bg_941","Receivables from spot transactions","941","off_balance","","False","Вземания по спот операции"
"l10n_bg_942","Write-off of outstanding receivables in banks","942","off_balance","","False","Отписани непогасени вземания в банките"
"l10n_bg_943","Debtors on collection operations in banks","943","off_balance","","False","Дебитори по операции за инкасиране в банките"
"l10n_bg_945","Contingent receivables from other banking operations","945","off_balance","","False","Условни вземания по други банкови операции"
"l10n_bg_951","Guarantees and other similar contingent liabilities","951","off_balance","","False","Гаранции и други подобни непредвидени задължения"
"l10n_bg_952","Irrevocable commitments","952","off_balance","","False","Неотменяеми ангажименти"
"l10n_bg_953","Cancellable commitments","953","off_balance","","False","Задължения по дериватни сделки"
"l10n_bg_954","Liabilities under spot operations","954","off_balance","","False","Задължения по спот операции"
"l10n_bg_955","Liabilities under derivative transactions","955","off_balance","","False","Задължения по дериватни сделки"
"l10n_bg_956","Contingent liabilities on other banking operations","956","off_balance","","False","Условни задължения по други банкови операции"
"l10n_bg_961","Reserve fund","961","off_balance","","False","Резервна каса"
"l10n_bg_962","Assets in use, reported","962","off_balance","","False","Активи в употреба, отчете"
"l10n_bg_971","Issue reserve","971","off_balance","","False","Емисионен резерв"
"l10n_bg_972","Own issues out of circulation with expired term for exchange","972","off_balance","","False","Собствени емисии извън обращение с изтекъл срок за обмяна"
"l10n_bg_981","Reserve account","981","off_balance","","False","Сметка по резервите"
"l10n_bg_982","Accounts receivable","982","off_balance","","False","Сметки по изискуеми обезпечения"
"l10n_bg_983","Commitments on authorized loans","983","off_balance","","False","Ангажименти по резрешени кредити"
"l10n_bg_984","Various statistical accounts","984","off_balance","","False","Разни статистически сметки"
"l10n_bg_989","Other contingent asset accounts","985","off_balance","","False","Други сметки за условни активи"
"l10n_bg_991","Other contingent liability accounts","991","off_balance","","False","Други сметки за условни пасиви"
"l10n_bg_791001","Cash Difference Gain","791001","income_other","","False","Печалба от парична разлика"
"l10n_bg_691001","Cash Difference Loss","691001","expense","","False","Загуба на парична разлика"

```

## File: data\template\account.fiscal.position-bg.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@bg"
"fiscal_position_template_dom","10","Domestic","1","1","base.bg","","","","Домашен"
"fiscal_position_bg_private_eu","20","EU B2C","1","","","base.europe","","","Частен ЕС"
"fiscal_position_template_in_eu","30","EU B2B","1","1","","base.europe","l10n_bg_sale_vat_20","l10n_bg_sale_vat_0_icd","Вътре в ЕС"
"","","","","","","","l10n_bg_sale_vat_20_personal","l10n_bg_sale_vat_0_icd",""
"","","","","","","","l10n_bg_sale_vat_9","l10n_bg_sale_vat_0_icd",""
"","","","","","","","l10n_bg_sale_vat_9_personal","l10n_bg_sale_vat_0_icd",""
"","","","","","","","l10n_bg_sale_vat_0_exempt","l10n_bg_sale_vat_0_icd",""
"","","","","","","","l10n_bg_purchase_vat_20_otc","l10n_bg_purchase_vat_0_otc_ica",""
"","","","","","","","l10n_bg_purchase_vat_20_ftc","l10n_bg_purchase_vat_20_ftc_ica",""
"","","","","","","","l10n_bg_purchase_vat_20_ptc","l10n_bg_purchase_vat_20_ptc_ica",""
"fiscal_position_template_out_eu","40","Outside EU","1","","","","l10n_bg_sale_vat_20","l10n_bg_sale_vat_0_export","Извън ЕС"
"","","","","","","","l10n_bg_sale_vat_20_personal","l10n_bg_sale_vat_0_export",""
"","","","","","","","l10n_bg_sale_vat_9","l10n_bg_sale_vat_0_export",""
"","","","","","","","l10n_bg_sale_vat_0_exempt","l10n_bg_sale_vat_0_export",""
"","","","","","","","l10n_bg_purchase_vat_20_otc","l10n_bg_purchase_vat_0_otc_exempt",""
"","","","","","","","l10n_bg_purchase_vat_20_ftc","l10n_bg_purchase_vat_20_ftc_exempt",""
"","","","","","","","l10n_bg_purchase_vat_20_ptc","l10n_bg_purchase_vat_20_ptc_exempt",""

```

## File: data\template\account.tax-bg.csv

```csv
"id","sequence","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@bg","active"
"l10n_bg_sale_vat_20","101","20%","20% VAT","20% VAT","20.0","percent","sale","tax_group_vat_20","100","base","invoice","+11","","20% ДДС",""
"","","","","","","","","","100","tax","invoice","+21","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-11","","",""
"","","","","","","","","","100","tax","refund","-21","l10n_bg_4532","",""
"l10n_bg_sale_vat_20_remote","102","20% R","20% Remote","20% Remote","20.0","percent","sale","tax_group_vat_20","100","base","invoice","+11||+18","","20% дист.прод.","False"
"","","","","","","","","","100","tax","invoice","+21","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-11||-18","","",""
"","","","","","","","","","100","tax","refund","-21","l10n_bg_4532","",""
"l10n_bg_sale_vat_20_personal","103","20% Pers","20% Personal use","20% Personal use","20.0","percent","sale","tax_group_vat_20","100","base","invoice","+11","","20% лични","False"
"","","","","","","","","","100","tax","invoice","+23","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-11","","",""
"","","","","","","","","","100","tax","refund","-23","l10n_bg_4532","",""
"l10n_bg_sale_vat_9","111","9%","9% VAT","9% VAT","9.0","percent","sale","tax_group_vat_9","100","base","invoice","+13","","9% ДДС",""
"","","","","","","","","","100","tax","invoice","+24","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-13","","",""
"","","","","","","","","","100","tax","refund","-24","l10n_bg_4532","",""
"l10n_bg_sale_vat_9_personal","112","9% Pers","9% Personal use","9% Personal use","9.0","percent","sale","tax_group_vat_9","100","base","invoice","+13","","9% лични","False"
"","","","","","","","","","100","tax","invoice","+23","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-13","","",""
"","","","","","","","","","100","tax","refund","-23","l10n_bg_4532","",""
"l10n_bg_sale_vat_0_export","121","0% EX","0% Export","0% Export","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+14","","0% Експорт",""
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-14","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_sale_vat_0_icd","122","0% EU","0% Intra-Community Deliveries","0% Intra-Community Deliveries","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+15","","0% ВОД",""
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-15","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_sale_vat_0_140","123","0% 140","0% Art. 140, 146, 173","0% Art. 140, 146, 173","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+16","","0% чл. 140, 146, 173","False"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-16","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_sale_vat_0_21","124","0% 21","0% Art. 21","0% Art. 21","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+17","","0% чл. 21","False"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-17","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_sale_vat_0_exempt","125","0% EXEMPT","0% Exempt","0% Exempt","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+19","","0% осв.",""
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-19","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_sale_vat_0_tri","126","0% Tri","0% Tripartite","0% Tripartite","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+18","","0% тристр.","False"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-18","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_purchase_vat_20_ftc","201","20% FTC","20% Foreign Tax Credit","20% Foreign Tax Credit","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+31","","20% ПДК",""
"","","","","","","","","","100","tax","invoice","+41","l10n_bg_4531","",""
"","","","","","","","","","100","base","refund","-31","","",""
"","","","","","","","","","100","tax","refund","-41","l10n_bg_4531","",""
"l10n_bg_purchase_vat_20_ptc","202","20% PTC","20% Payable Tax Credit","20% Payable Tax Credit","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+32","","20% ЧДК",""
"","","","","","","","","","100","tax","invoice","+42","l10n_bg_4531","",""
"","","","","","","","","","100","base","refund","-32","","",""
"","","","","","","","","","100","tax","refund","-42","l10n_bg_4531","",""
"l10n_bg_purchase_vat_20_otc","203","20% OTC","20% Other Tax Credit","20% Other Tax Credit","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+30","","20% бДК",""
"","","","","","","","","","100","tax","invoice","+30","l10n_bg_4531","",""
"","","","","","","","","","100","base","refund","-30","","",""
"","","","","","","","","","100","tax","refund","-30","l10n_bg_4531","",""
"l10n_bg_purchase_vat_20_ftc_exempt","204","20% FTC EXEMPT","20% Foreign Tax Credit – Exempt","20% Foreign Tax Credit – Exempt","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+19||+31","","",""
"","","","","","","","","","100","tax","invoice","+41","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","invoice","-22","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-19||-31","","",""
"","","","","","","","","","100","tax","refund","-41","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","refund","+22","l10n_bg_4532","",""
"l10n_bg_purchase_vat_20_ftc_ica","204","20% EU FTC","20% Foreign Tax Credit – Intra-Community Acquisition","20% Foreign Tax Credit – Intra-Community Acquisition","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+12_1||+31","","20% ПДК (ВОП)",""
"","","","","","","","","","100","tax","invoice","+41","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","invoice","-22","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-12_1||-31","","",""
"","","","","","","","","","100","tax","refund","-41","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","refund","+22","l10n_bg_4532","",""
"l10n_bg_purchase_vat_20_ftc_82","205","20% FTC 82","20% Foreign Tax Credit – Art. 82","20% Foreign Tax Credit – Art. 82","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+12_2||+31","","20% ПДК (чл.82)","False"
"","","","","","","","","","100","tax","invoice","+41","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","invoice","-22","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-12_2||-31","","",""
"","","","","","","","","","100","tax","refund","-41","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","refund","+22","l10n_bg_4532","",""
"l10n_bg_purchase_vat_20_ptc_exempt","206","20% PTC EXEMPT","20% Payable Tax Credit – Exempt","20% Payable Tax Credit – Exempt","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+19||+32","","",""
"","","","","","","","","","100","tax","invoice","+42","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","invoice","-22","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-19||-32","","",""
"","","","","","","","","","100","tax","refund","-42","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","refund","+22","l10n_bg_4532","",""
"l10n_bg_purchase_vat_20_ptc_ica","206","20% EU PTC","20% Payable Tax Credit – Intra-Community Acquisition","20% Payable Tax Credit – Intra-Community Acquisition","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+12_1||+32","","",""
"","","","","","","","","","100","tax","invoice","+42","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","invoice","-22","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-12_1||-32","","",""
"","","","","","","","","","100","tax","refund","-42","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","refund","+22","l10n_bg_4532","",""
"l10n_bg_purchase_vat_20_ptc_82","207","20% PTC 82","20% Payable Tax Credit - Art. 82","20% Payable Tax Credit - Art. 82","20.0","percent","purchase","tax_group_vat_20","100","base","invoice","+12_2||+32","","20% ЧДК (чл.82)","False"
"","","","","","","","","","100","tax","invoice","+42","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","invoice","-22","l10n_bg_4532","",""
"","","","","","","","","","100","base","refund","-12_2||-32","","",""
"","","","","","","","","","100","tax","refund","-42","l10n_bg_4531","",""
"","","","","","","","","","-100","tax","refund","+22","l10n_bg_4532","",""
"l10n_bg_purchase_vat_9_ftc","211","9% FTC","9% Foreign Tax Credit","9% Foreign Tax Credit","9.0","percent","purchase","tax_group_vat_9","100","base","invoice","+31","","9% ПДК",""
"","","","","","","","","","100","tax","invoice","+41","l10n_bg_4531","",""
"","","","","","","","","","100","base","refund","-31","","",""
"","","","","","","","","","100","tax","refund","-41","l10n_bg_4531","",""
"l10n_bg_purchase_vat_9_otc","212","9% OTC","9% Other Tax Credit","9% Other Tax Credit","9.0","percent","purchase","tax_group_vat_9","100","base","invoice","+30","","9% бДК",""
"","","","","","","","","","100","tax","invoice","+30","l10n_bg_4531","",""
"","","","","","","","","","100","base","refund","-30","","",""
"","","","","","","","","","100","tax","refund","-30","l10n_bg_4531","",""
"l10n_bg_purchase_vat_0_otc_ica","231","0% EU OTC","0% Other Tax Credit – Intra-Community Acquisition","0% Other Tax Credit – Intra-Community Acquisition","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","+19||+30","","0% бДК (ВОП)",""
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-19||-30","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_purchase_vat_0_otc_exempt","232","0% OTC EXEMPT","0% Other Tax Credit – Exempt","0% Other Tax Credit – Exempt","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","+30","","0% бДК (осв.)",""
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-30","","",""
"","","","","","","","","","100","tax","refund","","","",""
"l10n_bg_purchase_vat_0_tri","233","0% Tri","0% Tripartite","0% Tripartite","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","+18","","0% тристр.","False"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","-18","","",""
"","","","","","","","","","100","tax","refund","","","",""

```

## File: data\template\account.tax.group-bg.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@bg"
"tax_group_vat_20","20% VAT","base.bg","l10n_bg_4539","l10n_bg_4538","20% ДДС"
"tax_group_vat_9","9% VAT","base.bg","l10n_bg_4539","l10n_bg_4538","9% ДДС"
"tax_group_vat_0","0% VAT","base.bg","l10n_bg_4539","l10n_bg_4538","0% ДДС"

```

## File: models\template_bg.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bg')
    def _get_bg_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_bg_411',
            'property_account_payable_id': 'l10n_bg_401',
            'property_account_expense_categ_id': 'l10n_bg_601',
            'property_account_income_categ_id': 'l10n_bg_701',
            'default_cash_difference_income_account_id': 'l10n_bg_791001',
            'default_cash_difference_expense_account_id': 'l10n_bg_691001',
            'code_digits': '6',
        }

    @template('bg', 'res.company')
    def _get_bg_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.bg',
                'bank_account_code_prefix': '503',
                'cash_account_code_prefix': '501',
                'transfer_account_code_prefix': '430',
                'income_currency_exchange_account_id': 'l10n_bg_624',
                'expense_currency_exchange_account_id': 'l10n_bg_624',
                'account_sale_tax_id': 'l10n_bg_sale_vat_20',
                'account_purchase_tax_id': 'l10n_bg_purchase_vat_20_ptc',
                'account_default_pos_receivable_account_id': 'l10n_bg_4111',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_bg

```

