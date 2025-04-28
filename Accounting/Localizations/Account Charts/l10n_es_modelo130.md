# Odoo Module: l10n_es_modelo130

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


def _add_mod130_tax_tags(env):
    taxes_repartition_lines = env['account.tax.repartition.line'].search([
        ('tax_id', 'any', [
            ('l10n_es_type', '=', 'retencion'),
            ('country_code', '=', 'ES'),
            ('type_tax_use', '=', 'sale'),
        ]),
        ('repartition_type', '=', 'tax'),
    ])
    invoice_mod130_tax_tag = env['account.account.tag'].search([('name', '=', '-mod130[06]')])
    refund_mod130_tax_tag = env['account.account.tag'].search([('name', '=', '+mod130[06]')])

    for line in taxes_repartition_lines:
        if line.document_type == 'invoice':
            line.tag_ids += invoice_mod130_tax_tag
        else:
            line.tag_ids += refund_mod130_tax_tag

```

## File: __manifest__.py

```python
{
    'name': 'Spain - Modelo 130 Tax report',
    'website': 'https://www.odoo.com/documentation/18.0/applications/finance/fiscal_localizations/spain.html',
    'version': '1.0',
    'icon': '/account/static/description/l10n.png',
    'countries': ['es'],
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'l10n_es',
    ],
    'data': [
        'data/mod130.xml',
    ],
    'post_init_hook': '_add_mod130_tax_tags',
    'license': 'LGPL-3',
}

