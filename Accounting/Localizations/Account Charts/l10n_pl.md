# Odoo Module: l10n_pl

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

def _preserve_tag_on_taxes(env):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(env, 'l10n_pl')

def post_init_hook(env):
    _preserve_tag_on_taxes(env)

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Poland - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['pl'],
    'version': '2.0',
    'author': 'Odoo S.A., Grzegorz Grzelak (OpenGLOBE) (http://www.openglobe.pl)',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the accounting chart and taxes for Poland in Odoo.
==================================================================================

To jest moduł do tworzenia wzorcowego planu kont, podatków, obszarów podatkowych i
rejestrów podatkowych. Moduł ustawia też konta do kupna i sprzedaży towarów
zakładając, że wszystkie towary są w obrocie hurtowym.

Niniejszy moduł jest przeznaczony dla odoo 8.0.
Wewnętrzny numer wersji OpenGLOBE 1.02
    """,
    'depends': [
        'base_iban',
        'base_vat',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'data/l10n_pl.l10n_pl_tax_office.csv',
        'data/res.country.state.csv',
        'data/account.account.tag.csv',
        'data/account_tax_report_data.xml',
        'views/account_move_views.xml',
        'views/product_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_preserve_tag_on_taxes',
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
"id","name","applicability","country_id/id"
"gold_tag","Gold","taxes","base.pl"

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.pl"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_razem_c" model="account.report.line">
                <field name="name">Base - Total C</field>
                <field name="aggregation_formula">PLTAXC_01_10.balance + PLTAXC_02_11.balance + PLTAXC_03_13.balance + PLTAXC_04_15.balance + PLTAXC_05_17.balance + PLTAXC_06_19.balance + PLTAXC_07_21.balance + PLTAXC_08_22.balance + PLTAXC_09_23.balance + PLTAXC_10_25.balance + PLTAXC_11_27.balance + PLTAXC_12_31.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_kraj_zwolnione" model="account.report.line">
                        <field name="name">Base - Supply of goods/services, domestic, exempt</field>
                        <field name="code">PLTAXC_01_10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_zwolnione_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_poza_kraj" model="account.report.line">
                        <field name="name">Base - Supply of goods/services, out of the country</field>
                        <field name="code">PLTAXC_02_11</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_poza_kraj_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_11</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_uslugi_art_100_1_4" model="account.report.line">
                                <field name="name">Base - Services included in art. 100.1.4</field>
                                <field name="code">PLTAXC_02a_12</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_uslugi_art_100_1_4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_12</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_triangular_buyer_2nd_payer" model="account.report.line">
                                <field name="name">Triangular transaction 2. VAT payer</field>
                                <field name="code">PLTAXC_02a_12_Triangular</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_triangular_buyer_2nd_payer_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Triangular Purchase</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_uslugi_kraj_0" model="account.report.line">
                        <field name="name">Base - Supply of goods/services, domestic, 0%</field>
                        <field name="code">PLTAXC_03_13</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_uslugi_kraj_0_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_13</field>
                            </record>
                            <record id="account_tax_report_line_uslugi_kraj_0_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PLTAXC_03_13.tag + PLTAXC_03_14.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_towary_art_129" model="account.report.line">
                                <field name="name">Base - Goods under art. 129</field>
                                <field name="code">PLTAXC_03_14</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_towary_art_129_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_14</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_kraj_5" model="account.report.line">
                        <field name="name">Base - Supply of goods/services, domestic, 5%</field>
                        <field name="code">PLTAXC_04_15</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_15</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_kraj_8" model="account.report.line">
                        <field name="name">Base - Supply of goods/services, domestic, 8%</field>
                        <field name="code">PLTAXC_05_17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_17</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_kraj_23" model="account.report.line">
                        <field name="name">Base - Supply of goods/services, domestic, 23%</field>
                        <field name="code">PLTAXC_06_19</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_19</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_dostawa_towarow" model="account.report.line">
                        <field name="name">Base - Intra-Community supply of goods</field>
                        <field name="code">PLTAXC_07_21</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_dostawa_towarow_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_21</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_intracom_procedure_i_42" model="account.report.line">
                                <field name="name">- under customs procedure 42</field>
                                <field name="code">PLTAXC_07_21_I42</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_intracom_procedure_i_42_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I_42</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_intracom_procedure_i_63" model="account.report.line">
                                <field name="name">- under customs procedure 63</field>
                                <field name="code">PLTAXC_07_21_I63</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_intracom_procedure_i_63_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I_63</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_eksport_towarow" model="account.report.line">
                        <field name="name">Base - Export of goods</field>
                        <field name="code">PLTAXC_08_22</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_eksport_towarow_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_22</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_nabycie_towarow" model="account.report.line">
                        <field name="name">Base - Intra-Community acquisition of goods</field>
                        <field name="code">PLTAXC_09_23</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_nabycie_towarow_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_23</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_triangular_2nd_payer" model="account.report.line">
                                <field name="name">Triangular transaction 2. VAT payer</field>
                                <field name="code">PLTAXC_09_23_Triangular</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_triangular_2nd_payer_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Triangular Sale</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_art_33a" model="account.report.line">
                        <field name="name">Base - Import of goods under art. 33a</field>
                        <field name="code">PLTAXC_10_25</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_art_33a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_25</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_import_uslug" model="account.report.line">
                        <field name="name">Base - Importation of services</field>
                        <field name="code">PLTAXC_11_27</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_import_uslug_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_27</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_art_28b" model="account.report.line">
                                <field name="name">Base - Acquisition under art. 28b</field>
                                <field name="code">PLTAXC_11a_29</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_art_28b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_29</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_podatnik_nabywca" model="account.report.line">
                        <field name="name">Base - Supply of goods, taxable person acquiring</field>
                        <field name="code">PLTAXC_12_31</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_podatnik_nabywca_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_31</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_razem_d" model="account.report.line">
                <field name="name">Base - Total D</field>
                <field name="aggregation_formula">PLTAXD_02_40.balance + PLTAXD_02_42.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_uslug_s_trwale" model="account.report.line">
                        <field name="name">Basis - Acquisition of goods and services, fixed assets</field>
                        <field name="code">PLTAXD_02_40</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_uslug_s_trwale_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_40</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_uslug_pozostalych" model="account.report.line">
                        <field name="name">Base - Purchase of other goods and services</field>
                        <field name="code">PLTAXD_02_42</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_uslug_pozostalych_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_42</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_do_przeniesienia" model="account.report.line">
                <field name="name">Tax - To be carried over</field>
                <field name="code">PLTAX</field>
                <field name="aggregation_formula" eval="False"/>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_do_przeniesienia_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">PLTAXD.balance + PLTAX_49.balance + PLTAX_50.balance - PLTAXC.balance</field>
                    </record>
                    <record id="tax_report_line_do_przeniesienia_carryover" model="account.report.expression">
                        <field name="label">_carryover_balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">PLTAX.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                        <field name="carryover_target">PLTAXC_39._applied_carryover_balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_podatek_razem_c" model="account.report.line">
                        <field name="name">Tax - Total C</field>
                        <field name="code">PLTAXC</field>
                        <field name="aggregation_formula">PLTAXC_04_16.balance + PLTAXC_05_18.balance + PLTAXC_06_20.balance + PLTAXC_09_24.formula + PLTAXC_10_26.balance + PLTAXC_11_28.balance + PLTAXC_12_32.balance + PLTAXC_12_33.balance + PLTAXC_01_34.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_podatek_kraj_5" model="account.report.line">
                                <field name="name">Tax - Supply of goods/services, domestic, 5%</field>
                                <field name="code">PLTAXC_04_16</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_kraj_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_16</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_kraj_8" model="account.report.line">
                                <field name="name">Tax - Supply of goods/services, domestic, 8%</field>
                                <field name="code">PLTAXC_05_18</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_kraj_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_kraj_23" model="account.report.line">
                                <field name="name">Tax - Supply of goods/services, domestic, 23%</field>
                                <field name="code">PLTAXC_06_20</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_kraj_23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_nabycie_towarow" model="account.report.line">
                                <field name="name">Tax - Intra-Community acquisition of goods</field>
                                <field name="code">PLTAXC_09_24</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_nabycie_towarow_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_24</field>
                                    </record>
                                    <record id="account_tax_report_line_podatek_nabycie_towarow_formula" model="account.report.expression">
                                        <field name="label">formula</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">PLTAXC_09_24.balance + PLTAXC_10_35.balance</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_podatek_transp_termin" model="account.report.line">
                                        <field name="name">Tax - Inter-Community acquisition of means of transport</field>
                                        <field name="code">PLTAXC_10_35</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_podatek_transp_termin_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">K_35</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_art_33a" model="account.report.line">
                                <field name="name">Tax - Importation of goods under art. 33a</field>
                                <field name="code">PLTAXC_10_26</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_art_33a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_26</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_import_uslug" model="account.report.line">
                                <field name="name">Tax - Importation of services</field>
                                <field name="code">PLTAXC_11_28</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_import_uslug_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_28</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_podatek_art_28b" model="account.report.line">
                                        <field name="name">Tax - Acquisition under art. 28b</field>
                                        <field name="code">PLTAXC_11a_30</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_podatek_art_28b_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">K_30</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_podatnik_nabywca" model="account.report.line">
                                <field name="name">Tax - Supply of goods, taxable person acquiring</field>
                                <field name="code">PLTAXC_12_32</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_podatnik_nabywca_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_32</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_art_14_5" model="account.report.line">
                                <field name="name">Tax - From physical inventory under art. 14.5</field>
                                <field name="code">PLTAXC_12_33</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_art_14_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_33</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_kasy_rejestrujace" model="account.report.line">
                                <field name="name">Tax - Expenditure on cash registers</field>
                                <field name="code">PLTAXC_01_34</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_kasy_rejestrujace_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_34</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_wewnątrzwspólnotowe_103_5a" model="account.report.line">
                                <field name="name">Tax - Intra-Community acquisition of goods under art. 103 sec. 5a</field>
                                <field name="code">PLTAXC_01_36</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_wewnątrzwspólnotowe_103_5a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_36</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_podatek_razem_d" model="account.report.line">
                        <field name="name">Tax - Total D</field>
                        <field name="code">PLTAXD</field>
                        <field name="aggregation_formula">PLTAXC_39.balance + PLTAXD_02_41.balance + PLTAXD_02_43.balance + PLTAXD_02_44.balance + PLTAXD_02_45.balance + PLTAXD_02_46.balance + PLTAXD_02_47.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_podatek_deklaracji" model="account.report.line">
                                <field name="name">Tax - Surplus from previous declaration</field>
                                <field name="code">PLTAXC_39</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_deklaracji_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="account_tax_report_line_podatek_deklaracji_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_39</field>
                                    </record>
                                    <record id="account_tax_report_line_podatek_deklaracji_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">PLTAXC_39.tag + PLTAXC_39._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_s_trwale" model="account.report.line">
                                <field name="name">Tax - Acquisition of goods and services, fixed assets</field>
                                <field name="code">PLTAXD_02_41</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_s_trwale_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_41</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_uslug_pozostalych" model="account.report.line">
                                <field name="name">Tax - Purchase of other goods and services</field>
                                <field name="code">PLTAXD_02_43</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_uslug_pozostalych_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_43</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_s_trwalych" model="account.report.line">
                                <field name="name">Tax - Adjustment of input tax on acquisition of fixed assets</field>
                                <field name="code">PLTAXD_02_44</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_s_trwalych_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_44</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_pozostalych_nabyc" model="account.report.line">
                                <field name="name">Tax - Adjustment of input tax on other acquisitions</field>
                                <field name="code">PLTAXD_02_45</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_pozostalych_nabyc_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_45</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_korekta_art89b1" model="account.report.line">
                                <field name="name">Tax - Input tax adjustments under art. 89b sec 1</field>
                                <field name="code">PLTAXD_02_46</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_korekta_art89b1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_46</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_korekta_art89b4" model="account.report.line">
                                <field name="name">Tax - Input tax adjustments under art. 89b sec 4</field>
                                <field name="code">PLTAXD_02_47</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_korekta_art89b4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_47</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_podatek_okresie" model="account.report.line">
                        <field name="name">Tax - Expenditure on cash registers to be reimbursed in the period</field>
                        <field name="code">PLTAX_49</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_podatek_okresie_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_zaniechaniem_poboru" model="account.report.line">
                        <field name="name">Tax - Subject to non-collection</field>
                        <field name="code">PLTAX_50</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_zaniechaniem_poboru_tag" model="account.report.expression">
                                <field name="label">balance</field>
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

## File: data\l10n_pl.l10n_pl_tax_office.csv

```csv
"id","code","name"
"pl_tax_office_0202","0202","URZĄD SKARBOWY W BOLESŁAWCU"
"pl_tax_office_0203","0203","URZĄD SKARBOWY W BYSTRZYCY KŁODZKIEJ"
"pl_tax_office_0204","0204","URZĄD SKARBOWY W DZIERŻONIOWIE"
"pl_tax_office_0205","0205","URZĄD SKARBOWY W GŁOGOWIE"
"pl_tax_office_0206","0206","URZĄD SKARBOWY W JAWORZE"
"pl_tax_office_0207","0207","URZĄD SKARBOWY W JELENIEJ GÓRZE"
"pl_tax_office_0208","0208","URZĄD SKARBOWY W KAMIENNEJ GÓRZE"
"pl_tax_office_0209","0209","URZĄD SKARBOWY W KŁODZKU"
"pl_tax_office_0210","0210","URZĄD SKARBOWY W LEGNICY"
"pl_tax_office_0211","0211","URZĄD SKARBOWY W LUBANIU"
"pl_tax_office_0212","0212","URZĄD SKARBOWY W LUBINIE"
"pl_tax_office_0213","0213","URZĄD SKARBOWY W LWÓWKU ŚLĄSKIM"
"pl_tax_office_0214","0214","URZĄD SKARBOWY W MILICZU"
"pl_tax_office_0215","0215","URZĄD SKARBOWY W NOWEJ RUDZIE"
"pl_tax_office_0216","0216","URZĄD SKARBOWY W OLEŚNICY"
"pl_tax_office_0217","0217","URZĄD SKARBOWY W OŁAWIE"
"pl_tax_office_0218","0218","URZĄD SKARBOWY W STRZELINIE"
"pl_tax_office_0219","0219","URZĄD SKARBOWY W ŚRODZIE ŚLĄSKIEJ"
"pl_tax_office_0220","0220","URZĄD SKARBOWY W ŚWIDNICY"
"pl_tax_office_0221","0221","URZĄD SKARBOWY W TRZEBNICY"
"pl_tax_office_0222","0222","URZĄD SKARBOWY W WAŁBRZYCHU"
"pl_tax_office_0223","0223","URZĄD SKARBOWY W WOŁOWIE"
"pl_tax_office_0224","0224","URZĄD SKARBOWY WROCŁAW-FABRYCZNA"
"pl_tax_office_0225","0225","URZĄD SKARBOWY WROCŁAW-KRZYKI"
"pl_tax_office_0226","0226","URZĄD SKARBOWY WROCŁAW-PSIE POLE"
"pl_tax_office_0227","0227","URZĄD SKARBOWY WROCŁAW-STARE MIASTO"
"pl_tax_office_0228","0228","URZĄD SKARBOWY WROCŁAW-ŚRÓDMIEŚCIE"
"pl_tax_office_0229","0229","PIERWSZY URZĄD SKARBOWY WE WROCŁAWIU"
"pl_tax_office_0230","0230","URZĄD SKARBOWY W ZĄBKOWICACH ŚLĄSKICH"
"pl_tax_office_0231","0231","URZĄD SKARBOWY W ZGORZELCU"
"pl_tax_office_0232","0232","URZĄD SKARBOWY W ZŁOTORYI"
"pl_tax_office_0233","0233","URZĄD SKARBOWY W GÓRZE"
"pl_tax_office_0234","0234","URZĄD SKARBOWY W POLKOWICACH"
"pl_tax_office_0271","0271","DOLNOŚLĄSKI URZĄD SKARBOWY WE WROCŁAWIU"
"pl_tax_office_0402","0402","URZĄD SKARBOWY W ALEKSANDROWIE KUJAWSKIM"
"pl_tax_office_0403","0403","URZĄD SKARBOWY W BRODNICY"
"pl_tax_office_0404","0404","PIERWSZY URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0405","0405","DRUGI URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0406","0406","TRZECI URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0407","0407","URZĄD SKARBOWY W CHEŁMNIE"
"pl_tax_office_0408","0408","URZĄD SKARBOWY W GRUDZIĄDZU"
"pl_tax_office_0409","0409","URZĄD SKARBOWY W INOWROCŁAWIU"
"pl_tax_office_0410","0410","URZĄD SKARBOWY W LIPNIE"
"pl_tax_office_0411","0411","URZĄD SKARBOWY W MOGILNIE"
"pl_tax_office_0412","0412","URZĄD SKARBOWY W NAKLE NAD NOTECIĄ"
"pl_tax_office_0413","0413","URZĄD SKARBOWY W RADZIEJOWIE"
"pl_tax_office_0414","0414","URZĄD SKARBOWY W RYPINIE"
"pl_tax_office_0415","0415","URZĄD SKARBOWY W ŚWIECIU"
"pl_tax_office_0416","0416","PIERWSZY URZĄD SKARBOWY W TORUNIU"
"pl_tax_office_0417","0417","DRUGI URZĄD SKARBOWY W TORUNIU"
"pl_tax_office_0418","0418","URZĄD SKARBOWY W TUCHOLI"
"pl_tax_office_0419","0419","URZĄD SKARBOWY W WĄBRZEŹNIE"
"pl_tax_office_0420","0420","URZĄD SKARBOWY WE WŁOCŁAWKU"
"pl_tax_office_0421","0421","URZĄD SKARBOWY W ŻNINIE"
"pl_tax_office_0422","0422","URZĄD SKARBOWY W GOLUBIU-DOBRZYNIU"
"pl_tax_office_0423","0423","URZĄD SKARBOWY W SĘPÓLNIE KRAJEŃSKIM"
"pl_tax_office_0471","0471","KUJAWSKO-POMORSKI URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0602","0602","URZĄD SKARBOWY W BIAŁEJ PODLASKIEJ"
"pl_tax_office_0603","0603","URZĄD SKARBOWY W BIŁGORAJU"
"pl_tax_office_0604","0604","URZĄD SKARBOWY W CHEŁMIE"
"pl_tax_office_0605","0605","URZĄD SKARBOWY W HRUBIESZOWIE"
"pl_tax_office_0606","0606","URZĄD SKARBOWY W JANOWIE LUBELSKIM"
"pl_tax_office_0607","0607","URZĄD SKARBOWY W KRASNYMSTAWIE"
"pl_tax_office_0608","0608","URZĄD SKARBOWY W KRAŚNIKU"
"pl_tax_office_0609","0609","URZĄD SKARBOWY W LUBARTOWIE"
"pl_tax_office_0610","0610","PIERWSZY URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0611","0611","DRUGI URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0612","0612","TRZECI URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0613","0613","URZĄD SKARBOWY W ŁUKOWIE"
"pl_tax_office_0614","0614","URZĄD SKARBOWY W OPOLU LUBELSKIM"
"pl_tax_office_0615","0615","URZĄD SKARBOWY W PARCZEWIE"
"pl_tax_office_0616","0616","URZĄD SKARBOWY W PUŁAWACH"
"pl_tax_office_0617","0617","URZĄD SKARBOWY W RADZYNIU PODLASKIM"
"pl_tax_office_0618","0618","URZĄD SKARBOWY W TOMASZOWIE LUBELSKIM"
"pl_tax_office_0619","0619","URZĄD SKARBOWY WE WŁODAWIE"
"pl_tax_office_0620","0620","URZĄD SKARBOWY W ZAMOŚCIU"
"pl_tax_office_0621","0621","URZĄD SKARBOWY W ŁĘCZNEJ"
"pl_tax_office_0622","0622","URZĄD SKARBOWY W RYKACH"
"pl_tax_office_0671","0671","LUBELSKI URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0802","0802","URZĄD SKARBOWY W GORZOWIE WIELKOPOLSKIM"
"pl_tax_office_0803","0803","URZĄD SKARBOWY W KROŚNIE ODRZAŃSKIM"
"pl_tax_office_0804","0804","URZĄD SKARBOWY W MIĘDZYRZECZU"
"pl_tax_office_0805","0805","URZĄD SKARBOWY W NOWEJ SOLI"
"pl_tax_office_0806","0806","URZĄD SKARBOWY W SŁUBICACH"
"pl_tax_office_0807","0807","URZĄD SKARBOWY W ŚWIEBODZINIE"
"pl_tax_office_0808","0808","PIERWSZY URZĄD SKARBOWY W ZIELONEJ GÓRZE"
"pl_tax_office_0809","0809","DRUGI URZĄD SKARBOWY W ZIELONEJ GÓRZE"
"pl_tax_office_0810","0810","URZĄD SKARBOWY W ŻAGANIU"
"pl_tax_office_0811","0811","URZĄD SKARBOWY W ŻARACH"
"pl_tax_office_0812","0812","URZĄD SKARBOWY W DREZDENKU"
"pl_tax_office_0813","0813","URZĄD SKARBOWY W SULĘCINIE"
"pl_tax_office_0814","0814","URZĄD SKARBOWY WE WSCHOWIE"
"pl_tax_office_0871","0871","LUBUSKI URZĄD SKARBOWY W ZIELONEJ GÓRZE"
"pl_tax_office_1002","1002","URZĄD SKARBOWY W BEŁCHATOWIE"
"pl_tax_office_1003","1003","URZĄD SKARBOWY W BRZEZINACH"
"pl_tax_office_1004","1004","URZĄD SKARBOWY W GŁOWNIE"
"pl_tax_office_1005","1005","URZĄD SKARBOWY W KUTNIE"
"pl_tax_office_1006","1006","URZĄD SKARBOWY W ŁASKU"
"pl_tax_office_1007","1007","URZĄD SKARBOWY W ŁOWICZU"
"pl_tax_office_1008","1008","PIERWSZY URZĄD SKARBOWY ŁÓDŹ-BAŁUTY"
"pl_tax_office_1009","1009","DRUGI URZĄD SKARBOWY ŁÓDŹ-BAŁUTY"
"pl_tax_office_1010","1010","PIERWSZY URZĄD SKARBOWY ŁÓDŹ-GÓRNA"
"pl_tax_office_1011","1011","DRUGI URZĄD SKARBOWY ŁÓDŹ-GÓRNA"
"pl_tax_office_1012","1012","URZĄD SKARBOWY ŁÓDŹ-POLESIE"
"pl_tax_office_1013","1013","URZĄD SKARBOWY ŁÓDŹ-ŚRÓDMIEŚCIE"
"pl_tax_office_1014","1014","URZĄD SKARBOWY ŁÓDŹ-WIDZEW"
"pl_tax_office_1015","1015","URZĄD SKARBOWY W OPOCZNIE"
"pl_tax_office_1016","1016","URZĄD SKARBOWY W PABIANICACH"
"pl_tax_office_1017","1017","URZĄD SKARBOWY W PIOTRKOWIE TRYBUNALSKIM"
"pl_tax_office_1018","1018","URZĄD SKARBOWY W PODDĘBICACH"
"pl_tax_office_1019","1019","URZĄD SKARBOWY W RADOMSKU"
"pl_tax_office_1020","1020","URZĄD SKARBOWY W RAWIE MAZOWIECKIEJ"
"pl_tax_office_1021","1021","URZĄD SKARBOWY W SIERADZU"
"pl_tax_office_1022","1022","URZĄD SKARBOWY W SKIERNIEWICACH"
"pl_tax_office_1023","1023","URZĄD SKARBOWY W TOMASZOWIE MAZOWIECKIM"
"pl_tax_office_1024","1024","URZĄD SKARBOWY W WIELUNIU"
"pl_tax_office_1025","1025","URZĄD SKARBOWY W ZDUŃSKIEJ WOLI"
"pl_tax_office_1026","1026","URZĄD SKARBOWY W ZGIERZU"
"pl_tax_office_1027","1027","URZĄD SKARBOWY W WIERUSZOWIE"
"pl_tax_office_1028","1028","URZĄD SKARBOWY W ŁĘCZYCY"
"pl_tax_office_1029","1029","URZĄD SKARBOWY W PAJĘCZNIE"
"pl_tax_office_1071","1071","ŁÓDZKI URZĄD SKARBOWY W ŁODZI"
"pl_tax_office_1202","1202","URZĄD SKARBOWY W BOCHNI"
"pl_tax_office_1203","1203","URZĄD SKARBOWY W BRZESKU"
"pl_tax_office_1204","1204","URZĄD SKARBOWY W CHRZANOWIE"
"pl_tax_office_1205","1205","URZĄD SKARBOWY W DĄBROWIE TARNOWSKIEJ"
"pl_tax_office_1206","1206","URZĄD SKARBOWY W GORLICACH"
"pl_tax_office_1207","1207","PIERWSZY URZĄD SKARBOWY KRAKÓW"
"pl_tax_office_1208","1208","URZĄD SKARBOWY KRAKÓW-KROWODRZA"
"pl_tax_office_1209","1209","URZĄD SKARBOWY KRAKÓW-NOWA HUTA"
"pl_tax_office_1210","1210","URZĄD SKARBOWY KRAKÓW-PODGÓRZE"
"pl_tax_office_1211","1211","URZĄD SKARBOWY KRAKÓW-PRĄDNIK"
"pl_tax_office_1212","1212","URZĄD SKARBOWY KRAKÓW-STARE MIASTO"
"pl_tax_office_1213","1213","URZĄD SKARBOWY KRAKÓW-ŚRÓDMIEŚCIE"
"pl_tax_office_1214","1214","URZĄD SKARBOWY W LIMANOWEJ"
"pl_tax_office_1215","1215","URZĄD SKARBOWY W MIECHOWIE"
"pl_tax_office_1216","1216","URZĄD SKARBOWY W MYŚLENICACH"
"pl_tax_office_1217","1217","URZĄD SKARBOWY W NOWYM SĄCZU"
"pl_tax_office_1218","1218","URZĄD SKARBOWY W NOWYM TARGU"
"pl_tax_office_1219","1219","URZĄD SKARBOWY W OLKUSZU"
"pl_tax_office_1220","1220","URZĄD SKARBOWY W OŚWIĘCIMIU"
"pl_tax_office_1221","1221","URZĄD SKARBOWY W PROSZOWICACH"
"pl_tax_office_1222","1222","URZĄD SKARBOWY W SUCHEJ BESKIDZKIEJ"
"pl_tax_office_1223","1223","PIERWSZY URZĄD SKARBOWY W TARNOWIE"
"pl_tax_office_1224","1224","DRUGI URZĄD SKARBOWY W TARNOWIE"
"pl_tax_office_1225","1225","URZĄD SKARBOWY W WADOWICACH"
"pl_tax_office_1226","1226","URZĄD SKARBOWY W WIELICZCE"
"pl_tax_office_1227","1227","URZĄD SKARBOWY W ZAKOPANEM"
"pl_tax_office_1228","1228","DRUGI URZĄD SKARBOWY KRAKÓW"
"pl_tax_office_1271","1271","MAŁOPOLSKI URZĄD SKARBOWY W KRAKOWIE"
"pl_tax_office_1402","1402","URZĄD SKARBOWY W BIAŁOBRZEGACH"
"pl_tax_office_1403","1403","URZĄD SKARBOWY W CIECHANOWIE"
"pl_tax_office_1404","1404","URZĄD SKARBOWY W GARWOLINIE"
"pl_tax_office_1405","1405","URZĄD SKARBOWY W GOSTYNINIE"
"pl_tax_office_1406","1406","URZĄD SKARBOWY W GRODZISKU MAZOWIECKIM"
"pl_tax_office_1407","1407","URZĄD SKARBOWY W GRÓJCU"
"pl_tax_office_1408","1408","URZĄD SKARBOWY W KOZIENICACH"
"pl_tax_office_1409","1409","URZĄD SKARBOWY W LEGIONOWIE"
"pl_tax_office_1410","1410","URZĄD SKARBOWY W ŁOSICACH"
"pl_tax_office_1411","1411","URZĄD SKARBOWY W MAKOWIE MAZOWIECKIM"
"pl_tax_office_1412","1412","URZĄD SKARBOWY W MIŃSKU MAZOWIECKIM"
"pl_tax_office_1413","1413","URZĄD SKARBOWY W MŁAWIE"
"pl_tax_office_1414","1414","URZĄD SKARBOWY W NOWYM DWORZE MAZOWIECKIM"
"pl_tax_office_1415","1415","URZĄD SKARBOWY W OSTROŁĘCE"
"pl_tax_office_1416","1416","URZĄD SKARBOWY W OSTROWI MAZOWIECKIEJ"
"pl_tax_office_1417","1417","URZĄD SKARBOWY W OTWOCKU"
"pl_tax_office_1418","1418","URZĄD SKARBOWY W PIASECZNIE"
"pl_tax_office_1419","1419","URZĄD SKARBOWY W PŁOCKU"
"pl_tax_office_1420","1420","URZĄD SKARBOWY W PŁOŃSKU"
"pl_tax_office_1421","1421","URZĄD SKARBOWY W PRUSZKOWIE"
"pl_tax_office_1422","1422","URZĄD SKARBOWY W PRZASNYSZU"
"pl_tax_office_1423","1423","URZĄD SKARBOWY W PUŁTUSKU"
"pl_tax_office_1424","1424","PIERWSZY URZĄD SKARBOWY W RADOMIU"
"pl_tax_office_1425","1425","DRUGI URZĄD SKARBOWY W RADOMIU"
"pl_tax_office_1426","1426","URZĄD SKARBOWY W SIEDLCACH"
"pl_tax_office_1427","1427","URZĄD SKARBOWY W SIERPCU"
"pl_tax_office_1428","1428","URZĄD SKARBOWY W SOCHACZEWIE"
"pl_tax_office_1429","1429","URZĄD SKARBOWY W SOKOŁOWIE PODLASKIM"
"pl_tax_office_1430","1430","URZĄD SKARBOWY W SZYDŁOWCU"
"pl_tax_office_1431","1431","URZĄD SKARBOWY WARSZAWA-BEMOWO"
"pl_tax_office_1432","1432","URZĄD SKARBOWY WARSZAWA-BIELANY"
"pl_tax_office_1433","1433","URZĄD SKARBOWY WARSZAWA-MOKOTÓW"
"pl_tax_office_1434","1434","URZĄD SKARBOWY WARSZAWA-PRAGA"
"pl_tax_office_1435","1435","PIERWSZY URZĄD SKARBOWY WARSZAWA-ŚRÓDMIEŚCIE"
"pl_tax_office_1436","1436","DRUGI URZĄD SKARBOWY WARSZAWA-ŚRÓDMIEŚCIE"
"pl_tax_office_1437","1437","URZĄD SKARBOWY WARSZAWA-TARGÓWEK"
"pl_tax_office_1438","1438","URZĄD SKARBOWY WARSZAWA-URSYNÓW"
"pl_tax_office_1439","1439","URZĄD SKARBOWY WARSZAWA-WAWER"
"pl_tax_office_1440","1440","URZĄD SKARBOWY WARSZAWA-WOLA"
"pl_tax_office_1441","1441","URZĄD SKARBOWY W WĘGROWIE"
"pl_tax_office_1442","1442","URZĄD SKARBOWY W WOŁOMINIE"
"pl_tax_office_1443","1443","URZĄD SKARBOWY W WYSZKOWIE"
"pl_tax_office_1444","1444","URZĄD SKARBOWY W ZWOLENIU"
"pl_tax_office_1445","1445","URZĄD SKARBOWY W ŻUROMINIE"
"pl_tax_office_1446","1446","URZĄD SKARBOWY W ŻYRARDOWIE"
"pl_tax_office_1447","1447","URZĄD SKARBOWY W LIPSKU"
"pl_tax_office_1448","1448","URZĄD SKARBOWY W PRZYSUSZE"
"pl_tax_office_1449","1449","TRZECI URZĄD SKARBOWY WARSZAWA-ŚRÓDMIEŚCIE"
"pl_tax_office_1471","1471","PIERWSZY MAZOWIECKI URZĄD SKARBOWY W WARSZAWIE"
"pl_tax_office_1472","1472","DRUGI MAZOWIECKI URZĄD SKARBOWY W WARSZAWIE"
"pl_tax_office_1473","1473","TRZECI MAZOWIECKI URZĄD SKARBOWY W RADOMIU"
"pl_tax_office_1602","1602","URZĄD SKARBOWY W BRZEGU"
"pl_tax_office_1603","1603","URZĄD SKARBOWY W GŁUBCZYCACH"
"pl_tax_office_1604","1604","URZĄD SKARBOWY W KĘDZIERZYNIE-KOŹLU"
"pl_tax_office_1605","1605","URZĄD SKARBOWY W KLUCZBORKU"
"pl_tax_office_1606","1606","URZĄD SKARBOWY W NAMYSŁOWIE"
"pl_tax_office_1607","1607","URZĄD SKARBOWY W NYSIE"
"pl_tax_office_1608","1608","URZĄD SKARBOWY W OLEŚNIE"
"pl_tax_office_1609","1609","PIERWSZY URZĄD SKARBOWY W OPOLU"
"pl_tax_office_1610","1610","DRUGI URZĄD SKARBOWY W OPOLU"
"pl_tax_office_1611","1611","URZĄD SKARBOWY W PRUDNIKU"
"pl_tax_office_1612","1612","URZĄD SKARBOWY W STRZELCACH OPOLSKICH"
"pl_tax_office_1613","1613","URZĄD SKARBOWY W KRAPKOWICACH"
"pl_tax_office_1671","1671","OPOLSKI URZĄD SKARBOWY W OPOLU"
"pl_tax_office_1802","1802","URZĄD SKARBOWY W BRZOZOWIE"
"pl_tax_office_1803","1803","URZĄD SKARBOWY W DĘBICY"
"pl_tax_office_1804","1804","URZĄD SKARBOWY W JAROSŁAWIU"
"pl_tax_office_1805","1805","URZĄD SKARBOWY W JAŚLE"
"pl_tax_office_1806","1806","URZĄD SKARBOWY W KOLBUSZOWEJ"
"pl_tax_office_1807","1807","URZĄD SKARBOWY W KROŚNIE"
"pl_tax_office_1808","1808","URZĄD SKARBOWY W LESKU"
"pl_tax_office_1809","1809","URZĄD SKARBOWY W LEŻAJSKU"
"pl_tax_office_1810","1810","URZĄD SKARBOWY W LUBACZOWIE"
"pl_tax_office_1811","1811","URZĄD SKARBOWY W ŁAŃCUCIE"
"pl_tax_office_1812","1812","URZĄD SKARBOWY W MIELCU"
"pl_tax_office_1813","1813","URZĄD SKARBOWY W PRZEMYŚLU"
"pl_tax_office_1814","1814","URZĄD SKARBOWY W PRZEWORSKU"
"pl_tax_office_1815","1815","URZĄD SKARBOWY W ROPCZYCACH"
"pl_tax_office_1816","1816","PIERWSZY URZĄD SKARBOWY W RZESZOWIE"
"pl_tax_office_1817","1817","URZĄD SKARBOWY W SANOKU"
"pl_tax_office_1818","1818","URZĄD SKARBOWY W STALOWEJ WOLI"
"pl_tax_office_1819","1819","URZĄD SKARBOWY W STRZYŻOWIE"
"pl_tax_office_1820","1820","URZĄD SKARBOWY W TARNOBRZEGU"
"pl_tax_office_1821","1821","URZĄD SKARBOWY W USTRZYKACH DOLNYCH"
"pl_tax_office_1822","1822","DRUGI URZĄD SKARBOWY W RZESZOWIE"
"pl_tax_office_1823","1823","URZĄD SKARBOWY W NISKU"
"pl_tax_office_1871","1871","PODKARPACKI URZĄD SKARBOWY W RZESZOWIE"
"pl_tax_office_2002","2002","URZĄD SKARBOWY W AUGUSTOWIE"
"pl_tax_office_2003","2003","PIERWSZY URZĄD SKARBOWY W BIAŁYMSTOKU"
"pl_tax_office_2004","2004","DRUGI URZĄD SKARBOWY W BIAŁYMSTOKU"
"pl_tax_office_2005","2005","URZĄD SKARBOWY W BIELSKU PODLASKIM"
"pl_tax_office_2006","2006","URZĄD SKARBOWY W GRAJEWIE"
"pl_tax_office_2007","2007","URZĄD SKARBOWY W KOLNIE"
"pl_tax_office_2008","2008","URZĄD SKARBOWY W ŁOMŻY"
"pl_tax_office_2009","2009","URZĄD SKARBOWY W MOŃKACH"
"pl_tax_office_2010","2010","URZĄD SKARBOWY W SIEMIATYCZACH"
"pl_tax_office_2011","2011","URZĄD SKARBOWY W SOKÓŁCE"
"pl_tax_office_2012","2012","URZĄD SKARBOWY W SUWAŁKACH"
"pl_tax_office_2013","2013","URZĄD SKARBOWY W WYSOKIEM MAZOWIECKIEM"
"pl_tax_office_2014","2014","URZĄD SKARBOWY W ZAMBROWIE"
"pl_tax_office_2015","2015","URZĄD SKARBOWY W HAJNÓWCE"
"pl_tax_office_2071","2071","PODLASKI URZĄD SKARBOWY W BIAŁYMSTOKU"
"pl_tax_office_2202","2202","URZĄD SKARBOWY W BYTOWIE"
"pl_tax_office_2203","2203","URZĄD SKARBOWY W CHOJNICACH"
"pl_tax_office_2204","2204","URZĄD SKARBOWY W CZŁUCHOWIE"
"pl_tax_office_2205","2205","PIERWSZY URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2206","2206","DRUGI URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2207","2207","TRZECI URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2208","2208","PIERWSZY URZĄD SKARBOWY W GDYNI"
"pl_tax_office_2209","2209","DRUGI URZĄD SKARBOWY W GDYNI"
"pl_tax_office_2210","2210","URZĄD SKARBOWY W KARTUZACH"
"pl_tax_office_2211","2211","URZĄD SKARBOWY W KOŚCIERZYNIE"
"pl_tax_office_2212","2212","URZĄD SKARBOWY W KWIDZYNIE"
"pl_tax_office_2213","2213","URZĄD SKARBOWY W LĘBORKU"
"pl_tax_office_2214","2214","URZĄD SKARBOWY W MALBORKU"
"pl_tax_office_2215","2215","URZĄD SKARBOWY W PUCKU"
"pl_tax_office_2216","2216","URZĄD SKARBOWY W SŁUPSKU"
"pl_tax_office_2217","2217","URZĄD SKARBOWY W SOPOCIE"
"pl_tax_office_2218","2218","URZĄD SKARBOWY W STAROGARDZIE GDAŃSKIM"
"pl_tax_office_2219","2219","URZĄD SKARBOWY W TCZEWIE"
"pl_tax_office_2220","2220","URZĄD SKARBOWY W WEJHEROWIE"
"pl_tax_office_2221","2221","URZĄD SKARBOWY W PRUSZCZU GDAŃSKIM"
"pl_tax_office_2271","2271","POMORSKI URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2402","2402","URZĄD SKARBOWY W BĘDZINIE"
"pl_tax_office_2403","2403","PIERWSZY URZĄD SKARBOWY W BIELSKU-BIAŁEJ"
"pl_tax_office_2404","2404","DRUGI URZĄD SKARBOWY W BIELSKU-BIAŁEJ"
"pl_tax_office_2405","2405","URZĄD SKARBOWY W BYTOMIU"
"pl_tax_office_2406","2406","URZĄD SKARBOWY W CHORZOWIE"
"pl_tax_office_2407","2407","URZĄD SKARBOWY W CIESZYNIE"
"pl_tax_office_2408","2408","URZĄD SKARBOWY W CZECHOWICACH-DZIEDZICACH"
"pl_tax_office_2409","2409","PIERWSZY URZĄD SKARBOWY W CZĘSTOCHOWIE"
"pl_tax_office_2410","2410","DRUGI URZĄD SKARBOWY W CZĘSTOCHOWIE"
"pl_tax_office_2411","2411","URZĄD SKARBOWY W DĄBROWIE GÓRNICZEJ"
"pl_tax_office_2412","2412","PIERWSZY URZĄD SKARBOWY W GLIWICACH"
"pl_tax_office_2413","2413","DRUGI URZĄD SKARBOWY W GLIWICACH"
"pl_tax_office_2414","2414","URZĄD SKARBOWY W JASTRZĘBIU-ZDROJU"
"pl_tax_office_2415","2415","URZĄD SKARBOWY W JAWORZNIE"
"pl_tax_office_2416","2416","PIERWSZY URZĄD SKARBOWY W KATOWICACH"
"pl_tax_office_2417","2417","DRUGI URZĄD SKARBOWY W KATOWICACH"
"pl_tax_office_2418","2418","URZĄD SKARBOWY W KŁOBUCKU"
"pl_tax_office_2419","2419","URZĄD SKARBOWY W LUBLIŃCU"
"pl_tax_office_2420","2420","URZĄD SKARBOWY W MIKOŁOWIE"
"pl_tax_office_2421","2421","URZĄD SKARBOWY W MYSŁOWICACH"
"pl_tax_office_2422","2422","URZĄD SKARBOWY W MYSZKOWIE"
"pl_tax_office_2423","2423","URZĄD SKARBOWY W PIEKARACH ŚLĄSKICH"
"pl_tax_office_2424","2424","URZĄD SKARBOWY W PSZCZYNIE"
"pl_tax_office_2425","2425","URZĄD SKARBOWY W RACIBORZU"
"pl_tax_office_2426","2426","URZĄD SKARBOWY W RUDZIE ŚLĄSKIEJ"
"pl_tax_office_2427","2427","URZĄD SKARBOWY W RYBNIKU"
"pl_tax_office_2428","2428","URZĄD SKARBOWY W SIEMIANOWICACH ŚLĄSKICH"
"pl_tax_office_2429","2429","URZĄD SKARBOWY W SOSNOWCU"
"pl_tax_office_2430","2430","URZĄD SKARBOWY W TARNOWSKICH GÓRACH"
"pl_tax_office_2431","2431","URZĄD SKARBOWY W TYCHACH"
"pl_tax_office_2432","2432","URZĄD SKARBOWY W WODZISŁAWIU ŚLĄSKIM"
"pl_tax_office_2433","2433","URZĄD SKARBOWY W ZABRZU"
"pl_tax_office_2434","2434","URZĄD SKARBOWY W ZAWIERCIU"
"pl_tax_office_2435","2435","URZĄD SKARBOWY W ŻORACH"
"pl_tax_office_2436","2436","URZĄD SKARBOWY W ŻYWCU"
"pl_tax_office_2471","2471","PIERWSZY ŚLĄSKI URZĄD SKARBOWY W SOSNOWCU"
"pl_tax_office_2472","2472","DRUGI ŚLĄSKI URZĄD SKARBOWY W BIELSKU-BIAŁEJ"
"pl_tax_office_2602","2602","URZĄD SKARBOWY W BUSKU-ZDROJU"
"pl_tax_office_2603","2603","URZĄD SKARBOWY W JĘDRZEJOWIE"
"pl_tax_office_2604","2604","PIERWSZY URZĄD SKARBOWY W KIELCACH"
"pl_tax_office_2605","2605","DRUGI URZĄD SKARBOWY W KIELCACH"
"pl_tax_office_2606","2606","URZĄD SKARBOWY W KOŃSKICH"
"pl_tax_office_2607","2607","URZĄD SKARBOWY W OPATOWIE"
"pl_tax_office_2608","2608","URZĄD SKARBOWY W OSTROWCU ŚWIĘTOKRZYSKIM"
"pl_tax_office_2609","2609","URZĄD SKARBOWY W PIŃCZOWIE"
"pl_tax_office_2610","2610","URZĄD SKARBOWY W SANDOMIERZU"
"pl_tax_office_2611","2611","URZĄD SKARBOWY W SKARŻYSKU-KAMIENNEJ"
"pl_tax_office_2612","2612","URZĄD SKARBOWY W STARACHOWICACH"
"pl_tax_office_2613","2613","URZĄD SKARBOWY W STASZOWIE"
"pl_tax_office_2614","2614","URZĄD SKARBOWY W KAZIMIERZY WIELKIEJ"
"pl_tax_office_2615","2615","URZĄD SKARBOWY WE WŁOSZCZOWIE"
"pl_tax_office_2671","2671","ŚWIĘTOKRZYSKI URZĄD SKARBOWY W KIELCACH"
"pl_tax_office_2802","2802","URZĄD SKARBOWY W BARTOSZYCACH"
"pl_tax_office_2803","2803","URZĄD SKARBOWY W BRANIEWIE"
"pl_tax_office_2804","2804","URZĄD SKARBOWY W DZIAŁDOWIE"
"pl_tax_office_2805","2805","URZĄD SKARBOWY W ELBLĄGU"
"pl_tax_office_2806","2806","URZĄD SKARBOWY W EŁKU"
"pl_tax_office_2807","2807","URZĄD SKARBOWY W GIŻYCKU"
"pl_tax_office_2808","2808","URZĄD SKARBOWY W IŁAWIE"
"pl_tax_office_2809","2809","URZĄD SKARBOWY W KĘTRZYNIE"
"pl_tax_office_2810","2810","URZĄD SKARBOWY W NIDZICY"
"pl_tax_office_2811","2811","URZĄD SKARBOWY W NOWYM MIEŚCIE LUBAWSKIM"
"pl_tax_office_2812","2812","URZĄD SKARBOWY W OLECKU"
"pl_tax_office_2813","2813","URZĄD SKARBOWY W OLSZTYNIE"
"pl_tax_office_2814","2814","URZĄD SKARBOWY W OSTRÓDZIE"
"pl_tax_office_2815","2815","URZĄD SKARBOWY W PISZU"
"pl_tax_office_2816","2816","URZĄD SKARBOWY W SZCZYTNIE"
"pl_tax_office_2871","2871","WARMIŃSKO-MAZURSKI URZĄD SKARBOWY W OLSZTYNIE"
"pl_tax_office_3002","3002","URZĄD SKARBOWY W CZARNKOWIE"
"pl_tax_office_3003","3003","URZĄD SKARBOWY W GNIEŹNIE"
"pl_tax_office_3004","3004","URZĄD SKARBOWY W GOSTYNIU"
"pl_tax_office_3005","3005","URZĄD SKARBOWY W GRODZISKU WIELKOPOLSKIM"
"pl_tax_office_3006","3006","URZĄD SKARBOWY W JAROCINIE"
"pl_tax_office_3007","3007","PIERWSZY URZĄD SKARBOWY W KALISZU"
"pl_tax_office_3008","3008","DRUGI URZĄD SKARBOWY W KALISZU"
"pl_tax_office_3009","3009","URZĄD SKARBOWY W KĘPNIE"
"pl_tax_office_3010","3010","URZĄD SKARBOWY W KOLE"
"pl_tax_office_3011","3011","URZĄD SKARBOWY W KONINIE"
"pl_tax_office_3012","3012","URZĄD SKARBOWY W KOŚCIANIE"
"pl_tax_office_3013","3013","URZĄD SKARBOWY W KROTOSZYNIE"
"pl_tax_office_3014","3014","URZĄD SKARBOWY W LESZNIE"
"pl_tax_office_3015","3015","URZĄD SKARBOWY W MIĘDZYCHODZIE"
"pl_tax_office_3016","3016","URZĄD SKARBOWY W NOWYM TOMYŚLU"
"pl_tax_office_3017","3017","URZĄD SKARBOWY W OSTROWIE WIELKOPOLSKIM"
"pl_tax_office_3018","3018","URZĄD SKARBOWY W OSTRZESZOWIE"
"pl_tax_office_3019","3019","URZĄD SKARBOWY W PILE"
"pl_tax_office_3020","3020","URZĄD SKARBOWY POZNAŃ-GRUNWALD"
"pl_tax_office_3021","3021","URZĄD SKARBOWY POZNAŃ-JEŻYCE"
"pl_tax_office_3022","3022","URZĄD SKARBOWY POZNAŃ-NOWE MIASTO"
"pl_tax_office_3023","3023","PIERWSZY URZĄD SKARBOWY W POZNANIU"
"pl_tax_office_3025","3025","URZĄD SKARBOWY POZNAŃ-WINOGRADY"
"pl_tax_office_3026","3026","URZĄD SKARBOWY POZNAŃ-WILDA"
"pl_tax_office_3027","3027","URZĄD SKARBOWY W RAWICZU"
"pl_tax_office_3028","3028","URZĄD SKARBOWY W SŁUPCY"
"pl_tax_office_3029","3029","URZĄD SKARBOWY W SZAMOTUŁACH"
"pl_tax_office_3030","3030","URZĄD SKARBOWY W ŚREMIE"
"pl_tax_office_3031","3031","URZĄD SKARBOWY W ŚRODZIE WIELKOPOLSKIEJ"
"pl_tax_office_3032","3032","URZĄD SKARBOWY W TURKU"
"pl_tax_office_3033","3033","URZĄD SKARBOWY W WĄGROWCU"
"pl_tax_office_3034","3034","URZĄD SKARBOWY W WOLSZTYNIE"
"pl_tax_office_3035","3035","URZĄD SKARBOWY WE WRZEŚNI"
"pl_tax_office_3036","3036","URZĄD SKARBOWY W ZŁOTOWIE"
"pl_tax_office_3037","3037","URZĄD SKARBOWY W CHODZIEŻY"
"pl_tax_office_3038","3038","URZĄD SKARBOWY W OBORNIKACH"
"pl_tax_office_3039","3039","URZĄD SKARBOWY W PLESZEWIE"
"pl_tax_office_3071","3071","PIERWSZY WIELKOPOLSKI URZĄD SKARBOWY W POZNANIU"
"pl_tax_office_3072","3072","DRUGI WIELKOPOLSKI URZĄD SKARBOWY W KALISZU"
"pl_tax_office_3202","3202","URZĄD SKARBOWY W BIAŁOGARDZIE"
"pl_tax_office_3203","3203","URZĄD SKARBOWY W CHOSZCZNIE"
"pl_tax_office_3204","3204","URZĄD SKARBOWY W DRAWSKU POMORSKIM"
"pl_tax_office_3205","3205","URZĄD SKARBOWY W GOLENIOWIE"
"pl_tax_office_3206","3206","URZĄD SKARBOWY W GRYFICACH"
"pl_tax_office_3207","3207","URZĄD SKARBOWY W GRYFINIE"
"pl_tax_office_3208","3208","URZĄD SKARBOWY W KAMIENIU POMORSKIM"
"pl_tax_office_3209","3209","URZĄD SKARBOWY W KOŁOBRZEGU"
"pl_tax_office_3210","3210","PIERWSZY URZĄD SKARBOWY W KOSZALINIE"
"pl_tax_office_3211","3211","DRUGI URZĄD SKARBOWY W KOSZALINIE"
"pl_tax_office_3212","3212","URZĄD SKARBOWY W MYŚLIBORZU"
"pl_tax_office_3213","3213","URZĄD SKARBOWY W PYRZYCACH"
"pl_tax_office_3214","3214","URZĄD SKARBOWY W STARGARDZIE"
"pl_tax_office_3215","3215","PIERWSZY URZĄD SKARBOWY W SZCZECINIE"
"pl_tax_office_3216","3216","DRUGI URZĄD SKARBOWY W SZCZECINIE"
"pl_tax_office_3217","3217","TRZECI URZĄD SKARBOWY W SZCZECINIE"
"pl_tax_office_3218","3218","URZĄD SKARBOWY W SZCZECINKU"
"pl_tax_office_3219","3219","URZĄD SKARBOWY W ŚWINOUJŚCIU"
"pl_tax_office_3220","3220","URZĄD SKARBOWY W WAŁCZU"
"pl_tax_office_3271","3271","ZACHODNIOPOMORSKI URZĄD SKARBOWY W SZCZECINIE"

```

## File: data\res.country.state.csv

```csv
"id","country_id:id","name","code"
state_pl_ds,base.pl,"dolnośląskie","DŚ"
state_pl_kp,base.pl,"kujawsko-pomorskie","KP"
state_pl_lb,base.pl,"lubelskie","LB"
state_pl_ls,base.pl,"lubuskie","LS"
state_pl_ld,base.pl,"łódzkie","ŁD"
state_pl_mp,base.pl,"małopolskie","MP"
state_pl_mz,base.pl,"mazowieckie","MZ"
state_pl_op,base.pl,"opolskie","OP"
state_pl_pk,base.pl,"podkarpackie","PK"
state_pl_pl,base.pl,"podlaskie","PL"
state_pl_pm,base.pl,"pomorskie","PM"
state_pl_sl,base.pl,"śląskie","ŚL"
state_pl_sk,base.pl,"świętokrzyskie","ŚK"
state_pl_wm,base.pl,"warmińsko-mazurskie","WM"
state_pl_wp,base.pl,"wielkopolskie","WP"
state_pl_zp,base.pl,"zachodniopomorskie","ZP"

```

## File: data\template\account.account-pl.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@pl"
"chart01000100","Proprietary land and rights of perpetual usufruct of land","01.000.100","asset_fixed","","False","Grunty własne i prawa wieczystego użytkowania gruntów"
"chart01000200","Buildings, premises and civil engineering structures","01.000.200","asset_fixed","","False","Budynki, lokale i obiekty inżynierii lądowej i wodnej"
"chart01000200","Technical equipment and machinery","01.000.200","asset_fixed","","False","Urządzenia techniczne i maszyny"
"chart01000400","Means of transport","01.000.400","asset_fixed","","False","Środki transportu"
"chart01000900","Other fixed assets","01.000.900","asset_fixed","","False","Inne środki trwałe"
"chart02000100","Costs of completed development","02.000.100","asset_non_current","","False","Koszty zakończonych prac rozwojowych"
"chart02000200","Acquired goodwill","02.000.200","asset_non_current","","False","Nabyta wartość firmy"
"chart02000300","Advances for intangible assets","02.000.300","asset_non_current","","False","Zaliczki na wartości niematerialne i prawne"
"chart02000900","Other intangible assets","02.000.900","asset_non_current","","False","Inne wartości niematerialne i prawne"
"chart03000100","Long-term shares or stocks","03.000.100","asset_non_current","","False","Długoterminowe akcje lub udziały"
"chart03000200","Other long-term securities","03.000.200","asset_non_current","","False","Inne długoterminowe papiery wartościowe"
"chart03000300","Long-term loans granted","03.000.300","asset_non_current","","False","Udzielone pożyczki długoterminowe"
"chart03000400","Other long-term financial assets","03.000.400","asset_non_current","","False","Inne długoterminowe aktywa finansowe"
"chart03000500","Other types of long-term financial assets","03.000.500","asset_non_current","","False","Inne rodzaje długoterminowych aktywów finansowych"
"chart03000600","Real Estate","03.000.600","asset_non_current","","False","Nieruchomości"
"chart03000700","Intangible assets","03.000.700","asset_non_current","","False","Wartości niematerialne i prawne"
"chart03000800","Deferred Tax Assets","03.000.800","asset_non_current","","False","Aktywa z tytułu odroczonego podatku dochodowego"
"chart03000900","Other prepayments","03.000.900","asset_non_current","","False","Inne rozliczenia międzyokresowe"
"chart03050100","Write-downs of shares in foreign entities","03.050.100","asset_non_current","","False","Odpisy aktualizujące udziały i akcje w obcych jednostkach"
"chart03050200","Write-downs on deposits","03.050.200","asset_non_current","","False","Odpisy aktualizujące lokaty"
"chart03050300","Write-downs on long-term loans granted","03.050.300","asset_non_current","","False","Odpisy aktualizujące udzielone pożyczki długoterminowe"
"chart03050900","Write-downs of other types of long-term financial assets","03.050.900","asset_non_current","","False","Odpisy aktualizujące inne rodzaje długoterminowych aktywów finansowych"
"chart07010200","Depreciation of buildings, premises and civil engineering structures","07.010.200","asset_non_current","","False","Odpisy umorzeniowe budynków, lokali i obiektów inżynierii lądowej i wodnej"
"chart07010300","Depreciation of technical equipment and machinery","07.010.300","asset_non_current","","False","Odpisy umorzeniowe urządzeń technicznych i maszyn"
"chart07010400","Depreciation of means of transport","07.010.400","asset_non_current","","False","Odpisy umorzeniowe środków transportu"
"chart07010500","Depreciation of other fixed assets","07.010.500","asset_non_current","","False","Odpisy umorzeniowe innych środków trwałych"
"chart07010600","Depreciation of improvements of third-party fixed assets","07.010.600","asset_non_current","","False","Odpisy umorzeniowe ulepszeń obcych środków trwałych"
"chart07020100","Depreciation of completed development work","07.020.100","asset_non_current","","False","Odpisy umorzeniowe zakończonych prac rozwojowych"
"chart07020200","Depreciation of goodwill","07.020.200","asset_non_current","","False","Odpisy umorzeniowe wartości firmy"
"chart07020300","Depreciation of other intangible assets","07.020.300","asset_non_current","","False","Odpisy umorzeniowe innych wartości niematerialnych i prawnych"
"chart07030100","Depreciation of real estate investments","07.030.100","asset_non_current","","False","Odpisy umorzeniowe inwestycji w nieruchomości"
"chart07030200","Depreciation of investments in intangible assets","07.030.200","asset_non_current","","False","Odpisy umorzeniowe inwestycji w wartości niematerialne i prawne"
"chart08000100","Investments in the construction of a fixed asset","08.000.100","asset_non_current","","False","Inwestycje budowy środka trwałego"
"chart08000200","Improvements to fixed assets","08.000.200","asset_non_current","","False","Ulepszenia środka trwałego"
"chart08000300","Expenditures for the construction of a fixed asset","08.000.300","asset_non_current","","False","Nakłady na budowę środka trwałego"
"chart08000400","Advances for fixed assets under construction","08.000.400","asset_non_current","","False","Zaliczki na środki trwałe w budowie"
"chart08000500","Improvements to third-party fixed assets","08.000.500","asset_non_current","","False","Ulepszenia obcych środków trwałych"
"chart10000100","Domestic cash","10.000.100","asset_current","","False","Kasa krajowych środków pieniężnych"
"chart10000200","Foreign cash","10.000.200","asset_current","","False","Kasa zagranicznych środków pieniężnych"
"chart10000900","Other remaining cash","10.000.900","asset_current","","False","Pozostałe inne środki pieniężne"
"chart13000100","Other cash assets","13.000.100","asset_current","","False","Inne aktywa pieniężne"
"chart13000200","Shares or stocks","13.000.200","asset_current","","False","Udziały lub akcje"
"chart13000900","Other securities","13.000.900","asset_current","","False","Inne papiery wartościowe"
"chart14000100","Short-term loans granted","14.000.100","asset_current","","False","udzielone pożyczki krótkoterminowe"
"chart14000200","Short-term prepayments and accruals","14.000.200","asset_current","","False","Krótkoterminowe rozliczenia międzyokresowe"
"chart14000900","Other short-term financial assets","14.000.900","asset_current","","False","Inne krótkoterminowe aktywa finansowe"
"chart14050100","Write-downs of short-term financial assets","14.050.100","asset_current","","False","Odpisy aktualizujące krótkoterminowe aktywa finansowe"
"chart20000100","Settlements with customers","20.000.100","asset_receivable","","True","Rozrachunki z odbiorcami"
"chart20000200","Settlements with customers (PoS)","20.000.200","asset_receivable","","True","Rozrachunki z odbiorcami (PoS)"
"chart20000300","Advances from customers","20.000.300","liability_current","","False","Zaliczki od klientów"
"chart21000100","Settlements with suppliers","21.000.100","liability_payable","","True","Rozrachunki z dostawcami"
"chart22000100","VAT settlements with the tax authorities","22.000.100","liability_current","","False","Rozrachunki z urzędem skarbowym z tytułu VAT"
"chart22010100","Settlement of input VAT 23%","22.010.100","asset_current","","False","Rozliczenie naliczonego VAT 23%"
"chart22010200","Settlement of input VAT 8%","22.010.200","asset_current","","False","Rozliczenie naliczonego VAT 8%"
"chart22010300","Settlement of input VAT 5%","22.010.300","asset_current","","False","Rozliczenie naliczonego VAT 5%"
"chart22020100","Settlement of due VAT 23%","22.020.100","liability_current","","False","Rozliczenie należnego VAT 23%"
"chart22020200","Settlement of due VAT 8%","22.020.200","liability_current","","False","Rozliczenie należnego VAT 8%"
"chart22020300","Settlement of due VAT 5%","22.020.300","liability_current","","False","Rozliczenie należnego VAT 5%"
"chart22030100","Other public and legal settlements","22.030.100","liability_current","","False","Pozostałe rozrachunki publicznoprawne"
"chart22030200","Other settlements with the tax office","22.030.200","liability_current","","False","Pozostałe rozrachunki z urzędem skarbowym"
"chart22030300","Public law settlements with the city/municipality","22.030.300","liability_current","","False","Rozrachunki publicznoprawne z urzędem miasta/gminy"
"chart22030400","Public and legal settlements with the customs office","22.030.400","liability_current","","False","Rozrachunki publicznoprawne z urzędem celnym"
"chart22030500","Public and legal settlements with Social Security","22.030.500","liability_current","","False","Rozrachunki publicznoprawne z ZUS"
"chart22030600","Public and legal settlements with PFRON","22.030.600","liability_current","","False","Rozrachunki publicznoprawne z PFRON"
"chart23000100","Payroll settlements","23.000.100","liability_current","","False","Rozrachunki z tytułu wynagrodzeń"
"chart23000200","Settlements of loans granted to employees","23.000.200","liability_current","","False","Rozrachunki z tytułu pożyczek udzielonych pracownikom"
"chart23000900","Other settlements with employees","23.000.900","liability_current","","False","Inne rozrachunki z pracownikami"
"chart24010100","Loans received","24.010.100","liability_non_current","","False","Pożyczki otrzymane"
"chart24010200","Loans granted","24.010.200","asset_current","","False","Udzielone pożyczki"
"chart24020100","Settlement of shortages","24.020.100","asset_current","","False","Rozliczenie niedoborów"
"chart24020200","Settlement of surpluses","24.020.200","liability_current","","False","Rozliczenie nadwyżek"
"chart24030100","Settlements for share capital contributions","24.030.100","liability_current","","False","Rozrachunki z tytułu wpłat na kapitał zakładowy"
"chart24030200","Settlements of non-cash contributions to share capital","24.030.200","liability_current","","False","Rozrachunki z tytułu wkładów niepieniężnych na kapitał zakładowy"
"chart24030300","Settlement of capital increase with company's own funds","24.030.300","liability_current","","False","Rozrachunki z tytułu podwyższenia kapitału ze środków własnych spółki"
"chart24030400","Settlements from redemption of own shares","24.030.400","liability_current","","False","Rozrachunki z tytułu umorzenia udziałów własnych"
"chart24050100","Internal settlements","24.050.100","liability_current","","False","Rozrachunki wewnątrzzakładowe"
"chart24090100","Receivables claimed in court","24.090.100","asset_current","","False","Należności dochodzone na drodze sądowej"
"chart24090200","Settlements of dividends","24.090.200","asset_current","","False","Rozrachunki z tytułu dywidend"
"chart24090300","Settlements for subsidies and subsidy refunds","24.090.300","asset_current","","False","Rozrachunki z tytułu dopłat i zwrotu dopłat"
"chart24090900","Other settlements","24.090.900","asset_current","","False","Pozostałe rozrachunki"
"chart28000100","Write-downs of settlements","28.000.100","liability_current","","False","Odpisy aktualizujące rozrachunki"
"chart29000100","Contingent receivables","29.000.100","asset_current","","False","Należności warunkowe"
"chart29010100","Contingent liabilities","29.010.100","liability_current","","False","Zobowiązania warunkowe"
"chart29020100","Foreign bills discounted or endorsed","29.020.100","asset_current","","False","Weksle obce dyskontowane lub indosowane"
"chart30000100","Settlement of purchases of materials","30.000.100","asset_current","","False","Rozliczenie zakupu materiałów"
"chart30000200","Settlement of purchases of goods","30.000.200","asset_current","","False","Rozliczenie zakupu towarów"
"chart30000300","Settlement of purchases of third-party services","30.000.300","asset_current","","False","Rozliczenie zakupu usług obcych"
"chart30000400","Settlement of purchases of fixed assets","30.000.400","asset_current","","False","Rozliczenie zakupu składników aktywów trwałych"
"chart30000500","Settlement of the value of materials and goods in transit","30.000.500","asset_current","","False","Rozliczenie wartości materiałów i towarów w drodze"
"chart30000600","Values of non-invoiced deliveries","30.000.600","asset_current","","False","Wartości dostaw niefakturowanych"
"chart30000700","Handling fees counted by the Customs Office","30.000.700","asset_current","","False","Opłaty manipulacyjne policzone przez Urząd Celny"
"chart30000800","Shortages, damages and surpluses in transportation","30.000.800","asset_current","","False","Niedobory, szkody i nadwyżki w transporcie"
"chart30000900","Claims for supplier invoices","30.000.900","asset_current","","False","Reklamacje faktur dostawców"
"chart31010100","Materials","31.010.100","asset_current","","False","Materiały"
"chart31060100","Packaging","31.060.100","asset_current","","False","Opakowania"
"chart31090100","Materials in processing","31.090.100","asset_current","","False","Materiały w przerobie"
"chart33000100","Wholesale goods","33.000.100","asset_current","","False","Towary w hurcie"
"chart33000200","Goods in retail","33.000.200","asset_current","","False","Towary w detalu"
"chart33000300","Goods in catering establishments","33.000.300","asset_current","","False","Towary w zakładach gastronomicznych"
"chart33000400","Goods purchased","33.000.400","asset_current","","False","Towary skupu"
"chart33000500","Goods outside the entity","33.000.500","asset_current","","False","Towary poza jednostką"
"chart33000600","Real estate and property rights to be traded","33.000.600","asset_current","","False","Nieruchomości i prawa majątkowe przeznaczone do obrotu"
"chart34010100","Deviations from the inventory prices of materials","34.010.100","asset_current","","False","Odchylenia od cen ewidencyjnych materiałów"
"chart34020100","Deviations from the inventory prices of goods in wholesale","34.020.100","asset_current","","False","Odchylenia od cen ewidencyjnych towarów w hurcie"
"chart34020200","Deviations from the inventory prices of goods in retail","34.020.200","asset_current","","False","Odchylenia od cen ewidencyjnych towarów w detalu"
"chart34020300","Deviations from inventory prices of goods in catering establishments","34.020.300","asset_current","","False","Odchylenia od cen ewidencyjnych towarów w zakładach gastronomicznych"
"chart34020400","Deviations from the inventory prices of procurement goods","34.020.400","asset_current","","False","Odchylenia od cen ewidencyjnych towarów skupu"
"chart34060100","Deviations from the inventory prices of packaging","34.060.100","asset_current","","False","Odchylenia od cen ewidencyjnych opakowań"
"chart34070100","Deviations from revaluation of inventories of materials and goods","34.070.100","asset_current","","False","Odchylenia z tytułu aktualizacji wartości zapasów materiałów i towarów"
"chart39000100","Third-party stocks","39.000.100","asset_current","","False","Zapasy obce"
"chart40000100","Depreciation","40.000.100","expense_depreciation","","False","Amortyzacja"
"chart40010100","Consumption of raw materials for the manufacture of products","40.010.100","expense","account.account_tag_operating","False","Zużycie surowców do wytwarzania produktów"
"chart40010200","Energy consumption","40.010.200","expense","","False","Zużycie energii"
"chart40010300","Fuel consumption for means of transport","40.010.300","expense","","False","Zużycie paliwa do środków transportu"
"chart40010400","Consumption of office supplies","40.010.400","expense","","False","Zużycie materiałów biurowych"
"chart40010900","Consumption of other materials","40.010.900","expense","","False","Zużycie innych materiałów"
"chart40020100","Customs services","40.020.100","expense","","False","Usługi celne"
"chart40020200","Telecommunications services","40.020.200","expense","","False","Usługi telekomunikacyjne"
"chart40020300","Postal services","40.020.300","expense","","False","Usługi pocztowe"
"chart40020400","Courier and transportation services","40.020.400","expense","","False","Usługi kurierskie i transportowe"
"chart40020500","Sanitary analyses","40.020.500","expense","","False","Analizy sanitarne"
"chart40020600","Graphic and printing services","40.020.600","expense","","False","Usługi graficzne i drukarskie"
"chart40020700","Repair services","40.020.700","expense","","False","Usługi remontowe"
"chart40020900","Other services","40.020.900","expense","","False","Pozostałe usługi"
"chart40030100","Real estate tax","40.030.100","expense","","False","Podatek od nieruchomości"
"chart40030200","Tax on means of transport","40.030.200","expense","","False","Podatek od środków transportowych"
"chart40030300","Excise tax","40.030.300","expense","","False","Podatek akcyzowy"
"chart40030400","Non-deductible VAT","40.030.400","expense","","False","VAT niepodlegający odliczeniu"
"chart40030500","Bank fees and commissions","40.030.500","expense","","False","Opłaty i prowizje bankowe"
"chart40030600","Court, legal and notary fees","40.030.600","expense","","False","Opłaty sądowe, prawnicze i notarialne"
"chart40030700","Stamp duties","40.030.700","expense","","False","Opłaty skarbowe"
"chart40030800","Concessions","40.030.800","expense","","False","Koncesje"
"chart40030900","Other taxes and fees","40.030.900","expense","","False","Pozostałe podatki i opłaty"
"chart40040100","Salaries of employees","40.040.100","expense","","False","Wynagrodzenia pracowników"
"chart40040200","Salaries of ad hoc employees","40.040.200","expense","","False","Wynagrodzenia osób doraźnie zatrudnionych"
"chart40050100","Contributions to social insurance, FP, FGŚP","40.050.100","expense","","False","Składki na ubezpieczenia społeczne, FP, FGŚP"
"chart40050200","Deductions for company social benefit fund or vacation benefits","40.050.200","expense","","False","Odpisy na zakładowy fundusz świadczeń socjalnych lub świadczenia urlopowe"
"chart40050300","Benefits for employees","40.050.300","expense","","False","Świadczenia na rzecz pracowników"
"chart40050900","Other benefits","40.050.900","expense","","False","Pozostałe świadczenia"
"chart40090100","Other costs by type","40.090.100","expense","","False","Pozostałe koszty rodzajowe"
"chart49000100","Not subject to accrual","49.000.100","expense","","False","Nie podlegające rozliczeniu w czasie"
"chart49000200","Attributable to future periods","49.000.200","expense","","False","Przypadające na przyszłe okresy"
"chart49000300","Accumulated costs","49.000.300","expense","","False","Koszty zgromadzone"
"chart49000400","Costs not included in sales value","49.000.400","expense","","False","Koszty nie wliczane do wartości sprzedaży"
"chart50000100","Settled operating expenses","50.000.100","expense","account.account_tag_operating","False","Rozliczone koszty działalności"
"chart50010100","Costs of uncompleted long-term services","50.010.100","expense","account.account_tag_operating","False","Koszty nie zakończonych długotrwałych usług"
"chart50010200","Losses associated with the performance of long-term services","50.010.200","expense","account.account_tag_operating","False","Straty związane z wykonaniem długotrwałych usług"
"chart50000200","Warehouse maintenance costs","50.000.200","expense","account.account_tag_operating","False","Koszty utrzymania hurtowni"
"chart52010100","Maintenance costs of retail outlets","52.010.100","expense","account.account_tag_operating","False","Koszty utrzymania punktów sprzedaży detalicznej"
"chart52070100","Costs of product sales","52.070.100","expense","account.account_tag_operating","False","Koszty sprzedaży wyrobów"
"chart53000100","Provision of transportation services","53.000.100","expense","account.account_tag_operating","False","Świadczenia usług transportowych"
"chart53000200","Other costs","53.000.200","expense","account.account_tag_operating","False","Pozostałe koszty"
"chart55000100","Unit management costs","55.000.100","expense","account.account_tag_operating","False","Koszty zarządzania jednostką"
"chart55000200","Provision of services for representation and advertising","55.000.200","expense","account.account_tag_operating","False","Świadczenia usług na potrzeby reprezentacji i reklamy"
"chart58000100","Settlement of operating expenses","58.000.100","expense","account.account_tag_operating","False","Rozliczenie kosztów działalności"
"chart60000100","Finished products in stock","60.000.100","asset_current","","False","Produkty gotowe w magazynie"
"chart60010100","Intermediate products","60.010.100","asset_current","","False","Półprodukty"
"chart60020100","Finished products outside the unit","60.020.100","asset_current","","False","Produkty gotowe poza jednostką"
"chart62000100","Deviations from the inventory prices of products","62.000.100","asset_current","","False","Odchylenia od cen ewidencyjnych produktów"
"chart62010100","Deviations from revaluation of product inventories","62.010.100","asset_current","","False","Odchylenia z tytułu aktualizacji wartości zapasów produktów"
"chart64000100","Prepaid expenses","64.000.100","asset_current","","False","Czynne rozliczenia międzyokresowe kosztów"
"chart64010100","Accrued expenses","64.010.100","liability_current","","False","Bierne rozliczenia międzyokresowe kosztów"
"chart65000100","Deferred tax income assets","65.000.100","asset_current","","False","Aktywa z tytułu odroczonego podatku dochodowego"
"chart65010100","Other accruals","65.010.100","liability_current","","False","Pozostałe rozliczenia międzyokresowe"
"chart70000100","Sales of products to the country","70.000.100","income","account.account_tag_operating","False","Sprzedaż produktów na kraj"
"chart70000200","Sales of products for export","70.000.200","income","account.account_tag_operating","False","Sprzedaż produktów na eksport"
"chart70000300","Sale of services to the country","70.000.300","income","account.account_tag_operating","False","Sprzedaż usług na kraj"
"chart70000400","Sale of services for export","70.000.400","income","account.account_tag_operating","False","Sprzedaż usług na eksport"
"chart70010100","Cost of sales of products to the country","70.010.100","expense_direct_cost","account.account_tag_operating","False","Koszt własny sprzedaży produktów na kraj"
"chart70010200","Cost of sales of products for export","70.010.200","expense_direct_cost","account.account_tag_operating","False","Koszt własny sprzedaży produktów na eksport"
"chart70010300","Cost of sales of services to the country","70.010.300","expense_direct_cost","account.account_tag_operating","False","Koszt własny sprzedaży usług na kraj"
"chart70010400","Cost of sales of services for export","70.010.400","expense_direct_cost","account.account_tag_operating","False","Koszt własny sprzedaży usług na eksport"
"chart73000100","Bulk sales of goods","73.000.100","income","account.account_tag_operating","False","Sprzedaż hurtowa towarów"
"chart73000200","Retail sales of goods","73.000.200","income","account.account_tag_operating","False","Sprzedaż detaliczna towarów"
"chart73000300","Sales of goods by mail order","73.000.300","income","account.account_tag_operating","False","Sprzedaż wysyłkowa towarów"
"chart73000400","Commission income","73.000.400","income_other","account.account_tag_operating","False","Dochód z prowizji"
"chart73010100","Value of goods sold at wholesale","73.010.100","expense","account.account_tag_operating","False","Wartość sprzedanych towarów w sprzedaży hurtowej"
"chart73010200","Value of goods sold at retail","73.010.200","expense","account.account_tag_operating","False","Wartość sprzedanych towarów w sprzedaży detalicznej"
"chart73010300","Value of goods sold by mail order","73.010.300","expense","account.account_tag_operating","False","Wartość sprzedanych towarów w sprzedaży wysyłkowej"
"chart73010400","Commission fee","73.010.400","expense","account.account_tag_operating","False","Prowizja komisowa"
"chart74000100","Sales of materials","74.000.100","income","account.account_tag_operating","False","Sprzedaż materiałów"
"chart74000200","Sales of packaging","74.000.200","income","account.account_tag_operating","False","Sprzedaż opakowań"
"chart74000300","Sales of waste","74.000.300","income","account.account_tag_operating","False","Sprzedaż odpadów"
"chart74010100","Value at purchase prices of materials sold","74.010.100","expense","account.account_tag_operating","False","Wartość w cenach zakupu sprzedanych materiałów"
"chart74010200","Value at purchase prices of packages sold","74.010.200","expense","account.account_tag_operating","False","Wartość w cenach zakupu sprzedanych opakowań"
"chart74010300","Value at purchase prices of sold waste","74.010.300","expense","account.account_tag_operating","False","Wartość w cenach zakupu sprzedanych odpadów"
"chart75000100","Amounts due from sale of financial assets","75.000.100","income_other","account.account_tag_investing","False","Kwoty należne ze sprzedaży aktywów finansowych"
"chart75000200","Amounts due from dividends","75.000.200","income_other","account.account_tag_financing","False","Kwoty należne z tytułu dywidend"
"chart75000300","Interest received","75.000.300","income_other","account.account_tag_financing","False","Otrzymane odsetki"
"chart75000400","Income from disposal of investments","75.000.400","income","account.account_tag_investing","False","Przychody ze zbycia inwestycji"
"chart75000500","Revaluation income of investments","75.000.500","income_other","account.account_tag_investing","False","Aktualizacja wartości inwestycji-przychody"
"chart75000600","Positive exchange rate differences","75.000.600","income_other","account.account_tag_financing","False","Dodatnie różnice kursu walut"
"chart75000700","Extraordinary profits","75.000.700","income","","False","Zyski nadzwyczajne"
"chart75000900","Other financial income","75.000.900","income_other","account.account_tag_financing","False","Pozostałe przychody finansowe"
"chart75010100","Value of investments sold","75.010.100","expense","account.account_tag_investing","False","Wartość sprzedanych inwestycji"
"chart75010200","Impairment losses on investments costs","75.010.200","expense","account.account_tag_investing","False","Odpisy z tytułu utraty wartości inwestycjikoszty"
"chart75010300","Interest paid","75.010.300","expense","account.account_tag_financing","False","Odsetki zapłacone"
"chart75010400","Negative exchange rate differences","75.010.400","expense","account.account_tag_financing","False","Ujemne różnice kursu walut"
"chart75010500","Extraordinary losses","75.010.500","expense","","False","Straty nadzwyczajne"
"chart75010900","Other financial costs","75.010.900","expense","account.account_tag_financing","False","Pozostałe koszty finansowe"
"chart76000100","Income from disposal of non-financial fixed assets","76.000.100","income","account.account_tag_investing","False","Przychody ze zbycia niefinansowych aktywów trwałych"
"chart76000200","Grants received","76.000.200","income","account.account_tag_financing","False","Otrzymane dotacje"
"chart76000300","Income from social services","76.000.300","income","account.account_tag_financing","False","Przychody z usług socjalnych"
"chart76000400","Income from increase in value of non-financial fixed assets","76.000.400","income","account.account_tag_investing","False","Przychody ze wzrostu wartości niefinansowych aktywów trwałych"
"chart76000900","Other remaining operating income","76.000.900","income","account.account_tag_operating","False","Inne pozostałe przychody operacyjne"
"chart76010100","Value of sold non-financial fixed assets","76.010.100","expense","account.account_tag_investing","False","Wartość sprzedanych niefinansowych aktywów trwałych"
"chart76010200","Grants provided","76.010.200","expense","account.account_tag_financing","False","Dotacje przekazane"
"chart76010300","Impairment losses on non-financial assets","76.010.300","expense","account.account_tag_investing","False","Odpisy z tytułu utraty wartości aktywów niefinansowych"
"chart76010900","Other remaining operating expenses","76.010.900","expense","account.account_tag_operating","False","Inne pozostałe koszty operacyjne"
"chart79000100","Cost of finished goods issued to own stores","79.000.100","expense","account.account_tag_operating","False","Koszt wyrobów własnej produkcji wydanych do własnych sklepów"
"chart79000200","Cost of benefits for fixed assets under construction","79.000.200","expense","account.account_tag_investing","False","Koszt wytworzenia świadczeń na rzecz środków trwałych w budowie"
"chart79000300","Cost of completed development work","79.000.300","expense","account.account_tag_operating","False","Koszt wytworzenia zakończonych prac rozwojowych"
"chart79000400","Cost of products deemed to be shortages","79.000.400","expense","account.account_tag_operating","False","Koszt wytworzenia produktów uznanych za niedobory"
"chart79000500","Cost of discontinuing a type of activity","79.000.500","expense","account.account_tag_operating","False","Koszt zaniechania określonego rodzaju działalności"
"chart80000100","Share capital","80.000.100","equity","","False","Kapitał podstawowy"
"chart80000200","Payments due to share capital (negative amount)","80.000.200","asset_current","","False","Należne wpłaty na kapitał podstawowy (wielkość ujemna)"
"chart80000300","Own shares (negative amount)","80.000.300","asset_current","","False","Udziały (akcje) własne (wielkość ujemna)"
"chart80000400","Write-downs of net profit during the fiscal year (negative amount)","80.000.400","equity","","False","Odpisy z zysku netto w ciągu roku obrotowego (wielkość ujemna)"
"chart81010100","Spare capital","81.010.100","equity","","False","Kapitał zapasowy"
"chart81020100","Reserve capital","81.020.100","equity","","False","Kapitał rezerwowy"
"chart81030100","Revaluation capital","81.030.100","equity","","False","Kapitał z aktualizacji wyceny"
"chart81040100","Separate capitals (funds) in the statutory unit and establishments (branches) independently preparing the balance sheet","81.040.100","equity","","False","Kapitały wydzielone w jednostce statutowej i zakładach (oddziałach) samodzielnie sporządzających bilans"
"chart82000100","Financial result settlements","82.000.100","equity","","False","Rozliczenia wyniku finansowego"
"chart83000100","Long-term provision for pensions and similar benefits","83.000.100","liability_non_current","","False","Rezerwa długoterminowa na świadczenia emerytalne i podobne"
"chart83000200","Short-term provision for pensions and similar benefits","83.000.200","liability_current","","False","Rezerwa krótkoterminowa na świadczenia emerytalne i podobne"
"chart83010000","Provision for deferred income tax","83.010.000","liability_current","","False","Rezerwa z tytułu odroczonego podatku dochodowego"
"chart83010100","Other long-term provisions","83.010.100","liability_non_current","","False","Pozostałe rezerwy długoterminowe"
"chart83010200","Other short-term provisions","83.010.200","liability_current","","False","Pozostałe rezerwy krótkoterminowe"
"chart84010000","Negative goodwill","84.010.000","liability_non_current","","False","Ujemna wartość firmy"
"chart84020100","Other long-term prepayments","84.020.100","liability_non_current","","False","Inne rozliczenia międzyokresowe długoterminowe"
"chart84020200","Other short-term prepayments","84.020.200","liability_current","","False","Inne rozliczenia międzyokresowe krótkoterminowe"
"chart85010100","Company social benefits fund","85.010.100","liability_current","","False","Zakładowy fundusz świadczeń socjalnych"
"chart85020100","Company fund for rehabilitation of disabled persons","85.020.100","liability_current","","False","Zakładowy fundusz rehabilitacji osób niepełnosprawnych"
"chart85020200","Reward fund","85.020.200","liability_current","","False","Fundusz nagród"
"chart85020300","Fund for renovation of housing resources","85.020.300","liability_current","","False","Fundusz na remont zasobów mieszkaniowych"
"chart86000100","Financial result","86.000.100","equity_unaffected","","False","Wynik finansowy"
"chart87000100","Corporate income tax","87.000.100","expense","account.account_tag_financing","False","Podatek dochodowy od osób prawnych"
"chart87000900","Other mandatory charges to the financial result","87.000.900","expense","account.account_tag_financing","False","Inne obowiązkowe obciążenia wyniku finansowego"

```

## File: data\template\account.fiscal.position-pl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@pl"
"fiscal_position_template_1","1","Domestic","1","1","base.pl","","","","Kraj"
"fiscal_position_template_4","2","Community Private","1","","","base.europe","","","Wspólnota Prywatny"
"fiscal_position_template_2","3","Community","1","1","","base.europe","vs_kraj_23","vs_unia","Wspólnota"
"","","","","","","","vs_kraj_8","vs_unia",""
"","","","","","","","vs_kraj_5","vs_unia",""
"","","","","","","","vs_kraj_0","vs_unia",""
"","","","","","","","vs_kraj_zw","vs_unia",""
"","","","","","","","vs_kraj_usl_23","vs_dostu",""
"","","","","","","","vz_kraj_23","vz_unia",""
"","","","","","","","vz_kraj_8","vz_unia",""
"","","","","","","","vz_kraj_5","vz_unia",""
"","","","","","","","vz_kraj_0","vz_unia",""
"","","","","","","","vz_kraj_zw","vz_unia",""
"","","","","","","","vz_kraj_usl_23","vz_nabu",""
"fiscal_position_template_3","4","Import/Export","1","","","","vs_kraj_23","vs_eksp_tow","Import/Eksport"
"","","","","","","","vs_kraj_8","vs_eksp_tow",""
"","","","","","","","vs_kraj_5","vs_eksp_tow",""
"","","","","","","","vs_kraj_0","vs_eksp_tow",""
"","","","","","","","vs_kraj_zw","vs_eksp_tow",""
"","","","","","","","vs_kraj_usl_23","vs_ekspu",""
"","","","","","","","vz_kraj_23","vz_imp_tow",""
"","","","","","","","vz_kraj_8","vz_imp_tow",""
"","","","","","","","vz_kraj_5","vz_imp_tow",""
"","","","","","","","vz_kraj_0","vz_imp_tow",""
"","","","","","","","vz_kraj_zw","vz_imp_tow",""
"","","","","","","","vz_kraj_usl_23","vz_impu",""

```

## File: data\template\account.group-pl.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@pl"
"pl_group_0","0","","Non-current assets","Aktywa trwałe"
"pl_group_01","01","","Fixed assets","Środki trwałe"
"pl_group_02","02","","Intangible assets","Wartości niematerialne i prawne"
"pl_group_03","03","","Long-term financial assets","Długoterminowe aktywa finansowe"
"pl_group_035","035","","Write-downs of long-term financial assets","Odpisy aktualizujące długoterminowe aktywa finansowe"
"pl_group_07","07","","Write-offs for depreciation and impairment of fixed assets, intangible assets and investments in real estate and rights","Odpisy umorzeniowe oraz z tytułu trwałej utraty wartości środków trwałych, wartości niematerialnych i prawnych oraz inwestycji w nieruchomości i prawa"
"pl_group_071","071","","Depreciation of fixed assets","Odpisy umorzeniowe środków trwałych"
"pl_group_072","072","","Depreciation of intangible assets","Odpisy umorzeniowe wartości niematerialnych i prawnych"
"pl_group_073","073","","Depreciation of investments in real estate and rights","Odpisy umorzeniowe inwestycji w nieruchomości i prawa"
"pl_group_074","074","","Impairment losses of fixed assets","Odpisy z tytułu trwałej utraty wartości środków trwałych"
"pl_group_075","075","","Impairment losses on intangible assets","Odpisy z tytułu trwałej utraty wartości wartości niematerialnych i prawnych"
"pl_group_076","076","","Impairment losses on investments in real estate and rights","Odpisy z tytułu trwałej utraty wartości inwestycji w nieruchomości i prawa"
"pl_group_08","08","","Fixed assets under construction","Środki trwałe w budowie"
"pl_group_09","09","","Off-balance accounts in team 0","Konta pozabilansowe w zespole 0"
"pl_group_090","090","","Third-party fixed assets","Obce środki trwałe"
"pl_group_091","091","","Fixed assets in liquidation","Środki trwałe w likwidacji"
"pl_group_1","1","","Cash, bank accounts and other short-term financial assets","Środki pieniężne, rachunki bankowe oraz inne krótkoterminowe aktywa finansowe"
"pl_group_10","10","","Cash","Kasa"
"pl_group_13","13","","Bank accounts and loans","Rachunki i kredyty bankowe"
"pl_group_130","130","","Current account","Rachunek bieżący"
"pl_group_131","131","","Foreign currency account","Rachunek walutowy"
"pl_group_134","134","","Bank loans","Kredyty bankowe"
"pl_group_135","135","","Other bank accounts","Inne rachunki bankowe"
"pl_group_139","139","","Cash on the move","Środki pieniężne w drodze"
"pl_group_14","14","","Short-term financial assets","Krótkoterminowe aktywa finansowe"
"pl_group_145","145","","Write-downs of short-term financial assets","Odpisy aktualizujące krótkoterminowe aktywa finansowe"
"pl_group_2","2","","Settlements and claims","Rozrachunki i roszczenia"
"pl_group_20","20","","Settlements with customers","Rozliczenia z klientami"
"pl_group_21","21","","Settlements with suppliers","Rozrachunki z dostawcami"
"pl_group_22","22","","Public and legal settlements","Rozrachunki publicznoprawne"
"pl_group_220","220","","VAT settlements with the tax office","Rozrachunki z urzędem skarbowym z tytułu VAT"
"pl_group_221","221","","Input VAT and its settlement","VAT naliczony i jego rozliczenie"
"pl_group_222","222","","Settlements with the tax office for output VAT","Rozrachunki z urzędem skarbowym z tytułu VAT należnego"
"pl_group_223","223","","Other public and legal settlements","Pozostałe rozrachunki publicznoprawne"
"pl_group_23","23","","Settlements of salaries and other benefits for employees","Rozrachunki z tytułu wynagrodzeń i innych świadczeń na rzecz pracowników"
"pl_group_230","230","","Payroll settlements","Rozrachunki z tytułu wynagrodzeń"
"pl_group_234","234","","Other settlements with employees","Pozostałe rozrachunki z pracownikami"
"pl_group_24","24","","Other settlements","Pozostałe rozrachunki"
"pl_group_240","240","","Loans","Pożyczki"
"pl_group_241","241","","Accounting for shortages, damages and surpluses","Rozliczenie niedoborów, szkód i nadwyżek"
"pl_group_242","242","","Settlements with owners (partners of partnerships)","Rozrachunki z właścicielami (wspólnikami spółek osobowych)"
"pl_group_243","243","","Settlements with shareholders (stockholders)","Rozrachunki z udziałowcami (akcjonariuszami)"
"pl_group_245","245","","Intra-company settlements","Rozrachunki wewnątrzzakładowe"
"pl_group_246","246","","Receivables claimed in court","Należności dochodzone na drodze sądowej"
"pl_group_249","249","","Other settlements - other","Pozostałe rozrachunki - inne"
"pl_group_28","28","","Impairment losses on receivables","Odpisy aktualizujące wartość należności"
"pl_group_29","29","","Off-balance sheet accounts in team 2","Konta pozabilansowe w zespole 2"
"pl_group_290","290","","Contingent receivables","Należności warunkowe"
"pl_group_291","291","","Contingent liabilities","Zobowiązania warunkowe"
"pl_group_292","292","","Foreign bills discounted or endorsed","Weksle obce dyskontowane lub indosowane"
"pl_group_3","3","","Materials and goods","Materiały i towary"
"pl_group_30","30","","Purchase settlement","Rozliczenie zakupu"
"pl_group_31","31","","Materials and packaging","Materiały i opakowania"
"pl_group_311","311","","Materials","Materiały"
"pl_group_316","316","","Packaging","Opakowania"
"pl_group_319","319","","Materials in processing","Materiały w przerobie"
"pl_group_33","33","","Goods","Towary"
"pl_group_34","34","","Deviations from register prices","Odchylenia od cen ewidencyjnych"
"pl_group_341","341","","Deviations from the recording prices of materials","Odchylenia od cen ewidencyjnych materiałów"
"pl_group_342","342","","Deviations from the registered prices of goods","Odchylenia od cen ewidencyjnych towarów"
"pl_group_346","346","","Deviations from the inventory prices of packaging","Odchylenia od cen ewidencyjnych opakowań"
"pl_group_347","347","","Deviations from revaluation of inventories of materials and goods","Odchylenia z tytułu aktualizacji wartości zapasów materiałów i towarów"
"pl_group_39","39","","Off-balance sheet accounts in team 3","Konta pozabilansowe w zespole 3"
"pl_group_390","390","","Third-party stocks","Zapasy obce"
"pl_group_4","4","","Costs by type and their settlement","Koszty według rodzajów i ich rozliczenie"
"pl_group_40","40","","Costs by type","Koszty według rodzajów"
"pl_group_400","400","","Depreciation","Amortyzacja"
"pl_group_401","401","","Consumption of materials and energy","Zużycie materiałów i energii"
"pl_group_402","402","","Third-party services","Usługi obce"
"pl_group_403","403","","Taxes and fees","Podatki i opłaty"
"pl_group_404","404","","Salaries","Wynagrodzenia"
"pl_group_405","405","","Social security and other benefits","Ubezpieczenia społeczne i inne świadczenia"
"pl_group_409","409","","Other costs by type","Pozostałe koszty rodzajowe"
"pl_group_49","49","","Settlement of costs","Rozliczenie kosztów"
"pl_group_5","5","","Costs by type of activity and their settlement","Koszty według typów działalności i ich rozliczenie"
"pl_group_50","50","","Costs of primary activities - production","Koszty działalności podstawowej - produkcyjnej"
"pl_group_51","51","","Costs of primary activities - commercial","Koszty działalności podstawowej - handlowej"
"pl_group_52","52","","Departmental costs and cost of sales","Koszty wydziałowe i koszty sprzedaży"
"pl_group_521","521","","Department costs","Koszty wydziałowe"
"pl_group_527","527","","Cost of sales","Koszty sprzedaży"
"pl_group_53","53","","Costs of auxiliary activities","Koszty działalności pomocniczej"
"pl_group_55","55","","Management costs","Koszty zarządu"
"pl_group_58","58","","Settlement of operating expenses","Rozliczenie kosztów działalności"
"pl_group_6","6","","Products and accruals","Produkty i rozliczenia międzyokresowe"
"pl_group_60","60","","Finished and semi-finished products","Produkty gotowe i półprodukty"
"pl_group_600","600","","Finished products","Produkty gotowe"
"pl_group_601","601","","Semi-finished products","Półprodukty"
"pl_group_602","602","","Products in progress","Produkty w toku"
"pl_group_62","62","","Deviations from the inventory prices of products and semi-finished products","Odchylenia od cen ewidencyjnych produktów i półproduktów"
"pl_group_620","620","","Deviations from the inventory prices of products","Odchylenia od cen ewidencyjnych produktów"
"pl_group_621","621","","Deviations from revaluation of inventory of products and semi-finished products","Odchylenia z tytułu aktualizacji wartości zapasów produktów i półproduktów"
"pl_group_64","64","","Deferred expenses","Rozliczenia międzyokresowe kosztów"
"pl_group_640","640","","Prepaid expenses","Czynne rozliczenia międzyokresowe kosztów"
"pl_group_641","641","","Accrued expenses","Bierne rozliczenia międzyokresowe kosztów"
"pl_group_65","65","","Other accruals","Pozostałe rozliczenia międzyokresowe"
"pl_group_650","650","","Deferred tax assets","Aktywa z tytułu odroczonego podatku dochodowego"
"pl_group_651","651","","Other accruals and deferred income","Inne rozliczenia międzyokresowe"
"pl_group_67","67","","Livestock and deviations from the registered prices","Inwentarz żywy oraz odchylenia od cen ewidencyjnych"
"pl_group_670","670","","Livestock","Inwentarz żywy"
"pl_group_671","671","","Deviations from livestock registration prices","Odchylenia od cen ewidencyjnych inwentarza żywego"
"pl_group_7","7","","Revenues and related costs","Przychody i koszty związane z ich osiągnięciem"
"pl_group_70","70","","Revenues and expenses related to product sales","Przychody i koszty związane ze sprzedażą produktów"
"pl_group_700","700","","Product sales","Sprzedaż produktów"
"pl_group_701","701","","Cost of products sold","Koszt sprzedanych produktów"
"pl_group_73","73","","Revenues and expenses related to the sale of goods","Przychody i koszty związane ze sprzedażą towarów"
"pl_group_730","730","","Sale of goods","Sprzedaż towarów"
"pl_group_731","731","","Value of goods sold at purchase (acquisition) prices","Wartość sprzedanych towarów w cenach zakupu (nabycia)"
"pl_group_74","74","","Revenues and expenses related to the sale of materials and packaging","Przychody i koszty związane ze sprzedażą materiałów i opakowań"
"pl_group_740","740","","Sales of materials and packaging","Sprzedaż materiałów i opakowań"
"pl_group_741","741","","Value of materials and packaging sold","Wartość sprzedanych materiałów i opakowań"
"pl_group_75","75","","Financial income and expenses","Przychody i koszty finansowe"
"pl_group_750","750","","Financial income","Przychody finansowe"
"pl_group_751","751","","Financial costs","Koszty finansowe"
"pl_group_76","76","","Other operating income and expenses","Pozostałe przychody i koszty operacyjne"
"pl_group_760","760","","Other operating income","Pozostałe przychody operacyjne"
"pl_group_761","761","","Other operating expenses","Pozostałe koszty operacyjne"
"pl_group_79","79","","Internal turnover and internal turnover costs","Obroty wewnętrzne i koszty obrotów wewnętrznych"
"pl_group_790","790","","Internal turnover","Obroty wewnętrzne"
"pl_group_791","791","","Internal turnover costs","Koszty obrotów wewnętrznych"
"pl_group_8","8","","Equity (funds), special funds and financial result","Kapitały (fundusze) własne, fundusze specjalne i wynik finansowy"
"pl_group_80","80","","Primary capital (fund)","Kapitał (fundusz) podstawowy"
"pl_group_81","81","","Other equity (funds)","Pozostałe kapitały (fundusze) własne"
"pl_group_811","811","","Supplementary capital (fund)","Kapitał (fundusz) zapasowy"
"pl_group_812","812","","Reserve capital (fund)","Kapitał (fundusz) rezerwowy"
"pl_group_813","813","","Revaluation reserve (fund)","Kapitał (fundusz) z aktualizacji wyceny"
"pl_group_814","814","","Divisional capitals (funds)","Kapitały (fundusze) wydziałowe"
"pl_group_82","82","","Settlement of financial result","Rozliczenie wyniku finansowego"
"pl_group_83","83","","Reserves","Rezerwy"
"pl_group_830","830","","Deferred income tax provision","Rezerwa z tytułu odroczonego podatku dochodowego"
"pl_group_831","831","","Other reserves","Pozostałe rezerwy"
"pl_group_84","84","","Deferred income","Rozliczenia międzyokresowe przychodów"
"pl_group_85","85","","Special funds","Fundusze specjalne"
"pl_group_850","850","","Company social benefits fund","Zakładowy fundusz świadczeń socjalnych"
"pl_group_851","851","","Other special funds","Inne fundusze specjalne"
"pl_group_86","86","","Financial result","Wynik finansowy"
"pl_group_87","87","","Corporate income tax","Podatek dochodowy od osób prawnych"

```

## File: data\template\account.tax-pl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","sequence","tax_group_id","tax_scope","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@pl"
"vs_kraj_23","23% G","","23%","23.0","percent","sale","0","tax_group_vat_23","","","base","invoice","+K_19","","","23% T"
"","","","","","","","","","","","tax","invoice","+K_20","chart22020100","",""
"","","","","","","","","","","","base","refund","-K_19","","",""
"","","","","","","","","","","","tax","refund","-K_20","chart22020100","",""
"vs_kraj_8","8%","","8%","8.0","percent","sale","","tax_group_vat_8","","","base","invoice","+K_17","","",""
"","","","","","","","","","","","tax","invoice","+K_18","chart22020200","",""
"","","","","","","","","","","","base","refund","-K_17","","",""
"","","","","","","","","","","","tax","refund","-K_18","chart22020200","",""
"vs_kraj_5","5%","","5%","5.0","percent","sale","","tax_group_vat_5","","","base","invoice","+K_15","","",""
"","","","","","","","","","","","tax","invoice","+K_16","chart22020300","",""
"","","","","","","","","","","","base","refund","-K_15","","",""
"","","","","","","","","","","","tax","refund","-K_16","chart22020300","",""
"vs_kraj_0","0%","","0%","0.0","percent","sale","","tax_group_vat_0","","","base","invoice","+K_13","","",""
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_13","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vs_zwrot_podróżnych_0","0% Travel","","0%","0.0","percent","sale","","tax_group_vat_0","","","base","invoice","+K_14","","","0% Podróż"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_14","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vs_kraj_zw","0% Exempt","","0%","0.0","percent","sale","","tax_group_vat_0","","","base","invoice","+K_10","","","0% Zwolnione"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_10","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vs_kraj_zw_gold","0% Exempt Gold","","0%","0.0","percent","sale","","tax_group_vat_0","","","base","invoice","+K_10||Gold","","","0% Zwolnione Złota"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_10||Gold","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vs_kraj_usl_23","23% S","","23%","23.0","percent","sale","","tax_group_vat_23","service","","base","invoice","+K_19","","","23% U"
"","","","","","","","","","","","tax","invoice","+K_20","chart22020100","",""
"","","","","","","","","","","","base","refund","-K_19","","",""
"","","","","","","","","","","","tax","refund","-K_20","chart22020100","",""
"vz_kraj_23","23% G","","23%","23.0","percent","purchase","0","tax_group_vat_23","","","base","invoice","+K_42","","","23% T"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","",""
"vz_kraj_8","8%","","8%","8.0","percent","purchase","","tax_group_vat_8","","","base","invoice","+K_42","","",""
"","","","","","","","","","","","tax","invoice","+K_43","chart22010200","",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010200","",""
"vz_kraj_5","5%","","5%","5.0","percent","purchase","","tax_group_vat_5","","","base","invoice","+K_42","","",""
"","","","","","","","","","","","tax","invoice","+K_43","chart22010300","",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010300","",""
"vz_kraj_0","0%","","0%","0.0","percent","purchase","","tax_group_vat_0","","","base","invoice","+K_42","","",""
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_kraj_zw","0% Exempt","","0%","0.0","percent","purchase","","tax_group_vat_0","","","base","invoice","+K_42","","","0% Zwolnione"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_kraj_usl_23","23% S","","23%","23.0","percent","purchase","","tax_group_vat_23","service","","base","invoice","+K_42","","","23% U"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","",""
"vp_leas_sale","23% - vehicle leasing","","23%","23.0","percent","sale","1","tax_group_vat_23","","","base","invoice","+K_42","","","23% - leasing pojazdu"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","60",""
"","","","","","","","","","","","tax","invoice","","chart76010900","40",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","60",""
"","","","","","","","","","","","tax","refund","","chart76010900","40",""
"vp_leas_purchase","23% - vehicle leasing","","23%","23.0","percent","purchase","1","tax_group_vat_23","","","base","invoice","+K_42","","","23% - leasing pojazdu"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","60",""
"","","","","","","","","","","","tax","invoice","","chart76010900","40",""
"","","","","","","","","","","","base","refund","-K_42","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","60",""
"","","","","","","","","","","","tax","refund","","chart76010900","40",""
"vs_stal","0% Steel","","0%","0.0","percent","sale","","tax_group_vat_0","consu","False","base","invoice","+K_31","","","0% Stal"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_31","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_stal","23% Steel","","23%","23.0","percent","purchase","","tax_group_vat_0","consu","False","base","invoice","+K_42||+K_31","","","23% Stal"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","",""
"","","","","","","","","","","","tax","invoice","-K_32","chart22020100","-100",""
"","","","","","","","","","","","base","refund","-K_42||-K_31","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","",""
"","","","","","","","","","","","tax","refund","+K_32","chart22020100","-100",""
"vs_unia","0% EU G","","0%","0.0","percent","sale","","tax_group_vat_0","consu","","base","invoice","+K_21","","","0% EU T"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_21","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vs_unia_i42","0% EU I_42","0% I_42","0%","0.0","percent","sale","","tax_group_vat_0","consu","False","base","invoice","+K_21||+I_42","","","0% EU I_42"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_21||-I_42","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vs_unia_triangular","0% EU T 2","0% EU T 2nd Payer","0%","0.0","percent","sale","","tax_group_vat_0","consu","False","base","invoice","+K_21||+Triangular Sale","","","0% EU T 2"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_21||-Triangular Sale","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vs_unia_i63","0% EU I_63","0% I_63","0%","0.0","percent","sale","","tax_group_vat_0","consu","False","base","invoice","+K_21||+I_63","","","0% EU I_63"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_21||-I_63","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_unia_triangular","0% EU T 2","0% EU T 2nd Payer","0%","0.0","percent","purchase","","tax_group_vat_0","consu","False","base","invoice","+K_11||+Triangular Purchase","","","0% EU T 2"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_11||-Triangular Purchase","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_unia","23% EU G","","23%","23.0","percent","purchase","","tax_group_vat_0","consu","","base","invoice","+K_42||+K_23","","","23% EU T"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","",""
"","","","","","","","","","","","tax","invoice","-K_24","chart22020100","-100",""
"","","","","","","","","","","","base","refund","-K_42||-K_23","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","",""
"","","","","","","","","","","","tax","refund","+K_24","chart22020100","-100",""
"vs_dostu","0% EU S","","0%","0.0","percent","sale","","tax_group_vat_0","service","","base","invoice","+K_11||+K_12","","","0% EU U"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_11||-K_12","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_nabu","23% EU S","","23%","23.0","percent","purchase","","tax_group_vat_0","service","","base","invoice","+K_42||+K_29","","","23% EU S"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","",""
"","","","","","","","","","","","tax","invoice","-K_30","chart22020100","-100",""
"","","","","","","","","","","","base","refund","-K_42||-K_29","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","",""
"","","","","","","","","","","","tax","refund","+K_30","chart22020100","-100",""
"vs_eksp_tow","0% EX G","","0%","0.0","percent","sale","","tax_group_vat_0","consu","","base","invoice","+K_22","","","0% EX T"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_22","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_imp_tow","23% EX G","","23%","23.0","percent","purchase","","tax_group_vat_0","consu","","base","invoice","+K_42||+K_25","","","23% EX T"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","",""
"","","","","","","","","","","","tax","invoice","-K_26","chart22020100","-100",""
"","","","","","","","","","","","base","refund","-K_42||-K_25","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","",""
"","","","","","","","","","","","tax","refund","+K_26","chart22020100","-100",""
"vs_ekspu","0% EX S","","0%","0.0","percent","sale","","tax_group_vat_0","service","","base","invoice","+K_11","","","0% EX U"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-K_11","","",""
"","","","","","","","","","","","tax","refund","","","",""
"vz_impu","23% EX S","","23%","23.0","percent","purchase","","tax_group_vat_0","service","","base","invoice","+K_42||+K_27","","","23% EX U"
"","","","","","","","","","","","tax","invoice","+K_43","chart22010100","",""
"","","","","","","","","","","","tax","invoice","-K_28","chart22020100","-100",""
"","","","","","","","","","","","base","refund","-K_42||-K_27","","",""
"","","","","","","","","","","","tax","refund","-K_43","chart22010100","",""
"","","","","","","","","","","","tax","refund","+K_28","chart22020100","-100",""

```

## File: data\template\account.tax.group-pl.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"tax_group_vat_23","VAT 23%","base.pl","chart22000100","chart22000100"
"tax_group_vat_8","VAT 8%","base.pl","chart22000100","chart22000100"
"tax_group_vat_5","VAT 5%","base.pl","chart22000100","chart22000100"
"tax_group_vat_0","VAT 0%","base.pl","chart22000100","chart22000100"

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
from openerp.modules.registry import RegistryManager

def migrate(cr, version):
    registry = RegistryManager.get(cr.dbname)
    from openerp.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_pl')

```

## File: models\account_move.py

```python
from odoo import fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_pl_vat_b_spv = fields.Boolean(
        string='B_SPV',
        help="Transfer of a single-purpose voucher effected by a taxable person acting on his/its own behalf",
    )
    l10n_pl_vat_b_spv_dostawa = fields.Boolean(
        string='B_SPV_Dostawa',
        help="Supply of goods and/or services covered by a single-purpose voucher to a taxpayer",
    )
    l10n_pl_vat_b_mpv_prowizja = fields.Boolean(
        string='B_MPV_Prowizja',
        help="Supply of agency and other services pertaining to the transfer of a single-purpose voucher",
    )

```

## File: models\l10n_pl_tax_office.py

```python
from odoo import api, fields, models


class TaxOffice(models.Model):
    _name = 'l10n_pl.l10n_pl_tax_office'
    _description = 'Tax Office in Poland'
    _rec_names_search = ['name', 'code']
    _order = 'code'

    code = fields.Char('Code', required=True)
    name = fields.Char('Description', required=True)

    _sql_constraints = [
        ('code_company_uniq', 'unique (code)', 'The code of the tax office must be unique !')
    ]

    @api.depends('name', 'code')
    def _compute_display_name(self):
        for tax_office in self:
            tax_office.display_name = f'{tax_office.code} {tax_office.name}'

```

## File: models\product.py

```python
from odoo import models, fields


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    l10n_pl_vat_gtu = fields.Selection(
        string='GTU Codes',
        selection=[
            ('GTU_01', 'GTU_01 - Alcoholic beverages'),
            ('GTU_02', 'GTU_02 - Goods referred to under Art. 103 sec 5aa'),
            ('GTU_03', 'GTU_03 - Fuel oil for excise duty, lubricating oils and other oils'),
            ('GTU_04', 'GTU_04 - Tobacco products, tobacco, e-liquid'),
            ('GTU_05', 'GTU_05 - Wastes'),
            ('GTU_06', 'GTU_06 - Electronic devices, their parts and materials'),
            ('GTU_07', 'GTU_07 - Vehicles and vehicle parts'),
            ('GTU_08', 'GTU_08 - Precious metals and base metals'),
            ('GTU_09', 'GTU_09 - Medicament and medical devices, medicinal products'),
            ('GTU_10', 'GTU_10 - Buildings, structures and land'),
            ('GTU_11', 'GTU_11 - Services related to the greenhouse gas emission allowance trading'),
            ('GTU_12', 'GTU_12 - Intangible services'),
            ('GTU_13', 'GTU_13 - Transport services and warehouse management services'),
        ],
        help='Codes for specific types of products, needed for VAT declaration'
    )

```

## File: models\res_company.py

```python
from odoo import fields, models


class Company(models.Model):
    _inherit = 'res.company'

    l10n_pl_reports_tax_office_id = fields.Many2one('l10n_pl.l10n_pl_tax_office', string='Tax Office', groups="account.group_account_user")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_pl_reports_tax_office_id = fields.Many2one(related='company_id.l10n_pl_reports_tax_office_id', readonly=False)

```

## File: models\res_partner.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    """Inherited to add the information needed for the JPK"""
    _inherit = 'res.partner'

    l10n_pl_links_with_customer = fields.Boolean(
        string='Links With Company',
        help='TP: Existing connection or influence between the customer and the supplier'
    )

```

## File: models\template_pl.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('pl')
    def _get_pl_template_data(self):
        return {
            'property_account_receivable_id': 'chart20000100',
            'property_account_payable_id': 'chart21000100',
            'property_account_expense_categ_id': 'chart70010100',
            'property_account_income_categ_id': 'chart73000100',
            'code_digits': '8',
            'use_storno_accounting': True,
        }

    @template('pl', 'res.company')
    def _get_pl_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.pl',
                'bank_account_code_prefix': '11.000.00',
                'cash_account_code_prefix': '12.000.00',
                'transfer_account_code_prefix': '11.090.00',
                'account_default_pos_receivable_account_id': 'chart20000200',
                'income_currency_exchange_account_id': 'chart75000600',
                'expense_currency_exchange_account_id': 'chart75010400',
                'account_journal_early_pay_discount_loss_account_id': 'chart75010900',
                'account_journal_early_pay_discount_gain_account_id': 'chart75000900',
                'default_cash_difference_income_account_id': 'chart75000700',
                'default_cash_difference_expense_account_id': 'chart75010500',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import product
from . import account_move
from . import res_partner
from . import res_company
from . import res_config_settings
from . import l10n_pl_tax_office
from . import template_pl

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_pl_tax_office,l10n_pl.l10n_pl_tax_office,model_l10n_pl_l10n_pl_tax_office,account.group_account_user,1,1,1,0

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_move_form_l10n_pl" model="ir.ui.view">
        <field name="name">account.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <notebook position="inside">
                <page id="pl_extra" string="PL Extra" name="page_pl_extra" invisible="move_type not in ('out_invoice', 'out_refund') or country_code != 'PL'">
                    <group>
                        <group>
                            <field name="l10n_pl_vat_b_spv_dostawa" readonly="state != 'draft'"/>
                            <field name="l10n_pl_vat_b_mpv_prowizja" readonly="state != 'draft'"/>
                        </group>
                        <group>
                            <field name="l10n_pl_vat_b_spv" readonly="state != 'draft'"/>
                        </group>
                    </group>
                </page>
            </notebook>
        </field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="product_template_form" model="ir.ui.view">
        <field name="name">product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='invoicing']//group[@name='accounting']" position="inside">
                <group name="GTU" string="GTU">
                    <field name="l10n_pl_vat_gtu"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n_pl</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='invoicing_settings']" position="after">
                <block title="Polish Localization" invisible="country_code != 'PL'">
                    <div id="l10n_pl_section" class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <div class="content-group">
                                    <div class="row mt16">
                                        <span>
                                            <label for="l10n_pl_reports_tax_office_id"/>
                                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                            <field name="l10n_pl_reports_tax_office_id"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </block>
            </xpath>
        </field>
    </record>
</odoo>
```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record  model="ir.ui.view" id="res_partner_account_pl_form">
            <field name="name">res.partner.account.pl.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='accounting_entries']" position="after">
                    <group string="PL Information">
                        <field name="l10n_pl_links_with_customer"/>
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

