# Odoo Module: l10n_kz

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Kazakhstan - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['kz'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This provides a base chart of accounts and taxes template for use in Odoo for Kazakhstan.
    """,
    'depends': [
        'account',
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
<odoo>
    <record id="l10n_kz_tr_form_300_00" model="account.report">
        <field name="name">VAT Report - Form 300.00</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.kz"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_kz_tr_column_net" model="account.report.column">
                <field name="name">Sum of sales turnover without VAT</field>
                <field name="expression_label">net</field>
            </record>
            <record id="l10n_kz_tr_column_tax" model="account.report.column">
                <field name="name">Sum of VAT</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_kz_tr_title_vat_calculation" model="account.report.line">
                <field name="name">VAT accrual</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_kz_tr_line_300_00_001" model="account.report.line">
                        <field name="name">300.00.001. Sales turnover subject to VAT, including</field>
                        <field name="code">L10N_KZ_300_00_001</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_001_1" model="account.report.line">
                                <field name="name">I. With the issuance of invoices</field>
                                <field name="code">L10N_KZ_300_00_001_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_001_1_net" model="account.report.expression">
                                        <field name="label">net</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I. With the issuance of invoices net</field>
                                    </record>
                                    <record id="l10n_kz_tr_expression_300_00_001_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I. With the issuance of invoices tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_001_2" model="account.report.line">
                                <field name="name">II. Without the issuance of invoices</field>
                                <field name="code">L10N_KZ_300_00_001_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_001_2_net" model="account.report.expression">
                                        <field name="label">net</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                    <record id="l10n_kz_tr_expression_300_00_001_2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_001_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_001_1.net + L10N_KZ_300_00_001_2.net</field>
                            </record>
                            <record id="l10n_kz_tr_expression_300_00_001_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_001_1.tax + L10N_KZ_300_00_001_2.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_002" model="account.report.line">
                        <field name="name">300.00.002. Sales turnover subject to zero VAT</field>
                        <field name="code">L10N_KZ_300_00_002</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_002_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales turnover subject to zero VAT net</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_003" model="account.report.line">
                        <field name="name">300.00.003. Adjustment of taxable turnover</field>
                        <field name="code">L10N_KZ_300_00_003</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_003_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                            <record id="l10n_kz_tr_expression_300_00_003_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_004" model="account.report.line">
                        <field name="name">300.00.004. Turnover on sale of goods, works, services, the place of sale of which is not the Republic of Kazakhstan</field>
                        <field name="code">L10N_KZ_300_00_004</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_004_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Turnover - not in the Republic of Kazakhstan net</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_005" model="account.report.line">
                        <field name="name">300.00.005. Turnover exempt from VAT</field>
                        <field name="code">L10N_KZ_300_00_005</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_005_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Turnover exempt from VAT net</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_006" model="account.report.line">
                        <field name="name">300.00.006. Total turnover</field>
                        <field name="code">L10N_KZ_300_00_006</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_006_1" model="account.report.line">
                                <field name="name">I. Amount of taxable turnover</field>
                                <field name="code">L10N_KZ_300_00_006_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_006_1_net" model="account.report.expression">
                                        <field name="label">net</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">L10N_KZ_300_00_001.net + L10N_KZ_300_00_002.net + L10N_KZ_300_00_003.net</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_006_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_006_1.net + L10N_KZ_300_00_004.net + L10N_KZ_300_00_005.net</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_007" model="account.report.line">
                        <field name="name">300.00.007. Share of taxable turnover in total turnover</field>
                        <field name="code">L10N_KZ_300_00_007</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_007_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_006_1.net / L10N_KZ_300_00_006.net * 100</field>
                                <field name="figure_type">percentage</field>
                                <field name="subformula">ignore_zero_division</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_008" model="account.report.line">
                        <field name="name">300.00.008. Share of taxable turnover at zero rate in the total taxable turnover</field>
                        <field name="code">L10N_KZ_300_00_008</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_008_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">(L10N_KZ_300_00_002.net / (L10N_KZ_300_00_001.net + L10N_KZ_300_00_002.net + L10N_KZ_300_00_003.net)) * 100</field>
                                <field name="subformula">ignore_zero_division</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_009" model="account.report.line">
                        <field name="name">300.00.009. Share of taxable turnover in total turnover, calculated without taking into account turnovers, for which the proportional with the right to separate accounting for individual turnovers is applied</field>
                        <field name="code">L10N_KZ_300_00_009</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_009_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_010" model="account.report.line">
                        <field name="name">300.00.010. VAT accrued on import of goods by offset method in accordance with the terms of the subsoil use contract</field>
                        <field name="code">L10N_KZ_300_00_010</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_010_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_011" model="account.report.line">
                        <field name="name">300.00.011. VAT accrued on import of goods by offset method, except for line 300.00.010 (300.04.001 B)</field>
                        <field name="code">L10N_KZ_300_00_011</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_011_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_012" model="account.report.line">
                        <field name="name">300.00.012. Total VAT accrued</field>
                        <field name="code">L10N_KZ_300_00_012</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_012_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_001.tax + L10N_KZ_300_00_003.tax + L10N_KZ_300_00_010.tax + L10N_KZ_300_00_011.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_kz_tr_title_vat_sum" model="account.report.line">
                <field name="name">The amount of VAT to be credited</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_kz_tr_line_300_00_013" model="account.report.line">
                        <field name="name">300.00.013. Goods, works, services purchased with VAT in the Republic of Kazakhstan, including:</field>
                        <field name="code">L10N_KZ_300_00_013</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_013_1" model="account.report.line">
                                <field name="name">I. On invoices</field>
                                <field name="code">L10N_KZ_300_00_013_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_013_1_net" model="account.report.expression">
                                        <field name="label">net</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">On invoices net</field>
                                    </record>
                                    <record id="l10n_kz_tr_expression_300_00_013_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">On invoices tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_013_2" model="account.report.line">
                                <field name="name">II. On other documents</field>
                                <field name="code">L10N_KZ_300_00_013_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_013_2_net" model="account.report.expression">
                                        <field name="label">net</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                    <record id="l10n_kz_tr_expression_300_00_013_2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_013_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_013_1.net + L10N_KZ_300_00_013_2.net</field>
                            </record>
                            <record id="l10n_kz_tr_expression_300_00_013_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_013_1.tax + L10N_KZ_300_00_013_2.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_014" model="account.report.line">
                        <field name="name">300.00.014. Works, services purchased from a non-resident</field>
                        <field name="code">L10N_KZ_300_00_014</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_014_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Works, services purchased from a non-resident net</field>
                            </record>
                            <record id="l10n_kz_tr_expression_300_00_014_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Works, services purchased from a non-resident tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_015" model="account.report.line">
                        <field name="name">300.00.015. Goods, works, services purchased without VAT and for which set-off is not allowed</field>
                        <field name="code">L10N_KZ_300_00_015</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_015_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases without VAT net</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_016" model="account.report.line">
                        <field name="name">300.00.016. Imports with payment of VAT (on the basis of the declaration of goods, on the basis of the declaration form 320.00), including:</field>
                        <field name="code">L10N_KZ_300_00_016</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_016_1" model="account.report.line">
                                <field name="name">I. Imports from states that are not members of the Eurasian Economic Union</field>
                                <field name="code">L10N_KZ_300_00_016_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_016_1_net" model="account.report.expression">
                                        <field name="label">net</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Imports from non-EEU states net</field>
                                    </record>
                                    <record id="l10n_kz_tr_expression_300_00_016_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Imports from non-EEU states tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_016_2" model="account.report.line">
                                <field name="name">II. Imports from member states of the Eurasian Economic Union</field>
                                <field name="code">L10N_KZ_300_00_016_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_016_2_net" model="account.report.expression">
                                        <field name="label">net</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Imports from member states of the EEU net</field>
                                    </record>
                                    <record id="l10n_kz_tr_expression_300_00_016_2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Imports from member states of the EEU tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_017" model="account.report.line">
                        <field name="name">300.00.017. Exempted imports of goods</field>
                        <field name="code">L10N_KZ_300_00_017</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_017_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Exempted imports of goods net</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_018" model="account.report.line">
                        <field name="name">300.00.018. Import of goods for which the deadline for payment of VAT has been changed (on the basis of the declaration of goods)</field>
                        <field name="code">L10N_KZ_300_00_018</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_018_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_019" model="account.report.line">
                        <field name="name">300.00.019. VAT paid on imported goods, for which the deadline for VAT payment was changed</field>
                        <field name="code">L10N_KZ_300_00_019</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_019_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_020" model="account.report.line">
                        <field name="name">300.00.020. Imports of goods for which VAT was paid by the offset method in accordance with the terms of the subsoil use contract</field>
                        <field name="code">L10N_KZ_300_00_020</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_020_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                            <record id="l10n_kz_tr_expression_300_00_020_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_021" model="account.report.line">
                        <field name="name">300.00.021. Total purchases</field>
                        <field name="code">L10N_KZ_300_00_021</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_021_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_013.net + L10N_KZ_300_00_014.net + L10N_KZ_300_00_015.net + L10N_KZ_300_00_016_1.net + L10N_KZ_300_00_016_2.net + L10N_KZ_300_00_017.net + L10N_KZ_300_00_020.net + L10N_KZ_300_00_029.net</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_022" model="account.report.line">
                        <field name="name">300.00.022. Adjustment of the amount of VAT to be credited</field>
                        <field name="code">L10N_KZ_300_00_022</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_022_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_023" model="account.report.line">
                        <field name="name">300.00.023. Total amount of VAT to be credited</field>
                        <field name="code">L10N_KZ_300_00_023</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_023_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_013.tax + L10N_KZ_300_00_014.tax + L10N_KZ_300_00_016_1.tax + L10N_KZ_300_00_016_2.tax + L10N_KZ_300_00_019.tax + L10N_KZ_300_00_020.tax + L10N_KZ_300_00_022.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_024" model="account.report.line">
                        <field name="name">300.00.024. The total amount of VAT to be credited when applying the proportional method with the right to separate accounting for certain turnovers, including:</field>
                        <field name="code">L10N_KZ_300_00_024</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_024_1" model="account.report.line">
                                <field name="name">I. For goods, works, services for which the proportional method of offsetting is applied</field>
                                <field name="code">L10N_KZ_300_00_024_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_024_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_024_2" model="account.report.line">
                                <field name="name">II. For goods, works, services, for which the method of separate accounting is applied</field>
                                <field name="code">L10N_KZ_300_00_024_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_024_2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_024_3" model="account.report.line">
                                <field name="name">III. For goods, works, services used simultaneously for the purposes of taxable and non-taxable turnover</field>
                                <field name="code">L10N_KZ_300_00_024_3</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_024_3_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_024_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">L10N_KZ_300_00_024_1.tax + L10N_KZ_300_00_024_2.tax + L10N_KZ_300_00_024_3.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_025" model="account.report.line">
                        <field name="name">300.00.025. The amount of VAT allowed to be deducted:</field>
                        <field name="code">L10N_KZ_300_00_025</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_025_1" model="account.report.line">
                                <field name="name">I. With the proportional method</field>
                                <field name="code">L10N_KZ_300_00_025_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_025_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_025_2" model="account.report.line">
                                <field name="name">II. With the split method</field>
                                <field name="code">L10N_KZ_300_00_025_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_025_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_025_3" model="account.report.line">
                                <field name="name">III. At the proportional method with separate accounting on separate turnovers</field>
                                <field name="code">L10N_KZ_300_00_025_3</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_025_3_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_025_4" model="account.report.line">
                                <field name="name">IV. When applying the provisions of Article 411 of the Tax Code (additional VAT credit)</field>
                                <field name="code">L10N_KZ_300_00_025_4</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_025_4_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_026" model="account.report.line">
                        <field name="name">300.00.026. The amount of VAT not allowed to be set off:</field>
                        <field name="code">L10N_KZ_300_00_026</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_026_1" model="account.report.line">
                                <field name="name">I. With the proportional method</field>
                                <field name="code">L10N_KZ_300_00_026_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_026_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_026_2" model="account.report.line">
                                <field name="name">II. Through separate accounting</field>
                                <field name="code">L10N_KZ_300_00_026_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_026_2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_026_3" model="account.report.line">
                                <field name="name">III. At proportional and separate method with separate accounting on separate turnovers</field>
                                <field name="code">L10N_KZ_300_00_026_3</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_026_3_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_027" model="account.report.line">
                        <field name="name">300.00.027. The amount of excess VAT, attributable to offset, over the amount of accrued tax</field>
                        <field name="code">L10N_KZ_300_00_027</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_027_1" model="account.report.line">
                                <field name="name">I. Accrued at the beginning of the reporting tax period on an accrual basis for the activities provided for by Article 411 of the Tax Code</field>
                                <field name="code">L10N_KZ_300_00_027_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_027_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_027_2" model="account.report.line">
                                <field name="name">II. Cumulative total on the declaration at the end of the reporting tax period</field>
                                <field name="code">L10N_KZ_300_00_027_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_027_2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_028" model="account.report.line">
                        <field name="name">300.00.028. Amount of VAT on goods, works, services used for the purposes of turnovers taxed at zero rate</field>
                        <field name="code">L10N_KZ_300_00_028</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_028_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_029" model="account.report.line">
                        <field name="name">300.00.029. Import of goods for which VAT is paid by offset method</field>
                        <field name="code">L10N_KZ_300_00_029</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_029_net" model="account.report.expression">
                                <field name="label">net</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                            <record id="l10n_kz_tr_expression_300_00_029_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_kz_tr_title_vat_tax_period" model="account.report.line">
                <field name="name">VAT calculations for the tax period</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_kz_tr_line_300_00_030" model="account.report.line">
                        <field name="name">300.00.030. Calculated amount of VAT for the tax period:</field>
                        <field name="code">L10N_KZ_300_00_030</field>
                        <field name="children_ids">
                            <record id="l10n_kz_tr_line_300_00_030_1" model="account.report.line">
                                <field name="name">I. Amount of VAT payable to the budget</field>
                                <field name="code">L10N_KZ_300_00_030_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_030_1_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_kz_tr_line_300_00_030_2" model="account.report.line">
                                <field name="name">II. Excess of the amount of VAT credited over the amount of accrued tax</field>
                                <field name="code">L10N_KZ_300_00_030_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_kz_tr_expression_300_00_030_2_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_kz_tr_line_300_00_031" model="account.report.line">
                        <field name="name">300.00.031. Reducing the amount of excess VAT generated after meeting the requirements specified in subparagraph 3) of paragraph 1 of Article 369 of the Tax Code</field>
                        <field name="code">L10N_KZ_300_00_031</field>
                        <field name="expression_ids">
                            <record id="l10n_kz_tr_expression_300_00_031_tax" model="account.report.expression">
                                <field name="label">tax</field>
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

## File: data\template\account.account-kz.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@ru"
"kz1000","Cash","1000","asset_cash","","False","Денежные средства"
"kz1010","Cash on hand","1010","asset_cash","","False","Денежные средства в кассе"
"kz1030","Cash in current bank accounts","1030","asset_cash","","False","Денежные средства на текущих банковских счетах"
"kz1040","Cash on correspondent accounts","1040","asset_cash","","False","Денежные средства на корреспондентских счетах"
"kz1050","Cash in savings accounts","1050","asset_cash","","False","Денежные средства на сберегательных счетах"
"kz1060","Restricted cash","1060","asset_cash","","False","Денежные средства, ограниченные в использовании"
"kz1070","Cash accounting for electronic cash","1070","asset_cash","","False","Учет электронных денежных средств"
"kz1080","Other cash","1080","asset_cash","","False","Прочие денежные средства"
"kz1090","Valuation allowance for cash impairment losses","1090","asset_cash","","False","Оценочный резерв под убытки от обесценения денежных средств"
"kz1100","Short-term financial assets","1100","asset_current","","False","Краткосрочные финансовые активы"
"kz1110","Short-term financial assets measured at amortized cost","1110","asset_current","","False","Краткосрочные финансовые активы, оцениваемые по амортизированной стоимости"
"kz1120","Short-term financial assets measured at fair value through other comprehensive income","1120","asset_current","","False","Краткосрочные финансовые активы, оцениваемые по справедливой стоимости через прочий совокупный доход"
"kz1130","Short-term financial assets at fair value through profit or loss","1130","asset_current","","False","Краткосрочные финансовые активы, оцениваемые по справедливой стоимости через прибыль или убыток"
"kz1140","Derivative financial instruments","1140","asset_current","","False","Производные финансовые инструменты"
"kz1150","Short term interest receivable","1150","asset_current","","False","Краткосрочные вознаграждения к получению"
"kz1160","Other current financial assets","1160","asset_current","","False","Прочие краткосрочные финансовые активы"
"kz1170","Provision for impairment losses on short-term financial assets","1170","asset_current","","False","Оценочный резерв под убытки от обесценения краткосрочных финансовых активов"
"kz1200","Short-term accounts receivable","1200","asset_receivable","","True","Краткосрочная дебиторская задолженность"
"kz1210","Short-term accounts receivable from buyers and customers","1210","asset_receivable","","True","Краткосрочная дебиторская задолженность покупателей и заказчиков"
"kz1220","Short-term accounts receivable from subsidiaries","1220","asset_receivable","","True","Краткосрочная дебиторская задолженность дочерних организаций"
"kz1230","Short-term accounts receivable from associates and joint ventures","1230","asset_receivable","","True","Краткосрочная дебиторская задолженность ассоциированных и совместных организаций"
"kz1240","Short-term accounts receivable from branches and divisions","1240","asset_receivable","","True","Краткосрочная дебиторская задолженность филиалов и структурных подразделений"
"kz1250","Short-term accounts receivable from employees","1250","asset_receivable","","True","Краткосрочная дебиторская задолженность работников"
"kz1260","Short-term lease receivables","1260","asset_receivable","","True","Краткосрочная дебиторская задолженность по аренде"
"kz1270","Other short-term receivables","1270","asset_receivable","","True","Прочая краткосрочная дебиторская задолженность"
"kz1280","Estimated reserve for impairment of short-term accounts receivable","1280","asset_receivable","","True","Оценочный резерв под убытки от обесценения краткосрочной дебиторской задолженности"
"kz1300","Inventories","1300","asset_current","","False","Запасы"
"kz1310","Raw materials and supplies","1310","asset_current","","False","Сырье и материалы"
"kz1320","Finished goods","1320","asset_current","","False","Готовая продукция"
"kz1330","Goods","1330","asset_current","","False","Товары"
"kz1340","Work in progress","1340","asset_current","","False","Незавершенное производство"
"kz1350","Other inventories","1350","asset_current","","False","Прочие запасы"
"kz1360","Valuation allowance for inventory impairment","1360","asset_current","","False","Оценочный резерв под убытки от обесценения запасов"
"kz1370","Right-of-use asset","1370","asset_current","","False","Актив по праву на возврат запасов"
"kz1400","Current tax assets","1400","asset_current","","False","Текущие налоговые активы"
"kz1410","Corporate income tax","1410","asset_current","","False","Корпоративный подоходный налог"
"kz1420","Value added tax","1420","asset_current","","False","Налог на добавленную стоимость"
"kz1430","Other taxes and other obligatory payments to the budget","1430","asset_current","","False","Прочие налоги и другие обязательные платежи в бюджет"
"kz1500","Non-current assets held for sale","1500","asset_current","","False","Долгосрочные активы, предназначенные для продажи"
"kz1510","Non-current assets held for sale","1510","asset_current","","False","Долгосрочные активы, предназначенные для продажи"
"kz1520","Disposal groups held for sale","1520","asset_current","","False","Группы на выбытие, предназначенные для продажи"
"kz1530","Valuation allowance for impairment losses on long-lived assets (or disposal groups) held for sale","1530","asset_current","","False","Оценочный резерв под убытки от обесценения долгосрочных активов (или выбывающих групп), предназначенных для продажи"
"kz1600","Biological assets","1600","asset_current","","False","Биологические активы"
"kz1610","Plants","1610","asset_current","","False","Растения"
"kz1620","Animals","1620","asset_current","","False","Животные"
"kz1630","Estimated allowance for impairment losses on biological assets","1630","asset_current","","False","Оценочный резерв под убытки от обесценения биологических активов"
"kz1700","Other current assets","1700","asset_current","","False","Прочие краткосрочные активы"
"kz1710","Short term advances paid","1710","asset_current","","False","Краткосрочные авансы выданные"
"kz1720","Deferred expenses","1720","asset_current","","False","Расходы будущих периодов"
"kz1730","Short-term contract assets","1730","asset_current","","False","Краткосрочные активы по договорам"
"kz1740","Valuation allowance for losses on impairment of short-term contract assets","1740","asset_current","","False","Оценочный резерв под убытки от обесценения краткосрочных активов по договорам"
"kz1750","Other current assets","1750","asset_non_current","","False","Прочие краткосрочные активы"
"kz2000","Long-term financial assets","2000","asset_non_current","","False","Долгосрочные финансовые активы"
"kz2010","Long term financial assets measured at amortized cost","2010","asset_non_current","","False","Долгосрочные финансовые активы, оцениваемые по амортизированной стоимости"
"kz2020","Long term financial assets at fair value through other comprehensive income","2020","asset_non_current","","False","Долгосрочные финансовые активы, оцениваемые по справедливой стоимости через прочий совокупный доход"
"kz2030","Long term financial assets at fair value through profit or loss","2030","asset_non_current","","False","Долгосрочные финансовые активы, оцениваемые по справедливой стоимости через прибыль или убыток"
"kz2040","Derivative financial instruments","2040","asset_non_current","","False","Производные финансовые инструменты"
"kz2050","Long-term benefits receivable","2050","asset_non_current","","False","Долгосрочные вознаграждения к получению"
"kz2060","Equity instruments","2060","asset_non_current","","False","Долевые инструменты"
"kz2070","Other non-current financial assets","2070","asset_non_current","","False","Прочие долгосрочные финансовые активы"
"kz2080","Valuation allowance for long term financial assets","2080","asset_non_current","","False","Оценочный резерв под убытки от обесценения долгосрочных финансовых активов"
"kz2100","Long-term accounts receivable","2100","asset_non_current","","False","Долгосрочная дебиторская задолженность"
"kz2110","Long-term accounts receivable from buyers and customers","2110","asset_non_current","","False","Долгосрочная задолженность покупателей и заказчиков"
"kz2120","Long-term accounts receivable from subsidiaries","2120","asset_non_current","","False","Долгосрочная дебиторская задолженность дочерних организаций"
"kz2130","Long-term accounts receivable from associates and joint ventures","2130","asset_non_current","","False","Долгосрочная дебиторская задолженность ассоциированных и совместных организаций"
"kz2140","Long-term accounts receivable from affiliates and subsidiaries","2140","asset_non_current","","False","Долгосрочная дебиторская задолженность филиалов и структурных подразделений"
"kz2150","Long-term accounts receivable from employees","2150","asset_non_current","","False","Долгосрочная дебиторская задолженность работников"
"kz2160","Long-term lease receivables","2160","asset_non_current","","False","Долгосрочная дебиторская задолженность по аренде"
"kz2170","Other long-term receivables","2170","asset_non_current","","False","Прочая долгосрочная дебиторская задолженность"
"kz2180","Valuation allowance for impairment of long-term accounts receivable","2180","asset_non_current","","False","Оценочный резерв под убытки от обесценения долгосрочной дебиторской задолженности"
"kz2200","Investments accounted for using the equity method","2200","asset_non_current","","False","Инвестиции, учитываемые методом долевого участия"
"kz2210","Investments accounted for using the equity method","2210","asset_non_current","","False","Инвестиции, учитываемые методом долевого участия"
"kz2220","Investments accounted for under the historical cost method","2220","asset_non_current","","False","Инвестиции, учитываемые по первоначальной стоимости"
"kz2230","Valuation allowance for impairment losses on investments","2230","asset_non_current","","False","Оценочный резерв под убытки от обесценения инвестиций"
"kz2300","Investment property","2300","asset_non_current","","False","Инвестиционное имущество"
"kz2310","Investment property","2310","asset_non_current","","False","Инвестиционное имущество"
"kz2320","Depreciation of investment property","2320","asset_non_current","","False","Амортизация инвестиционного имущества"
"kz2330","Valuation allowance for impairment of investment property","2330","asset_non_current","","False","Оценочный резерв под убытки от обесценения инвестиционного имущества"
"kz2400","Property, plant and equipment","2400","asset_non_current","","False","Основные средства"
"kz2410","Property, plant and equipment","2410","asset_non_current","","False","Основные средства"
"kz2420","Depreciation of property, plant and equipment","2420","asset_non_current","","False","Амортизация основных средств"
"kz2430","Valuation allowance for impairment of property, plant and equipment","2430","asset_non_current","","False","Оценочный резерв под убытки от обесценения основных средств"
"kz2440","Right to use an asset","2440","asset_non_current","","False","Право пользования активом"
"kz2450","Depreciation of the right to use the asset","2450","asset_non_current","","False","Амортизация права пользования активом"
"kz2460","Loss from impairment of the right to use the asset","2460","asset_non_current","","False","Убыток от обесценения права пользования активом"
"kz2500","Biological assets","2500","asset_non_current","","False","Биологические активы"
"kz2510","Plants","2510","asset_non_current","","False","Растения"
"kz2520","Animals","2520","asset_non_current","","False","Животные"
"kz2530","Depreciation of biological assets","2530","asset_non_current","","False","Амортизация биологических активов"
"kz2540","Valuation allowance for impairment losses on biological assets","2540","asset_non_current","","False","Оценочный резерв под убытки от обесценения биологических активов"
"kz2600","Exploration and evaluation assets","2600","asset_non_current","","False","Разведочные и оценочные активы"
"kz2610","Exploration and evaluation assets","2610","asset_non_current","","False","Разведочные и оценочные активы"
"kz2620","Depreciation of Exploration and Appraisal Assets","2620","asset_non_current","","False","Амортизация разведочных и оценочных активов"
"kz2630","Appraisal allowance for losses on impairment of exploration and evaluation assets","2630","asset_non_current","","False","Оценочный резерв под убытки от обесценения разведочных и оценочных активов"
"kz2700","Intangible assets","2700","asset_fixed","","False","Нематериальные активы"
"kz2710","Goodwill","2710","asset_fixed","","False","Гудвилл"
"kz2720","Impairment of goodwill","2720","asset_fixed","","False","Обесценение гудвилла"
"kz2730","Other intangible assets","2730","asset_fixed","","False","Прочие нематериальные активы"
"kz2740","Amortization of other intangible assets","2740","asset_fixed","","False","Амортизация прочих нематериальных активов"
"kz2750","Valuation allowance for impairment losses on other intangible assets","2750","asset_fixed","","False","Оценочный резерв под убытки от обесценения прочих нематериальных активов"
"kz2760","Right to use the asset","2760","asset_fixed","","False","Право пользования активом"
"kz2770","Amortization of right to use an asset","2770","asset_fixed","","False","Амортизация права пользования активом"
"kz2780","Valuation allowance for impairment losses on right to use assets","2780","asset_fixed","","False","Оценочный резерв под убытки от обесценения права пользования активом"
"kz2800","Deferred tax assets","2800","asset_fixed","","False","Отложенные налоговые активы"
"kz2810","Deferred corporate income tax assets","2810","asset_fixed","","False","Отложенные налоговые активы по корпоративному подоходному налогу"
"kz2900","Other non-current assets","2900","asset_fixed","","False","Прочие долгосрочные активы"
"kz2910","Long term advances paid","2910","asset_fixed","","False","Долгосрочные авансы выданные"
"kz2920","Deferred expenses","2920","asset_fixed","","False","Расходы будущих периодов"
"kz2930","Construction in progress","2930","asset_fixed","","False","Незавершенное строительство"
"kz2940","Long-term contractual assets","2940","asset_fixed","","False","Долгосрочные активы по договорам"
"kz2950","Valuation allowance for losses on impairment of long-lived assets","2950","asset_fixed","","False","Оценочный резерв под убытки от обесценения долгосрочных активов по договорам"
"kz2960","Contractual costs","2960","asset_fixed","","False","Затраты по договорам"
"kz2970","Amortization of Contract Costs","2970","asset_fixed","","False","Амортизация затрат по договорам"
"kz2980","Valuation allowance for losses on impairment of contract costs","2980","asset_fixed","","False","Оценочный резерв под убытки от обесценения затрат по договорам"
"kz2990","Other non-current assets, which include other groups of non-current assets not shown in the previous groupings","2990","asset_fixed","","False","Прочие долгосрочные активы, где учитываются прочие группы долгосрочных активов, не указанные в предыдущих группах"
"kz3000","Current financial liabilities","3000","liability_current","","False","Краткосрочные финансовые обязательства"
"kz3010","Current financial liabilities measured at amortized cost","3010","liability_current","","False","Краткосрочные финансовые обязательства, оцениваемые по амортизированной стоимости"
"kz3020","Current financial liabilities at amortized cost","3020","liability_current","","False","Краткосрочные финансовые обязательства, оцениваемые по справедливой стоимости через прибыль или убыток"
"kz3030","Derivative financial instruments","3030","liability_current","","False","Производные финансовые инструменты"
"kz3040","Short-term payables on dividends and participants income","3040","liability_current","","False",""
"kz3050","Short-term benefits payable","3050","liability_current","","False","Краткосрочные вознаграждения к выплате"
"kz3060","Current portion of long-term financial liabilities at amortized cost","3060","liability_current","","False","Текущая часть долгосрочных финансовых обязательств, оцениваемых по амортизированной стоимости"
"kz3070","Current portion of long-term financial liabilities at fair value through profit or loss","3070","liability_current","","False","Текущая часть долгосрочных финансовых обязательств, оцениваемых по справедливой стоимости через прибыль или убытки"
"kz3080","Other current financial liabilities","3080","liability_current","","False","Прочие краткосрочные финансовые обязательства"
"kz3100","Tax liabilities","3100","liability_current","","False","Обязательства по налогам"
"kz3110","Corporate income tax payable","3110","liability_current","","False","Корпоративный подоходный налог, подлежащий уплате"
"kz3120","Individual income tax","3120","liability_current","","False","Индивидуальный подоходный налог"
"kz3130","Value added tax","3130","liability_current","","False","Налог на добавленную стоимость"
"kz3140","Excise tax","3140","liability_current","","False","Акцизы"
"kz3150","Social tax","3150","liability_current","","False","Социальный налог"
"kz3160","Land tax","3160","liability_current","","False","Земельный налог"
"kz3170","Vehicle tax","3170","liability_current","","False","Налог на транспортные средства"
"kz3180","Property tax","3180","liability_current","","False","Налог на имущество"
"kz3190","Other taxes","3190","liability_current","","False","Прочие налоги"
"kz3200","Other compulsory and voluntary payments","3200","liability_current","","False","Обязательства по другим обязательным и добровольным платежам"
"kz3210","Social insurance liabilities","3210","liability_current","","False","Обязательства по социальному страхованию"
"kz3220","Retirement benefit obligations","3220","liability_current","","False","Обязательства по пенсионным отчислениям"
"kz3230","Other liabilities on other obligatory payments","3230","liability_current","","False","Прочие обязательства по другим обязательным платежам"
"kz3240","Other liabilities on other voluntary payments","3240","liability_current","","False","Прочие обязательства по другим добровольным платежам"
"kz3300","Short-term accounts payable","3300","liability_payable","","True","Краткосрочная кредиторская задолженность"
"kz3310","Payables to suppliers and contractors","3310","liability_payable","","True","Краткосрочная кредиторская задолженность поставщикам и подрядчикам"
"kz3320","Short-term accounts payable to subsidiaries","3320","liability_payable","","True","Краткосрочная кредиторская задолженность дочерним организациям"
"kz3330","Short-term accounts payable to associates and joint ventures","3330","liability_payable","","True","Краткосрочная кредиторская задолженность ассоциированным и совместным организациям"
"kz3340","Short-term accounts payable to affiliates and subsidiaries","3340","liability_payable","","True","Краткосрочная кредиторская задолженность филиалам и структурным подразделениям"
"kz3350","Short-term accounts payable to employees","3350","liability_payable","","True","Краткосрочная задолженность по оплате труда"
"kz3360","Short-term payables for leases","3360","liability_payable","","True","Краткосрочная задолженность по аренде"
"kz3370","Current portion of long-term accounts payable","3370","liability_payable","","True","Текущая часть долгосрочной кредиторской задолженности"
"kz3380","Other short-term accounts payable","3380","liability_payable","","True","Прочая краткосрочная кредиторская задолженность"
"kz3400","Short-term estimated liabilities","3400","liability_current","","False","Краткосрочные оценочные обязательства"
"kz3410","Short-term warranty obligations","3410","liability_current","","False","Краткосрочные гарантийные обязательства"
"kz3420","Short-term liabilities on legal claims","3420","liability_current","","False","Краткосрочные обязательства по юридическим претензиям"
"kz3430","Short-term estimated liabilities on employee benefits","3430","liability_current","","False","Краткосрочные оценочные обязательства по вознаграждениям работникам"
"kz3440","Other current estimated liabilities","3440","liability_current","","False","Прочие краткосрочные оценочные обязательства"
"kz3500","Other current liabilities","3500","liability_current","","False","Прочие краткосрочные обязательства"
"kz3510","Short-term advances received","3510","liability_current","","False","Краткосрочные авансы полученные"
"kz3520","Deferred income","3520","liability_current","","False","Доходы будущих периодов"
"kz3530","Liabilities of the disposal group held for sale","3530","liability_current","","False","Обязательства группы на выбытие, предназначенной для продажи"
"kz3540","Short-term contractual liabilities","3540","liability_current","","False","Краткосрочные обязательства по договорам"
"kz3550","Debt component of a compound short-term financial instrument","3550","liability_current","","False","Долговой компонент комбинированного краткосрочного финансового инструмента"
"kz3560","Other current liabilities","3560","liability_current","","False","Прочие краткосрочные обязательства"
"kz3570","Government grants","3570","liability_current","","False","Государственные субсидии"
"kz4000","Long-term financial liabilities","4000","liability_non_current","","False","Долгосрочные финансовые обязательства"
"kz4010","Non-current financial liabilities measured at amortized cost","4010","liability_non_current","","False","Долгосрочные финансовые обязательства, оцениваемые по амортизированной стоимости"
"kz4020","Long-term financial liabilities at fair value through profit or loss","4020","liability_non_current","","False","Долгосрочные финансовые обязательства, оцениваемые по справедливой стоимости через прибыль или убыток"
"kz4030","Derivative financial instruments","4030","liability_non_current","","False","Производные финансовые инструменты"
"kz4040","Long-term dividend and participant return obligations","4040","liability_non_current","","False","Долгосрочная задолженность по дивидендам и доходам участников"
"kz4050","Long-term benefits payable","4050","liability_non_current","","False","Долгосрочные вознаграждения к выплате"
"kz4060","Other non-current financial liabilities","4060","liability_non_current","","False","Прочие долгосрочные финансовые обязательства"
"kz4100","Long-term accounts payable","4100","liability_non_current","","False","Долгосрочная кредиторская задолженность"
"kz4110","Long-term accounts payable to suppliers and contractors","4110","liability_non_current","","False","Долгосрочная кредиторская задолженность поставщикам и подрядчикам"
"kz4120","Long-term accounts payable to subsidiaries","4120","liability_non_current","","False","Долгосрочная кредиторская задолженность дочерним организациям"
"kz4130","Long-term accounts payable to associates and joint ventures","4130","liability_non_current","","False","Долгосрочная кредиторская задолженность ассоциированным и совместным организациям"
"kz4140","Long-term accounts payable to branches and subsidiaries","4140","liability_non_current","","False","Долгосрочная кредиторская задолженность филиалам и структурным подразделениям"
"kz4150","Long-term accounts payable - lease","4150","liability_non_current","","False","Долгосрочная задолженность по аренде"
"kz4160","Other long-term accounts payable","4160","liability_non_current","","False","Прочая долгосрочная кредиторская задолженность"
"kz4164","Long-term accounts payable to employees","4164","liability_non_current","","False","Долгосрочная задолженность по депонированной заработной плате"
"kz4200","Long-term estimated liabilities","4200","liability_non_current","","False","Долгосрочные оценочные обязательства"
"kz4210","Long-term warranty liabilities","4210","liability_non_current","","False","Долгосрочные гарантийные обязательства"
"kz4220","Long-term estimated liabilities for legal claims","4220","liability_non_current","","False","Долгосрочные оценочные обязательства по юридическим претензиям"
"kz4230","Non-current estimated liabilities for employee benefits","4230","liability_non_current","","False","Долгосрочные оценочные обязательства по вознаграждениям работникам"
"kz4240","Other non-current estimated liabilities","4240","liability_non_current","","False","Прочие долгосрочные оценочные обязательства"
"kz4300","Deferred tax liabilities","4300","liability_non_current","","False","Отложенные налоговые обязательства"
"kz4310","Deferred corporate income tax liabilities","4310","liability_non_current","","False","Отложенные налоговые обязательства по корпоративному подоходному налогу"
"kz4400","Other non-current liabilities","4400","liability_non_current","","False","Прочие долгосрочные обязательства"
"kz4410","Long-term advances received","4410","liability_non_current","","False","Долгосрочные авансы полученные"
"kz4420","Deferred income","4420","liability_non_current","","False","Доходы будущих периодов"
"kz4430","Long-term contractual liabilities","4430","liability_non_current","","False","Долгосрочные обязательства по договорам"
"kz4440","Debt component of a compounded long-term financial instrument","4440","liability_non_current","","False","Долговой компонент комбинированного долгосрочного финансового инструмента"
"kz4450","Other non-current liabilities","4450","liability_non_current","","False","Прочие долгосрочные обязательства"
"kz4460","Government grants","4460","liability_non_current","","False","Государственные субсидии"
"kz5000","Share capital","5000","equity","","False","Уставный капитал"
"kz5010","Preferred stock","5010","equity","","False","Привилегированные акции"
"kz5020","Common stock","5020","equity","","False","Простые акции"
"kz5030","Deposits and shares","5030","equity","","False","Вклады и паи"
"kz5100","Unpaid equity","5100","equity","","False","Неоплаченный капитал"
"kz5110","Unpaid equity","5110","equity","","False","Неоплаченный капитал"
"kz5200","Treasury equity instruments","5200","equity","","False","Выкупленные собственные долевые инструменты"
"kz5210","Treasury equity instruments","5210","equity","","False","Выкупленные собственные долевые инструменты"
"kz5300","Share premium","5300","equity","","False","Эмиссионный доход"
"kz5310","Share premium","5310","equity","","False","Эмиссионный доход"
"kz5400","Additional paid-in capital","5400","equity","","False","Дополнительно оплаченный капитал"
"kz5410","Additional paid-in capital from transactions with the parent company","5410","equity","","False","Дополнительно оплаченный капитал по безвозмездным операциям с основной организацией"
"kz5420","Additional paid-in capital from other transactions","5420","equity","","False","Дополнительно оплаченный капитал по прочим операциям"
"kz5500","Reserves","5500","equity","","False","Резервы"
"kz5510","Reserve capital as set forth in the constitutional documents","5510","equity","","False","Резервный капитал, установленный учредительными документами"
"kz5520","Reserve for revaluation of fixed assets","5520","equity","","False","Резерв на переоценку основных средств"
"kz5530","Reserve for reassessment of intangible assets","5530","equity","","False","Резерв на переоценку нематериальных активов"
"kz5540","Reserve for revaluation of financial assets accounted at fair value through other comprehensive income","5540","equity","","False","Резерв на переоценку финансовых активов, учитываемых по справедливой стоимости через прочий совокупный доход"
"kz5550","Reserve for losses on financial assets","5550","equity","","False","Резерв под убытки по финансовым активам"
"kz5560","Foreign currency translation reserve for foreign operations","5560","equity","","False","Резерв на пересчет иностранной валюты по зарубежной деятельности"
"kz5570","Other reserves","5570","equity","","False","Прочие резервы"
"kz5600","Retained earnings (uncovered loss)","5600","equity","","False","Нераспределенная прибыль (непокрытый убыток)"
"kz5610","Retained earnings (uncovered loss) for the reporting year","5610","equity","","False","Нераспределенная прибыль (непокрытый убыток) отчетного года"
"kz5620","Retained earnings (uncovered loss) of previous years","5620","equity","","False","Нераспределенная прибыль (непокрытый убыток) предыдущих лет"
"kz5700","Total profit (final loss)","5700","equity","","False","Итоговая прибыль (итоговый убыток)"
"kz5710","Total profit (final loss)","5710","equity","","False","Итоговая прибыль (итоговый убыток)"
"kz6000","Income from sales of products and services","6000","income","","False","Доход от реализации продукции и оказания услуг"
"kz6010","Income from sales of products and services","6010","income","","False","Доход от реализации продукции и оказания услуг"
"kz6020","Returns on products sold","6020","income","","False","Возврат проданной продукции"
"kz6030","Discounts on prices and sales","6030","income","","False","Скидки с цены и продаж"
"kz6100","Income from financing","6100","income","","False","Доход от финансирования"
"kz6110","Compensation income","6110","income","","False","Доходы по вознаграждениям"
"kz6120","Dividend income","6120","income","","False","Доходы по дивидендам"
"kz6130","Income from finance leases","6130","income","","False","Доходы от финансовой аренды"
"kz6140","Income from operations with real estate investments","6140","income","","False","Доходы от операций с инвестициями в недвижимость"
"kz6150","Income from changes in fair value of financial instruments","6150","income","","False","Доходы от изменения справедливой стоимости финансовых инструментов"
"kz6160","Other income from financing","6160","income","","False","Прочие доходы от финансирования"
"kz6200","Other income","6200","income","","False","Прочие доходы"
"kz6210","Income from disposal of assets","6210","income","","False","Доходы от выбытия активов"
"kz6220","Income from gratuitously received assets","6220","income","","False","Доходы от безвозмездно полученных активов"
"kz6230","Income from government grants","6230","income","","False","Доходы от государственных субсидий"
"kz6240","Income from reversal of impairment loss on nonfinancial assets","6240","income","","False","Доходы от восстановления убытка от обесценения по нефинансовым активам"
"kz6250","Foreign exchange income","6250","income","","False","Доходы от курсовой разницы"
"kz6260","Income from operating leases","6260","income","","False","Доходы от операционной аренды"
"kz6270","Income from changes in fair value of biological assets","6270","income","","False","Доходы от изменения справедливой стоимости биологических активов"
"kz6280","Gains from reversal of impairment losses on financial assets","6280","income","","False","Доходы от восстановления убытка от обесценения по финансовым активам"
"kz6290","Other income","6290","income_other","","False","Прочие доходы"
"kz6291","Cash Discount Gain","6291","income_other","","False","Прибыль от дисконтирования денежных средств"
"kz6300","Income related to discontinued operations","6300","income","","False","Доходы, связанные с прекращаемой деятельностью"
"kz6310","Income attributable to discontinued operations","6310","income","","False","Доходы, связанные с прекращаемой деятельностью"
"kz6400","Share of profit of equity-accounted entities","6400","income","","False","Доля прибыли организаций, учитываемых по методу долевого участия"
"kz6410","Share of profit of associates","6410","income","","False","Доля прибыли ассоциированных организаций"
"kz6420","Share of profit of joint ventures","6420","income","","False","Доля прибыли совместных организаций"
"kz7000","Cost of products sold and services rendered","7000","expense_direct_cost","","False","Себестоимость реализованной продукции и оказанных услуг"
"kz7010","Cost of products sold and services rendered","7010","expense_direct_cost","","False","Себестоимость реализованной продукции и оказанных услуг"
"kz7100","Expenses on sales of products and provision of services","7100","expense_direct_cost","","False","Расходы по реализации продукции и оказанию услуг"
"kz7110","Expenses on sales of products and provision of services","7110","expense_direct_cost","","False","Расходы по реализации продукции и оказанию услуг"
"kz7200","Administrative expenses","7200","expense","","False","Административные расходы"
"kz7210","Administrative expenses","7210","expense","","False","Административные расходы"
"kz7300","Expenses on financing","7300","expense","","False","Расходы на финансирование"
"kz7310","Expenses for remuneration","7310","expense","","False","Расходы по вознаграждениям"
"kz7320","Interest expense on finance leases","7320","expense","","False","Расходы на выплату процентов по финансовой аренде"
"kz7330","Expenses from changes in the fair value of financial instruments","7330","expense","","False","Расходы от изменения справедливой стоимости финансовых инструментов"
"kz7340","Other finance costs","7340","expense","","False","Прочие расходы на финансирование"
"kz7400","Other expenses","7400","expense","","False","Прочие расходы"
"kz7410","Expenses from disposal of assets","7410","expense","","False","Расходы по выбытию активов"
"kz7420","Expenses from impairment of nonfinancial assets","7420","expense","","False","Расходы от обесценения нефинансовых активов"
"kz7430","Expenses from exchange rate differences","7430","expense","","False","Расходы по курсовой разнице"
"kz7440","Expenses from impairment of accounts receivable","7440","expense","","False","Расходы по обесценению дебиторской задолженности"
"kz7450","Expenses from operating leases","7450","expense","","False","Расходы по операционной аренде"
"kz7460","Expenses from changes in fair value of biological assets","7460","expense","","False","Расходы от изменения справедливой стоимости биологических активов"
"kz7470","Expenses from impairment of financial instruments","7470","expense","","False","Расходы от обесценения финансовых инструментов"
"kz7480","Other expenses","7480","expense","","False","Прочие расходы"
"kz7481","Cash Discount Loss","7481","expense","","False","Убыток от дисконтирования денежных средств"
"kz7500","Costs related to discontinued operations","7500","expense","","False","Расходы, связанные с прекращаемой деятельностью"
"kz7510","Costs related to discontinued operations","7510","expense","","False","Расходы, связанные с прекращаемой деятельностью"
"kz7600","Share of loss of equity-accounted entities","7600","expense","","False","Доля в убытке организаций, учитываемых методом долевого участия"
"kz7610","Share of loss of equity-accounted investees","7610","expense","","False","Доля в убытке ассоциированных организациях"
"kz7620","Share of loss of joint ventures","7620","expense","","False","Доля в убытке совместных организациях"
"kz7700","Corporate income tax expense","7700","expense","","False","Расходы по корпоративному подоходному налогу"
"kz7710","Income tax expense","7710","expense","","False","Расходы по корпоративному подоходному налогу"
"kz8100","Core production","8100","expense","","False","Основное производство"
"kz8110","Core production","8110","expense","","False","Основное производство"
"kz8200","Semifinished goods of own production","8200","expense","","False","Полуфабрикаты собственного производства"
"kz8210","Semifinished goods of own production","8210","expense","","False","Полуфабрикаты собственного производства"
"kz8300","Auxiliary production","8300","expense","","False","Вспомогательные производства"
"kz8310","Auxiliary production","8310","expense","","False","Вспомогательные производства"
"kz8400","Overhead costs","8400","expense","","False","Накладные расходы"
"kz8410","Overhead costs","8410","expense","","False","Накладные расходы"
"l10nkz_chart_template_liquidity_transfer","Cash in transit","1020","asset_current","","True","Денежные средства в пути"

```

## File: data\template\account.fiscal.position-kz.csv

```csv
"id","sequence","name","auto_apply","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@ru"
"l10n_kz_fiscal_position_template_national","1","National","1","base.kz","","","","Национальные"
"l10n_kz_fiscal_position_template_eeu","2","EEU","1","","base.eurasian_economic_union","l10n_kz_tax_vat_12_purchase","l10n_kz_tax_vat_12_purchase_eeu",""
"","","","","","","l10n_kz_tax_vat_20_purchase","l10n_kz_tax_vat_20_purchase_eeu",""
"l10n_kz_fiscal_position_template_international","3","International","1","","","l10n_kz_tax_vat_12_purchase","l10n_kz_tax_vat_12_purchase_import","Международные"
"","","","","","","l10n_kz_tax_vat_20_purchase","l10n_kz_tax_vat_20_purchase_import",""

```

## File: data\template\account.tax-kz.csv

```csv
"id","name","tax_group_id","amount","amount_type","type_tax_use","description","invoice_label","tax_scope","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@ru"
"l10n_kz_tax_vat_12_sale","12%","tax_group_vat_12","12.0","percent","sale","","VAT 12%","","base","invoice","+I. With the issuance of invoices net","",""
"","","","","","","","","","tax","invoice","+I. With the issuance of invoices tax","kz3130",""
"","","","","","","","","","base","refund","+I. With the issuance of invoices net","",""
"","","","","","","","","","tax","refund","+I. With the issuance of invoices tax","kz3130",""
"l10n_kz_tax_vat_12_purchase","12%","tax_group_vat_12","12.0","percent","purchase","","VAT 12%","","base","invoice","+On invoices net","",""
"","","","","","","","","","tax","invoice","+On invoices tax","kz1420",""
"","","","","","","","","","base","refund","+On invoices net","",""
"","","","","","","","","","tax","refund","+On invoices tax","kz1420",""
"l10n_kz_tax_vat_0_sale","0%","tax_group_vat_0","0.0","percent","sale","","VAT 0%","","base","invoice","+Sales turnover subject to zero VAT net","",""
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Sales turnover subject to zero VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_0_purchase","0%","tax_group_vat_0","0.0","percent","purchase","","VAT 0%","","base","invoice","+Purchases without VAT net","",""
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Purchases without VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_20_sale","20%","tax_group_vat_20","20.0","percent","sale","20%","","","base","invoice","+I. With the issuance of invoices net","",""
"","","","","","","","","","tax","invoice","+I. With the issuance of invoices tax","kz3130",""
"","","","","","","","","","base","refund","+I. With the issuance of invoices net","",""
"","","","","","","","","","tax","refund","+I. With the issuance of invoices tax","kz3130",""
"l10n_kz_tax_vat_20_purchase","20%","tax_group_vat_20","20.0","percent","purchase","20%","","","base","invoice","+On invoices net","",""
"","","","","","","","","","tax","invoice","+On invoices tax","kz1420",""
"","","","","","","","","","base","refund","+On invoices net","",""
"","","","","","","","","","tax","refund","+On invoices tax","kz1420",""
"l10n_kz_tax_vat_12_sale_export","12% EX","tax_group_vat_12","12.0","percent","sale","12% EX","Export VAT 12%","","base","invoice","+Turnover - not in the Republic of Kazakhstan net","","12% Экспорт/Импорт"
"","","","","","","","","","tax","invoice","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"","","","","","","","","","base","refund","+Turnover - not in the Republic of Kazakhstan net","",""
"","","","","","","","","","tax","refund","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"l10n_kz_tax_vat_12_purchase_import","12% EX","tax_group_vat_12","12.0","percent","purchase","12% EX","Import VAT 12%","","base","invoice","+Imports from non-EEU states net","","12% Экспорт/Импорт"
"","","","","","","","","","tax","invoice","+Imports from non-EEU states tax","kz1420",""
"","","","","","","","","","base","refund","+Imports from non-EEU states net","",""
"","","","","","","","","","tax","refund","+Imports from non-EEU states tax","kz1420",""
"l10n_kz_tax_vat_0_sale_export","0% EX","tax_group_vat_0","0.0","percent","sale","0% EX","Export VAT 0%","","base","invoice","+Sales turnover subject to zero VAT net","","0% Экспорт"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Sales turnover subject to zero VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_20_sale_export","20% EX","tax_group_vat_20","20.0","percent","sale","20% EX","Export VAT 20%","","base","invoice","+Turnover - not in the Republic of Kazakhstan net","","20% Экспорт/Импорт"
"","","","","","","","","","tax","invoice","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"","","","","","","","","","base","refund","+Turnover - not in the Republic of Kazakhstan net","",""
"","","","","","","","","","tax","refund","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"l10n_kz_tax_vat_20_purchase_import","20% EX","tax_group_vat_20","20.0","percent","purchase","20% EX","Import VAT 20%","","base","invoice","+Imports from non-EEU states net","","20% Экспорт/Импорт"
"","","","","","","","","","tax","invoice","+Imports from non-EEU states tax","kz1420",""
"","","","","","","","","","base","refund","+Imports from non-EEU states net","",""
"","","","","","","","","","tax","refund","+Imports from non-EEU states tax","kz1420",""
"l10n_kz_tax_vat_12_purchase_non_resident","12% S NR","tax_group_vat_12","12.0","percent","purchase","12% S Non-resident","Non-resident VAT 12%","service","base","invoice","+Works, services purchased from a non-resident net","","12% С. Нерезиденты"
"","","","","","","","","","tax","invoice","+Works, services purchased from a non-resident tax","kz1420",""
"","","","","","","","","","base","refund","+Works, services purchased from a non-resident net","",""
"","","","","","","","","","tax","refund","+Works, services purchased from a non-resident tax","kz1420",""
"l10n_kz_tax_vat_20_purchase_non_resident","20% S NR","tax_group_vat_20","20.0","percent","purchase","20% S Non-resident","Non-resident VAT 20%","service","base","invoice","+Works, services purchased from a non-resident net","","20% С. Нерезиденты"
"","","","","","","","","","tax","invoice","+Works, services purchased from a non-resident tax","kz1420",""
"","","","","","","","","","base","refund","+Works, services purchased from a non-resident net","",""
"","","","","","","","","","tax","refund","+Works, services purchased from a non-resident tax","kz1420",""
"l10n_kz_tax_vat_12_sale_eeu","12% EEU","tax_group_vat_12","12.0","percent","sale","12% EEU","Euroasian Economic Union VAT 12%","","base","invoice","+Turnover - not in the Republic of Kazakhstan net","","12% ЕАЭС"
"","","","","","","","","","tax","invoice","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"","","","","","","","","","base","refund","+Turnover - not in the Republic of Kazakhstan net","",""
"","","","","","","","","","tax","refund","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"l10n_kz_tax_vat_12_purchase_eeu","12% EEU","tax_group_vat_12","12.0","percent","purchase","12% EEU","Euroasian Economic Union VAT 12%","","base","invoice","+Imports from member states of the EEU net","","12% ЕАЭС"
"","","","","","","","","","tax","invoice","+Imports from member states of the EEU tax","kz1420",""
"","","","","","","","","","base","refund","+Imports from member states of the EEU net","",""
"","","","","","","","","","tax","refund","+Imports from member states of the EEU tax","kz1420",""
"l10n_kz_tax_vat_0_sale_eeu","0% EEU","tax_group_vat_0","0.0","percent","sale","0% EEU","Euroasian Economic Union VAT 0%","","base","invoice","+Sales turnover subject to zero VAT net","","0% ЕАЭС"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Sales turnover subject to zero VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_20_sale_eeu","20% EEU","tax_group_vat_20","20.0","percent","sale","20% EEU","Euroasian Economic Union VAT 20%","","base","invoice","+Turnover - not in the Republic of Kazakhstan net","","20% ЕАЭС"
"","","","","","","","","","tax","invoice","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"","","","","","","","","","base","refund","+Turnover - not in the Republic of Kazakhstan net","",""
"","","","","","","","","","tax","refund","+Turnover - not in the Republic of Kazakhstan net","kz3130",""
"l10n_kz_tax_vat_20_purchase_eeu","20% EEU","tax_group_vat_20","20.0","percent","purchase","20% EEU","Euroasian Economic Union VAT 20%","","base","invoice","+Imports from member states of the EEU net","","20% ЕАЭС"
"","","","","","","","","","tax","invoice","+Imports from member states of the EEU tax","kz1420",""
"","","","","","","","","","base","refund","+Imports from member states of the EEU net","",""
"","","","","","","","","","tax","refund","+Imports from member states of the EEU tax","kz1420",""
"l10n_kz_tax_vat_exempt_sale","0% EXEMPT","tax_group_vat_exempt","0.0","percent","sale","Exempt","VAT Exempt","","base","invoice","+Turnover exempt from VAT net","","Без НДС"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Turnover exempt from VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_exempt_purchase","0% EXEMPT","tax_group_vat_exempt","0.0","percent","purchase","Exempt","VAT Exempt","","base","invoice","+Purchases without VAT net","","Без НДС"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Purchases without VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_exempt_sale_export","0% EXEMPT EX","tax_group_vat_exempt","0.0","percent","sale","Exempt EX","Export VAT Exempt","","base","invoice","+Turnover exempt from VAT net","","Экспорт/Импорт Без НДС"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Turnover exempt from VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_exempt_purchase_import","0% EXEMPT EX","tax_group_vat_exempt","0.0","percent","purchase","Exempt EX","Import VAT Exempt","","base","invoice","+Exempted imports of goods net","","Экспорт/Импорт Без НДС"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Exempted imports of goods net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_exempt_sale_eeu","0% EXEMPT EEU","tax_group_vat_exempt","0.0","percent","sale","Exempt EEU","Euroasian Economic Union VAT Exempt","","base","invoice","+Turnover exempt from VAT net","","ЕАЭС Без НДС"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Turnover exempt from VAT net","",""
"","","","","","","","","","tax","refund","","",""
"l10n_kz_tax_vat_exempt_purchase_eeu","0% EXEMPT EEU","tax_group_vat_exempt","0.0","percent","purchase","Exempt EEU","Euroasian Economic Union VAT Exempt","","base","invoice","+Exempted imports of goods net","","ЕАЭС Без НДС"
"","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","base","refund","+Exempted imports of goods net","",""
"","","","","","","","","","tax","refund","","",""

```

## File: data\template\account.tax.group-kz.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@ru"
"tax_group_vat_exempt","VAT Exempt","base.kz","kz1400","kz3100","НДС не облагается"
"tax_group_vat_0","VAT 0%","base.kz","kz1400","kz3100","НДС 0%"
"tax_group_vat_12","VAT 12%","base.kz","kz1400","kz3100","НДС 12%"
"tax_group_vat_20","VAT 20%","base.kz","kz1400","kz3100","НДС 20%"

```

## File: models\template_kz.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('kz')
    def _get_kz_template_data(self):
        return {
            'property_account_receivable_id': 'kz1210',
            'property_account_payable_id': 'kz3310',
            'property_account_income_categ_id': 'kz6010',
            'property_account_expense_categ_id': 'kz7010',
            'code_digits': '4',
        }

    @template('kz', 'res.company')
    def _get_kz_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.kz',
                'bank_account_code_prefix': '103',
                'cash_account_code_prefix': '101',
                'transfer_account_code_prefix': '102',
                'income_currency_exchange_account_id': 'kz6250',
                'expense_currency_exchange_account_id': 'kz7430',
                'account_journal_early_pay_discount_loss_account_id': 'kz7481',
                'account_journal_early_pay_discount_gain_account_id': 'kz6291',
                'default_cash_difference_income_account_id': 'kz6210',
                'default_cash_difference_expense_account_id': 'kz7410',
                'account_sale_tax_id': 'l10n_kz_tax_vat_12_sale',
                'account_purchase_tax_id': 'l10n_kz_tax_vat_12_purchase',
            },
        }

    @template('kz', 'account.journal')
    def _get_kz_account_journal(self):
        return {
            'cash': {'default_account_id': 'kz1010'},
            'bank': {'default_account_id': 'kz1030'},
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_kz

```