```

## File: data\mod130.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data auto_sequence="1">
        <record id="mod_130" model="account.report">
            <field name="name">Tax Report(Mod 130)</field>
            <field name="sequence">130</field>
            <field name="filter_analytic" eval="False"/>
            <field name="filter_date_range" eval="True"/>
            <field name="filter_period_comparison" eval="False"/>
            <field name="filter_unfold_all" eval="True"/>
            <field name="filter_journals" eval="True"/>
            <field name="country_id" ref="base.es"/>
            <field name="filter_multi_company">tax_units</field>
            <field name="root_report_id" ref="account.generic_tax_report"/>
            <field name="column_ids">
                <record id="mod_130_column" model="account.report.column">
                    <field name="name">Balance</field>
                    <field name="expression_label">balance</field>
                </record>
            </field>
            <field name="line_ids">
                <record id="mod_130_title_1" model="account.report.line">
                    <field name="name">I. Economic activities in the direct assessment system, normal or simplified mode, other than agriculture, livestock farming, forestry and fishing.</field>
                    <field name="code">aeat_mod_130_title_1</field>
                    <field name="hierarchy_level">0</field>
                    <field name="children_ids">
                        <record id="mod_130_casilla_01" model="account.report.line">
                            <field name="name">[01] Revenue</field>
                            <field name="code">aeat_mod_130_01</field>
                            <field name="groupby">account_id</field>
                            <field name="foldable" eval="True"/>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_01_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">domain</field>
                                    <field name="formula" eval="[('account_id.internal_group', '=', 'income'), '|', ('partner_id.industry_id', '=', False), ('partner_id.industry_id.name', '!=', 'Agriculture')]"/>
                                    <field name="subformula">-sum</field>
                                    <field name="date_scope">from_fiscalyear</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_02" model="account.report.line">
                            <field name="name">[02] Supplies</field>
                            <field name="code">aeat_mod_130_02</field>
                            <field name="groupby">account_id</field>
                            <field name="foldable" eval="True"/>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_02_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">domain</field>
                                    <field name="formula" eval="[('account_id.internal_group', '=', 'expense'), '|', ('partner_id.industry_id', '=', False), ('partner_id.industry_id.name', '!=', 'Agriculture')]"/>
                                    <field name="subformula">sum</field>
                                    <field name="date_scope">from_fiscalyear</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_03" model="account.report.line">
                            <field name="name">[03] Box 01 - Box 02</field>
                            <field name="code">aeat_mod_130_03</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_03_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_01.balance - aeat_mod_130_02.balance</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_04" model="account.report.line">
                            <field name="name">[04] 20% (or custom %) applied on Box 03</field>
                            <field name="code">aeat_mod_130_04</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_04_percentage" model="account.report.expression">
                                    <field name="label">percentage</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">20</field>
                                    <field name="figure_type">percentage</field>
                                </record>
                                <record id="mod_130_casilla_04_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_03.balance * (aeat_mod_130_04.percentage / 100)</field>
                                    <field name="subformula">if_above(EUR(0))</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_05" model="account.report.line">
                            <field name="name">[05] Residual amounts of past quarters</field>
                            <field name="code">aeat_mod_130_05</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_05_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">external</field>
                                    <field name="formula">sum</field>
                                    <field name="subformula">editable;rounding=2</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_06" model="account.report.line">
                            <field name="name">[06] Sum of withholding's</field>
                            <field name="code">aeat_mod_130_06</field>
                            <field name="groupby">account_id</field>
                            <field name="foldable" eval="True"/>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_06_tax_tags" model="account.report.expression">
                                    <field name="label">tags</field>
                                    <field name="engine">tax_tags</field>
                                    <field name="formula">mod130[06]</field>
                                    <field name="date_scope">from_fiscalyear</field>
                                </record>
                                <record id="mod_130_casilla_06_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">domain</field>
                                    <field name="formula" eval="[('tax_tag_ids', '=', '-mod130[06]'), '|', ('partner_id.industry_id', '=', False), ('partner_id.industry_id.name', '!=', 'Agriculture')]"/>
                                    <field name="subformula">sum</field>
                                    <field name="date_scope">from_fiscalyear</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_07" model="account.report.line">
                            <field name="name">[07] Box 04 - Box 05 - Box 06</field>
                            <field name="code">aeat_mod_130_07</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_07_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_04.balance - aeat_mod_130_05.balance - aeat_mod_130_06.balance</field>
                                </record>
                            </field>
                        </record>
                    </field>
                </record>
                <record id="mod_130_title_2" model="account.report.line">
                    <field name="name">II. Agriculture, livestock farming, forestry and fishing in the direct assessment system, normal or simplified mode.</field>
                    <field name="code">aeat_mod_130_title_2</field>
                    <field name="hierarchy_level">0</field>
                    <field name="children_ids">
                        <record id="mod_130_casilla_08" model="account.report.line">
                            <field name="name">[08] Revenue</field>
                            <field name="code">aeat_mod_130_08</field>
                            <field name="groupby">account_id</field>
                            <field name="foldable" eval="True"/>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_08_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">domain</field>
                                    <field name="formula" eval="[('account_id.internal_group', '=', 'income'), ('partner_id.industry_id.name', '=', 'Agriculture')]"/>
                                    <field name="subformula">-sum</field>
                                    <field name="date_scope">from_fiscalyear</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_09" model="account.report.line">
                            <field name="name">[09] 2% (or custom %) applied on Box 08</field>
                            <field name="code">aeat_mod_130_09</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_09_percentage" model="account.report.expression">
                                    <field name="label">percentage</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">2</field>
                                    <field name="figure_type">percentage</field>
                                </record>
                                <record id="mod_130_casilla_09_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_08.balance * (aeat_mod_130_09.percentage / 100)</field>
                                    <field name="subformula">if_above(EUR(0))</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_10" model="account.report.line">
                            <field name="name">[10] Sum of withholding's</field>
                            <field name="code">aeat_mod_130_10</field>
                            <field name="groupby">account_id</field>
                            <field name="foldable" eval="True"/>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_10_tax_tags" model="account.report.expression">
                                    <field name="label">tags</field>
                                    <field name="engine">tax_tags</field>
                                    <field name="formula">mod130[06]</field>
                                    <field name="date_scope">from_fiscalyear</field>
                                </record>
                                <record id="mod_130_casilla_10_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">domain</field>
                                    <field name="formula" eval="[('tax_tag_ids', '=', '-mod130[06]'), ('partner_id.industry_id.name', '=', 'Agriculture')]"/>
                                    <field name="subformula">sum</field>
                                    <field name="date_scope">from_fiscalyear</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_11" model="account.report.line">
                            <field name="name">[11] Box 09 - Box 10</field>
                            <field name="code">aeat_mod_130_11</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_11_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_09.balance - aeat_mod_130_10.balance</field>
                                </record>
                            </field>
                        </record>
                    </field>
                </record>
                <record id="mod_130_title_3" model="account.report.line">
                    <field name="name">III. Total settlement.</field>
                    <field name="code">aeat_mod_130_title_3</field>
                    <field name="hierarchy_level">0</field>
                    <field name="children_ids">
                        <record id="mod_130_casilla_12" model="account.report.line">
                            <field name="name">[12] Box 07 + Box 11</field>
                            <field name="code">aeat_mod_130_12</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_12_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_07.balance + aeat_mod_130_11.balance</field>
                                    <field name="subformula">if_above(EUR(0))</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_13" model="account.report.line">
                            <field name="name">[13] Value of the reduction based on sum of net earnings from the previous tax year</field>
                            <field name="code">aeat_mod_130_13</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_13_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">external</field>
                                    <field name="formula">sum</field>
                                    <field name="subformula">editable;rounding=2</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_14" model="account.report.line">
                            <field name="name">[14] Box 12 - Box 13</field>
                            <field name="code">aeat_mod_130_14</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_14_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_12.balance - aeat_mod_130_13.balance</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_15" model="account.report.line">
                            <field name="name">[15] Negative amount of previous self-assessments</field>
                            <field name="code">aeat_mod_130_15</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_15_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">external</field>
                                    <field name="formula">sum</field>
                                    <field name="subformula">editable;rounding=2</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_16" model="account.report.line">
                            <field name="name">[16] Deduction for allocating amounts to the payment of loans for the acquisition or rehabilitation</field>
                            <field name="code">aeat_mod_130_16</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_16_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">external</field>
                                    <field name="formula">sum</field>
                                    <field name="subformula">editable;rounding=2</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_17" model="account.report.line">
                            <field name="name">[17] Box 14 - Box 15 - Box 16</field>
                            <field name="code">aeat_mod_130_17</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_17_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_14.balance - aeat_mod_130_15.balance - aeat_mod_130_16.balance</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_18" model="account.report.line">
                            <field name="name">[18] Complementary declaration</field>
                            <field name="code">aeat_mod_130_18</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_18_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">external</field>
                                    <field name="formula">sum</field>
                                    <field name="subformula">editable;rounding=2;if_above(EUR(0))</field>
                                </record>
                            </field>
                        </record>
                        <record id="mod_130_casilla_19" model="account.report.line">
                            <field name="name">[19] Box 17 - Box 18</field>
                            <field name="code">aeat_mod_130_19</field>
                            <field name="expression_ids">
                                <record id="mod_130_casilla_19_balance" model="account.report.expression">
                                    <field name="label">balance</field>
                                    <field name="engine">aggregation</field>
                                    <field name="formula">aeat_mod_130_17.balance - aeat_mod_130_18.balance</field>
                                </record>
                            </field>
                        </record>
                    </field>
                </record>
            </field>
        </record>
    </data>
</odoo>

```

