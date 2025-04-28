# Odoo Module: l10n_bh

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Bahrain - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['bh'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Bahrain in Odoo.
===========================================================================
Bahrain accounting basic charts and localization.

Activates:
 - Chart of Accounts
 - Taxes
 - Tax reports
 - Fiscal Positions
 - States
    """,
    'depends': [
        'account',
    ],
    'data': [
        'data/tax_report_full.xml',
        'data/tax_report_simplified.xml',
        'data/res.country.state.csv',
        'data/res_country_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\res.country.state.csv

```csv
"id","country_id:id","name","code"
"state_bh_1","base.bh","Capital","01"
"state_bh_3","base.bh","Muharraq","02"
"state_bh_4","base.bh","Northern","03"
"state_bh_5","base.bh","Southern","04"

```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="gulf_cooperation_council_without_bh" model="res.country.group">
            <field name="name">Gulf Cooperation Council (GCC) without bh</field>
            <field name="country_ids" eval="[(6,0, [ref('base.sa'), ref('base.ae'), ref('base.om'), ref('base.qa'), ref('base.kw')])]"/>
        </record>
    </data>
</odoo>

```

## File: data\tax_report_full.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="l10n_bh_tax_report_full" model="account.report">
        <field name="name">Full VAT Return</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.bh"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_bh_tax_report_full_base" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="l10n_bh_tax_report_full_adjustment" model="account.report.column">
                <field name="name">Adj./App.</field>
                <field name="expression_label">adjustment</field>
            </record>
            <record id="l10n_bh_tax_report_full_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_bh_tax_report_full_vat_sale" model="account.report.line">
                <field name="name">VAT on sales</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_bh_tax_report_full_1a" model="account.report.line">
                        <field name="name">1(a). Standard rated sales at 10%</field>
                        <field name="code">bh_1a</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_1a_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1(a) B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_1a_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_1a_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1(a) T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_2" model="account.report.line">
                        <field name="name">2. Sales to registered VAT payers in other GCC states</field>
                        <field name="code">bh_2</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_2_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_3" model="account.report.line">
                        <field name="name">3. Sales subject to domestic reverse charge mechanism</field>
                        <field name="code">bh_3</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_3_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_4" model="account.report.line">
                        <field name="name">4. Zero rated domestic sales</field>
                        <field name="code">bh_4</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_4_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_4_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_5" model="account.report.line">
                        <field name="name">5. Exports</field>
                        <field name="code">bh_5</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_5_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_6" model="account.report.line">
                        <field name="name">6. Exempt sales</field>
                        <field name="code">bh_6</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_6_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_6_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_7" model="account.report.line">
                        <field name="name">7. Total sales</field>
                        <field name="code">bh_7</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_7_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    bh_1a.base +
                                    bh_2.base +
                                    bh_3.base +
                                    bh_4.base +
                                    bh_5.base +
                                    bh_6.base
                                </field>
                            </record>
                            <record id="l10n_bh_tax_report_full_7_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    bh_1a.adjustment +
                                    bh_2.adjustment +
                                    bh_3.adjustment +
                                    bh_4.adjustment +
                                    bh_5.adjustment +
                                    bh_6.adjustment
                                </field>
                            </record>
                            <record id="l10n_bh_tax_report_full_7_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    bh_1a.tax
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bh_tax_report_full_vat_purchase" model="account.report.line">
                <field name="name">VAT on purchases</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_bh_tax_report_full_8" model="account.report.line">
                        <field name="name">8(a). Standard rated domestic purchases at 10%</field>
                        <field name="code">bh_8a</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_8_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8(a) B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_8_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_8_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8(a) T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_9" model="account.report.line">
                        <field name="name">9(a). Imports subject to VAT paid at customs</field>
                        <field name="code">bh_9a</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_9_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9(a) B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_9_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_9_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9(a) T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_10" model="account.report.line">
                        <field name="name">10. Imports subject to deferral at customs</field>
                        <field name="code">bh_10</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_10_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_11" model="account.report.line">
                        <field name="name">11(a). Imports subject to VAT accounted for through reverse charge mechanism at 10%</field>
                        <field name="code">bh_11a</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_11_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11(a) B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_11_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_11_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11(a) T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_12" model="account.report.line">
                        <field name="name">12. Purchases subject to domestic reverse charge mechanism</field>
                        <field name="code">bh_12</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_12_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_12_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_12_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_13" model="account.report.line">
                        <field name="name">13. Purchases from non-registered suppliers, zero rated/ exempt purchases</field>
                        <field name="code">bh_13</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_full_13_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_14" model="account.report.line">
                        <field name="name">14. Total purchases</field>
                        <field name="code">bh_14</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_14_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    bh_8a.base +
                                    bh_9a.base +
                                    bh_10.base +
                                    bh_11a.base +
                                    bh_12.base +
                                    bh_13.base
                                </field>
                            </record>
                            <record id="l10n_bh_tax_report_full_14_adjustment" model="account.report.expression">
                                <field name="label">adjustment</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    bh_8a.adjustment +
                                    bh_9a.adjustment +
                                    bh_10.adjustment +
                                    bh_11a.adjustment +
                                    bh_12.adjustment +
                                    bh_13.adjustment
                                </field>
                            </record>
                            <record id="l10n_bh_tax_report_full_14_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    bh_8a.tax +
                                    bh_9a.tax +
                                    bh_10.tax +
                                    bh_11a.tax +
                                    bh_12.tax
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bh_tax_report_full_total_due" model="account.report.line">
                <field name="name">15. Total VAT due for current period</field>
                <field name="code">bh_15</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="l10n_bh_tax_report_full_15_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">(bh_7.tax + bh_1a.adjustment*0.1) - (bh_14.tax + 0.1*(bh_8a.adjustment + bh_9a.adjustment + bh_10.adjustment + bh_11a.adjustment + bh_12.adjustment))</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_bh_tax_report_full_16" model="account.report.line">
                        <field name="name">16. Corrections from previous period (between BHD +-5,000)</field>
                        <field name="code">bh_16</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_17" model="account.report.line">
                        <field name="name">17. VAT credit carried forward from previous period(s)</field>
                        <field name="code">bh_17</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_17_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_full_18" model="account.report.line">
                        <field name="name">18. Net VAT due (or reclaimed)</field>
                        <field name="code">bh_18</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_full_18_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">bh_15.tax + bh_16.tax - bh_17.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\tax_report_simplified.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="l10n_bh_tax_report_simplified" model="account.report">
        <field name="name">Simplified VAT Return</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.bh"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_bh_tax_report_simplified_base" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="l10n_bh_tax_report_simplified_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_bh_tax_report_simplified_1a" model="account.report.line">
                <field name="name">1(a). Standard rated sales at 10%</field>
                <field name="code">bh_simp_1a</field>
                <field name="expression_ids">
                    <record id="l10n_bh_tax_report_simplified_1a_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1(a) B</field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_1a_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1(a) T</field>
                    </record>
                </field>
            </record>
            <record id="l10n_bh_tax_report_simplified_2" model="account.report.line">
                <field name="name">2. Zero rated (including exports)</field>
                <field name="code">bh_simp_2</field>
                 <field name="foldable" eval="True"/>
                <field name="expression_ids">
                    <record id="l10n_bh_tax_report_simplified_2_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">bh_simp_2_1.base + bh_simp_2_2.base + bh_simp_2_3.base</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_bh_tax_report_simplified_2_1" model="account.report.line">
                        <field name="name">Zero rated</field>
                        <field name="code">bh_simp_2_1</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_2_1_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4 B</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_2_2" model="account.report.line">
                        <field name="name">Exports</field>
                        <field name="code">bh_simp_2_2</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_2_2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5 B</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_2_3" model="account.report.line">
                        <field name="name">GCC</field>
                        <field name="code">bh_simp_2_3</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_2_3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2 B</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bh_tax_report_simplified_3" model="account.report.line">
                <field name="name">3. Other and exempt sales</field>
                <field name="code">bh_simp_3</field>
                 <field name="foldable" eval="True"/>
                <field name="expression_ids">
                    <record id="l10n_bh_tax_report_simplified_3_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">bh_3_1.base + bh_3_2.base</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_bh_tax_report_simplified_3_1" model="account.report.line">
                        <field name="name">Sales subject to domestic reverse charge mechanism</field>
                        <field name="code">bh_3_1</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_3_1_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3 B</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_3_2" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">bh_3_2</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_3_2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6 B</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bh_tax_report_simplified_7" model="account.report.line">
                <field name="name">7.Total sales</field>
                <field name="code">bh_simp_7</field>
                <field name="expression_ids">
                    <record id="l10n_bh_tax_report_simplified_7_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">bh_simp_1a.base + bh_simp_2.base + bh_simp_3.base </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_7_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">bh_simp_1a.tax</field>
                    </record>
                </field>
            </record>
            <record id="l10n_bh_tax_report_simplified_14" model="account.report.line">
                <field name="name">14. Total purchases</field>
                <field name="code">bh_simp_14</field>
                 <field name="foldable" eval="True"/>
                <field name="expression_ids">
                    <record id="l10n_bh_tax_report_simplified_14_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">bh_simp_8a.base + bh_simp_9a.base + bh_simp_10.base + bh_simp_11a.base + bh_simp_12.base + bh_simp_13.base</field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_14_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">bh_simp_8a.tax + bh_simp_9a.tax + bh_simp_10.tax + bh_simp_11a.tax + bh_simp_12.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_bh_tax_report_simplified_8" model="account.report.line">
                        <field name="name">8(a). Standard rated domestic purchases at 10%</field>
                        <field name="code">bh_simp_8a</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_8_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8(a) B</field>
                            </record>
                            <record id="l10n_bh_tax_report_simplified_8_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8(a) T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_9" model="account.report.line">
                        <field name="name">9(a). Imports subject to VAT paid at customs</field>
                        <field name="code">bh_simp_9a</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_9_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9(a) B</field>
                            </record>
                            <record id="l10n_bh_tax_report_simplified_9_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9(a) T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_10" model="account.report.line">
                        <field name="name">10. Imports subject to deferral at customs</field>
                        <field name="code">bh_simp_10</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_simplified_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_11" model="account.report.line">
                        <field name="name">11(a). Imports subject to VAT accounted for through reverse charge mechanism at 10%</field>
                        <field name="code">bh_simp_11a</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_11_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11(a) B</field>
                            </record>
                            <record id="l10n_bh_tax_report_simplified_11_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11(a) T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_12" model="account.report.line">
                        <field name="name">12. Purchases subject to domestic reverse charge mechanism</field>
                        <field name="code">bh_simp_12</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_12_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 B</field>
                            </record>
                            <record id="l10n_bh_tax_report_simplified_12_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 T</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_13" model="account.report.line">
                        <field name="name">13. Purchases from non-registered suppliers, zero rated/ exempt purchases</field>
                        <field name="code">bh_simp_13</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13 B</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_bh_tax_report_simplified_15" model="account.report.line">
                <field name="name">15. Total VAT due for current period</field>
                <field name="code">bh_simp_15</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="l10n_bh_tax_report_simplified_15_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">bh_simp_7.tax - bh_simp_14.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_bh_tax_report_simplified_16" model="account.report.line">
                        <field name="name">16. Corrections from previous period (between BHD +-5,000)</field>
                        <field name="code">bh_simp_16</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_17" model="account.report.line">
                        <field name="name">17. VAT credit carried forward from previous period(s)</field>
                        <field name="code">bh_simp_17</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_17_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_bh_tax_report_simplified_18" model="account.report.line">
                        <field name="name">18. Net VAT due (or reclaimed)</field>
                        <field name="code">bh_simp_18</field>
                        <field name="expression_ids">
                            <record id="l10n_bh_tax_report_simplified_18_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">bh_simp_15.tax + bh_simp_16.tax - bh_simp_17.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-bh.csv

```csv
"id","name","code","account_type","reconcile","name@ar"
"bh_account_100102","Bank Suspense Account","100102","asset_current","False","حساب التعليق البنكي"
"bh_account_100103","Outstanding Receipts","100103","asset_current","False","الإيصالات المستحقة"
"bh_account_100104","Outstanding Payments","100104","asset_current","False","المدفوعات المستحقة"
"bh_account_100106","Credit cards","100106","asset_current","False","البطاقات الائتمانية"
"bh_account_100107","Post Dated Cheques Received","100107","asset_current","False","الشيكات مؤجلة الصرف المستلمة"
"bh_account_100201","Accounts Receivable","100201","asset_receivable","True","الحسابات المدينة"
"bh_account_100202","Accounts Receivable (PoS)","100202","asset_receivable","True","الحسابات المدينة (نقطة البيع)"
"bh_account_100203","Other Receivable","100203","asset_current","False","المستحقات الأخرى"
"bh_account_100301","Deposit - Office Rent","100301","asset_current","False","إيداع - إيجار المكتب"
"bh_account_100302","Deposits - Customs","100302","asset_current","False","الإيداعات - الجمارك"
"bh_account_100303","Deposit to Immigration (Visa)","100303","asset_current","False","إيداع للهجرة (فيزا)"
"bh_account_100304","Deposit Others","100304","asset_current","False","إيداع آخر"
"bh_account_100401","Prepaid Medical Insurance","100401","asset_current","False","التأمين الصحي مسبق الدفع"
"bh_account_100402","Prepaid Life Insurance","100402","asset_current","False","التأمين على الحياة مسبق الدفع"
"bh_account_100403","Prepaid Office Rent","100403","asset_current","False","إيجار المكتب مسبق الدفع"
"bh_account_100404","Prepaid Other Insurance","100404","asset_current","False","التأمينات الأخرى مسبقة الدفع"
"bh_account_100405","Prepaid License Fees","100405","asset_current","False","رسوم الرخصة مسبقة الدفع"
"bh_account_100406","Prepaid Maintenance","100406","asset_current","False","الصيانة مسبقة الدفع"
"bh_account_100407","Prepaid Employees Housing","100407","asset_current","False","سكن الموظفين مسبق الدفع"
"bh_account_100408","Prepaid Schooling Fees","100408","asset_current","False","الرسوم الدراسية مسبقة الدفع"
"bh_account_100409","Prepaid Consultancy Fees","100409","asset_current","False","الرسوم الاستشارية مسبقة الدفع"
"bh_account_100410","Prepaid Legal Fees","100410","asset_current","False","الرسوم القانونية مسبقة الدفع"
"bh_account_100411","Prepaid Sponsorship Fees","100411","asset_current","False","رسوم الكفالة مسبقة الدفع"
"bh_account_100412","Prepaid Advertisement Expenses","100412","asset_current","False","نفقات الإعلان مسبقة الدفع"
"bh_account_100413","Prepaid Bank Guarantee","100413","asset_current","False","ضمان البنك مسبق الدفع"
"bh_account_100414","Prepaid Finance charge for Loans","100414","asset_current","False","رسوم التمويل مسبقة الدفع للقروض"
"bh_account_100415","Other Prepayments","100415","asset_current","False","المدفوعات المسبقة الأخرى"
"bh_account_100416","Prepaid Expenses","100416","asset_current","False","المصروفات المدفوعة مقدما"
"bh_account_100501","Handling Difference in Inventory","100501","asset_current","False","التعامل مع الفرق في المخزون"
"bh_account_100502","Inventory Valuation","100502","asset_current","False","تقييم المخزون"
"bh_account_100503","Stock Incoming","100503","asset_current","False","المخزون الوارد"
"bh_account_100504","Stock Outgoing","100504","asset_current","False","المخزون الصادر"
"bh_account_100505","Work in Progress (Inventory)","100505","asset_current","False","العمل قيد التنفيذ (المخزون)"
"bh_account_100601","Accumulated Depreciation of Motor Vehicles","100601","asset_fixed","False","حساب الإهلاك للمركبات"
"bh_account_100602","Amortisation on Leasehold Improvement","100602","asset_fixed","False","الاستهلاك عند تحسين العقارات المستأجرة"
"bh_account_100603","Leasehold Improvement","100603","asset_fixed","False","تحسين العقارات المستأجرة"
"bh_account_100604","Furniture and Equipment","100604","asset_fixed","False","الأثاث والمعدات"
"bh_account_100605","Computer Hardware & Software","100605","asset_fixed","False","أجهزة وبرامج الحاسوب"
"bh_account_100606","Accumulated Depreciation of Furniture & Office Equipment","100606","asset_fixed","False","حساب الإهلاك للأثاث والأدوات المكتبية"
"bh_account_100607","Accumulated Depreciation of Computer Hardware & Software","100607","asset_fixed","False","حساب الإهلاك لبرامج وأجهزة الحاسوب"
"bh_account_100701","Registration of Trademarks","100701","asset_current","False","تسجيل العلامات التجارية"
"bh_account_100801","Right of use Asset (IFRS 16)","100801","asset_fixed","False","حق استخدام الأصل (IFRS 16)"
"bh_account_100802","Accumulated Depreciation Right of use Asset (IFRS 16)","100802","asset_fixed","False","حق استخدام الأصل للإهلاك المتراكم (IFRS 16)"
"bh_account_200101","Payables","200101","liability_payable","True","المبالغ مستحقة الدفع"
"bh_account_200102","Trade Payables","200102","liability_payable","True","الذمم التجارية الدائنة"
"bh_account_200103","Employees Payables","200103","liability_payable","True","المبالغ المستحقة للموظفين"
"bh_account_200104","Credit Notes to Customers","200104","liability_current","False","الإشعارات الدائنة للعملاء"
"bh_account_200201","Accrued - Salaries","200201","liability_current","False","مستحق - المرتبات"
"bh_account_200202","Accrued - Commissions","200202","liability_current","False","مستحق - العمولات"
"bh_account_200203","Accrued - Staff Bonus","200203","liability_current","False","مكافآت الموظفين المستحقة"
"bh_account_200204","Accrued Other Personnel Cost","200204","liability_current","False","تكاليف الموظفين الآخرين المستحقة"
"bh_account_200205","Accrued - Sponsorship","200205","liability_current","False","مستحق - الكفالة"
"bh_account_200301","Accrued - Utilities","200301","liability_current","False","مستحق - المرافق"
"bh_account_200302","Accrued - Telephone","200302","liability_current","False","مستحق - الهاتف"
"bh_account_200303","Accrued - Audit Fees","200303","liability_current","False","مستحق - رسوم التدقيق"
"bh_account_200304","Accrued - Office Rent","200304","liability_current","False","مستحق - إيجار المكتب"
"bh_account_200305","Accrued Others","200305","liability_current","False","المستحقات الأخرى"
"bh_account_200306","Accrued Bahrain Customs","200306","liability_current","False","جمارك البحرين المستحقة"
"bh_account_200401","Deferred income","200401","liability_current","False","الدخل المؤجل"
"bh_account_200501","Leave Tickets Provision","200501","liability_non_current","False","حكم تذاكر الطيران"
"bh_account_200502","Leave Days Provision","200502","liability_non_current","False","حكم أيام الإجازة"
"bh_account_200503","End of Service Provision","200503","liability_non_current","False","حكم نهاية الخدمة"
"bh_account_200504","Income Tax Provision","200504","liability_non_current","False","حكم ضريبة الدخل"
"bh_account_200901","VAT Input","200901","asset_current","False","مدخلات ضريبة القيمة المضافة"
"bh_account_200902","VAT Output","200902","liability_current","False","مخرجات ضريبة القيمة المضافة"
"bh_account_200903","VAT Receivable","200903","asset_non_current","False","ضريبة القيمة المضافة مستحقة الدفع"
"bh_account_200904","VAT Payable","200904","liability_non_current","False","ضريبة القيمة المضافة المستحقة"
"bh_account_200905","Tax Payable","200905","liability_current","False","الضريبة المستحقة"
"bh_account_200906","Tax Receivable","200906","asset_current","False","ضريبة مستحقة القبض"
"bh_account_200907","Recoverable VAT Input - Reverse Charge","200907","asset_current","False","مدخلات ضريبة القيمة المضافة القابلة للاسترداد - الرسوم العكسية"
"bh_account_300100","Retained Earnings","300100","equity","False","الأرباح المستبقاة"
"bh_account_300101","Undistributed Profits/Losses","300101","equity_unaffected","False","الأرباح/الخسائر غير الموزعة"
"bh_account_400100","Income Clearing Account","400100","income","False","حساب مقاصة الدخل"
"bh_account_400101","Sales Account","400101","income","False","حساب المبيعات"
"bh_account_400102","Sales of I/C","400102","income","False","المبيعات بين الشركات التابعة"
"bh_account_400103","Sales from Other Region","400103","income","False","المبيعات من منطقة أخرى"
"bh_account_400104","Management Consultancy Fees","400104","income","False","رسوم الاستشارة الإدارية"
"bh_account_400105","Advertising Income","400105","income","False","دخل الإعلان"
"bh_account_400201","Other Income","400201","income_other","False","دخل آخر"
"bh_account_400301","Gain on Difference on Exchange","400301","income_other","False","أرباح فرق صرف العملة"
"bh_account_400302","Cash Difference Gain","400302","income_other","False","أرباح فرق النقد"
"bh_account_400303","Excess In Till","400303","income_other","False","الفائض في صندوق النقود"
"bh_account_400304","Cash Discount Gain","400304","income_other","False","مكاسب الخصم النقدي"
"bh_account_500101","Cost of Goods Sold in Trading","500101","expense_direct_cost","False","تكاليف البضائع المباعة في التجارة"
"bh_account_500102","Cost Of Goods Sold I/C Sales","500102","expense_direct_cost","False","تكاليف البضائع المباعة - المبيعات بين الشركات التابعة"
"bh_account_500200","Expense Clearing Account","500200","expense","False","حساب مقاصة النفقات"
"bh_account_500201","Medical Insurance","500201","expense","False","التأمين الصحي"
"bh_account_500202","End of Service Indemnity","500202","expense","False","تعويض نهاية الخدمة"
"bh_account_500203","Sponsorship Fees","500203","expense","False","رسوم الكفالة"
"bh_account_500301","Basic Salary","500301","expense","False","الراتب الأساسي"
"bh_account_500302","Housing Allowance","500302","expense","False","بدل السكن"
"bh_account_500303","Transportation Allowance","500303","expense","False","بدل المواصلات"
"bh_account_500304","Leave Ticket","500304","expense","False","تذكرة الطيران"
"bh_account_500305","Leave Salary","500305","expense","False","راتب الإجازة"
"bh_account_500306","Sales Commission","500306","expense","False","عمولة المبيعات"
"bh_account_500307","Visa Expenses","500307","expense","False","نفقات الفيزا"
"bh_account_500308","Staff Other Allowances","500308","expense","False","نفقات الموظفين الأخرى"
"bh_account_500309","Air tickets","500309","expense","False","تذاكر الطيران"
"bh_account_500401","Office Rent","500401","expense","False","إيجار المكتب"
"bh_account_500402","Warehouse Rent","500402","expense","False","إيجار المستودع"
"bh_account_500403","Water & Electricity","500403","expense","False","الماء والكهرباء"
"bh_account_500404","Other Utility Charges","500404","expense","False","رسوم المرافق الأخرى"
"bh_account_500501","Audit Fees","500501","expense","False","رسوم التدقيق"
"bh_account_500502","Legal fees","500502","expense","False","الرسوم القانونية"
"bh_account_500503","Trade License Fees","500503","expense","False","رسوم الرخصة التجارية"
"bh_account_500504","Others - Professional Fees","500504","expense","False","غير ذلك - الرسوم المهنية"
"bh_account_500505","Insurance","500505","expense","False","التأمين"
"bh_account_500506","Previous Year Adjustments Account","500506","expense","False","حساب تعديلات العام الماضي"
"bh_account_500601","Credit Card Charges","500601","expense","False","رسوم البطاقة الائتمانية"
"bh_account_500602","Other Bank Charges","500602","expense","False","الرسوم البنكية الأخرى"
"bh_account_500603","Bank Finance & Loan Charges","500603","expense","False","رسوم القروض والتمويل البنكي"
"bh_account_500651","Income Tax Expense","500651","expense","False","نفقات ضريبة الدخل"
"bh_account_500701","Other - Advertising Expenses","500701","expense","False","غير ذلك - نفقات الإعلان"
"bh_account_500702","Training","500702","expense","False","التدريب"
"bh_account_500703","Consultancy Fees","500703","expense","False","الرسوم الاستشارية"
"bh_account_500801","Amortisation on Leasehold Improvement","500801","expense_depreciation","False","الاستهلاك عند تحسين العقارات المستأجرة"
"bh_account_500802","Vehicle Expenses","500802","expense_depreciation","False","نفقات المركبات"
"bh_account_500803","Depreciation of Motor Vehicles","500803","expense_depreciation","False","إهلاك المركبات"
"bh_account_500804","Depreciation of Furniture & Office Equipment","500804","expense_depreciation","False","إهلاك الأثاث والمعدات المكتبية"
"bh_account_500805","Depreciation of Computer Hard & Soft","500805","expense_depreciation","False","إهلاك أجهزة وبرامج الحاسوب"
"bh_account_500851","Depreciation on Right of use Asset (IFRS 16)","500851","expense_depreciation","False","إهلاك حق استخدام الأصل (IFRS 16)"
"bh_account_500901","Loss on Fixed Assets Disposal","500901","expense","False","خسائر التصرف في الأصول الثابتة"
"bh_account_500902","Cash Shortage","500902","expense","False","القصور النقدي"
"bh_account_500903","Loss on Difference on Exchange","500903","expense","False","حسائر فرق صرف العملة"
"bh_account_500904","Write Off Receivables & Payables","500904","expense","False","شطب الحسابات المدينة والدائنة"
"bh_account_500905","Write Off Inventory","500905","expense","False","شطب المخزون"
"bh_account_500906","Others - Provision & Write Off","500906","expense","False","غير ذلك - المَحافظ والتعديلات"
"bh_account_500907","Others","500907","expense","False","غير ذلك"
"bh_account_500908","Other Non-Operating Expenses","500908","expense","False","النفقات الأخرى غير التشغيلية"
"bh_account_500909","Cash Difference Loss","500909","expense","False","خسائر فريق النقد"
"bh_account_501101","Telephone","501101","expense","False","الهاتف"
"bh_account_501102","Others - Communication","501102","expense","False","غير ذلك - التواصل"
"bh_account_501103","Maintenance","501103","expense","False","الصيانة"
"bh_account_501104","Security & Guard","501104","expense","False","الأمن والحراسة"
"bh_account_501105","Cleaning","501105","expense","False","التنظيف"
"bh_account_501106","Others - Office Various Expenses","501106","expense","False","غير ذلك - نفقات المكتب المختلفة"
"bh_account_501107","Cash Discount Loss","501107","expense","False","خسارة الخصم النقدي"

```

## File: data\template\account.fiscal.position-bh.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@ar"
"fiscal_position_template_dom","1","Bahrain","1","","base.bh","","","","البحرين"
"fiscal_position_non_bahrain_gcc","2","Non-Bahrain (GCC)","1","","","l10n_bh.gulf_cooperation_council_without_bh","l10n_bh_purchase_vat_10","l10n_bh_purchase_vat_10_ex","غير البحرين (دول مجلس التعاون الخليجي)"
"","","","","","","","l10n_bh_sale_vat_10","l10n_bh_sale_vat_0_gcc",""
"fiscal_position_non_bahrain","3","Non-Bahrain","1","","","","l10n_bh_purchase_vat_10","l10n_bh_purchase_vat_10_ex","غير البحرين"
"","","","","","","","l10n_bh_sale_vat_10","l10n_bh_sale_0_ex",""

```

## File: data\template\account.tax-bh.csv

```csv
"id","sequence","name","description","invoice_label","price_include","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@ar"
"l10n_bh_sale_vat_10","1","10%","","10%","False","10.0","percent","sale","bh_tax_group_10","","base","invoice","+1(a) B","",""
"","","","","","","","","","","","tax","invoice","+1(a) T","bh_account_200902",""
"","","","","","","","","","","","base","refund","-1(a) B","",""
"","","","","","","","","","","","tax","refund","-1(a) T","bh_account_200902",""
"l10n_bh_purchase_vat_10","2","10%","","10%","False","10.0","percent","purchase","bh_tax_group_10","","base","invoice","+8(a) B","",""
"","","","","","","","","","","","tax","invoice","+8(a) T","bh_account_200901",""
"","","","","","","","","","","","base","refund","-8(a) B","",""
"","","","","","","","","","","","tax","refund","-8(a) T","bh_account_200901",""
"l10n_bh_purchase_vat_10_ex_df","3","10% EX DF","Subject to Deferral at Customs","0% Deferred Import","False","10.0","percent","purchase","bh_tax_group_10","","base","invoice","+10 B","","خاضعة للتأجيل في الجمارك"
"","","","","","","","","","","","tax","invoice","+10 T","bh_account_200901",""
"","","","","","","","","","","","base","refund","-10 B","",""
"","","","","","","","","","","","tax","refund","-10 T","bh_account_200901",""
"l10n_bh_purchase_vat_10_ex","4","10% EX","Import","10% Import","False","10.0","percent","purchase","bh_tax_group_10","","base","invoice","+9(a) B","","الاستيراد"
"","","","","","","","","","","","tax","invoice","+9(a) T","bh_account_200901",""
"","","","","","","","","","","","base","refund","-9(a) B","",""
"","","","","","","","","","","","tax","refund","-9(a) T","bh_account_200901",""
"l10n_bh_sale_vat_0_gcc","5","0% GCC","GCC","0% GCC","False","0.0","percent","sale","bh_tax_group_0","","base","invoice","+2 B","","دول مجلس التعاون الخليجي"
"","","","","","","","","","","","tax","invoice","","bh_account_200902",""
"","","","","","","","","","","","base","refund","-2 B","",""
"","","","","","","","","","","","tax","refund","","bh_account_200902",""
"l10n_bh_sale_0","6","0%","","0%","False","0.0","percent","sale","bh_tax_group_0","","base","invoice","+4 B","",""
"","","","","","","","","","","","tax","invoice","","bh_account_200902",""
"","","","","","","","","","","","base","refund","-4 B","",""
"","","","","","","","","","","","tax","refund","","bh_account_200902",""
"l10n_bh_purchase_0","7","0%","","0%","False","0.0","percent","purchase","bh_tax_group_0","","base","invoice","+13 B","",""
"","","","","","","","","","","","tax","invoice","","bh_account_200901",""
"","","","","","","","","","","","base","refund","-13 B","",""
"","","","","","","","","","","","tax","refund","","bh_account_200901",""
"l10n_bh_sale_0_ex","8","0% EX","Zero-rated exports","0%","False","0.0","percent","sale","bh_tax_group_0","","base","invoice","+5 B","","الصادرات ذات التصنيف الصفري"
"","","","","","","","","","","","tax","invoice","","bh_account_200902",""
"","","","","","","","","","","","base","refund","-5 B","",""
"","","","","","","","","","","","tax","refund","","bh_account_200902",""
"l10n_bh_sale_0_EXT","9","0% EXT","Exempt","Exempt","False","0.0","percent","sale","bh_tax_group_exempt","","base","invoice","+6 B","","معفاة"
"","","","","","","","","","","","tax","invoice","","bh_account_200902",""
"","","","","","","","","","","","base","refund","-6 B","",""
"","","","","","","","","","","","tax","refund","","bh_account_200902",""
"l10n_bh_purchase_0_EXT","10","0% EXT","Exempt","Exempt","False","0.0","percent","purchase","bh_tax_group_exempt","","base","invoice","+13 B","","معفاة"
"","","","","","","","","","","","tax","invoice","","bh_account_200901",""
"","","","","","","","","","","","base","refund","-13 B","",""
"","","","","","","","","","","","tax","refund","","bh_account_200901",""
"l10n_bh_purchase_10_RC_D","11","10% RC D","Domestic supplies subject to reverse charge provisions","10% Domestic Reverse Charge","False","10.0","percent","purchase","bh_tax_group_10","","base","invoice","+12 B","","التوريدات المحلية الخاضعة لأحكام الرسوم العكسية"
"","","","","","","","","","","","tax","invoice","-12 T||+12 T","bh_account_200907",""
"","","","","","","","","","","-100","tax","invoice","","bh_account_200902",""
"","","","","","","","","","","","base","refund","-12 B","",""
"","","","","","","","","","","","tax","refund","-12 T||+12 T","bh_account_200907",""
"","","","","","","","","","","-100","tax","refund","","bh_account_200902",""
"l10n_bh_purchase_10_RC","12","10% RC","Supplies subject to reverse charge provisions","10% Reverse Charge","False","10.0","percent","purchase","bh_tax_group_rc_10","","base","invoice","+11(a) B","","التوريدات الخاضعة لأحكام الرسوم العكسية"
"","","","","","","","","","","","tax","invoice","-11(a) T||+11(a) T","bh_account_200907",""
"","","","","","","","","","","-100","tax","invoice","","bh_account_200902",""
"","","","","","","","","","","","base","refund","-11(a) B","",""
"","","","","","","","","","","","tax","refund","-11(a) T||+11(a) T","bh_account_200907",""
"","","","","","","","","","","-100","tax","refund","","bh_account_200902",""
"l10n_bh_sale_10_RC","13","10% RC D","Domestic supplies subject to reverse charge provisions","10% Reverse Charge","False","0.0","percent","sale","bh_tax_group_rc_10","","base","invoice","+3 B","","التوريدات المحلية الخاضعة لأحكام الرسوم العكسية"
"","","","","","","","","","","","tax","invoice","","bh_account_200902",""
"","","","","","","","","","","","base","refund","-3 B","",""
"","","","","","","","","","","","tax","refund","","bh_account_200902",""

```

## File: data\template\account.tax.group-bh.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@ar"
"bh_tax_group_10","VAT 10%","base.bh","bh_account_200904","bh_account_200903","10% ضريبة"
"bh_tax_group_0","VAT 0%","base.bh","bh_account_200904","bh_account_200903","0% ضريبة"
"bh_tax_group_exempt","VAT Exempt","base.bh","bh_account_200904","bh_account_200903","معفاة من ضريبة القيمة المضافة"
"bh_tax_group_rc_10","Reverse Charge VAT 10%","base.bh","bh_account_200904","bh_account_200903","الرسوم العكسية لضريبة القيمة المضافة 10%"

```

## File: models\template_bh.py

```python
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bh')
    def _get_bh_template_data(self):
        return {
            'property_account_receivable_id': 'bh_account_100201',
            'property_account_payable_id': 'bh_account_200101',
            'property_account_expense_categ_id': 'bh_account_500101',
            'property_account_income_categ_id': 'bh_account_400101',
            'property_account_expense_id': 'bh_account_500101',
            'property_account_income_id': 'bh_account_400101',
            'property_stock_valuation_account_id': 'bh_account_100502',
            'property_stock_account_input_categ_id': 'bh_account_100503',
            'property_stock_account_output_categ_id': 'bh_account_100504',
            'property_stock_account_production_cost_id': 'bh_account_100505',
            'code_digits': '6',
        }

    @template('bh', 'res.company')
    def _get_bh_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.bh',
                'bank_account_code_prefix': '1000',
                'cash_account_code_prefix': '1009',
                'transfer_account_code_prefix': '1001',
                'account_default_pos_receivable_account_id': 'bh_account_100202',
                'income_currency_exchange_account_id': 'bh_account_400301',
                'expense_currency_exchange_account_id': 'bh_account_500903',
                'account_journal_suspense_account_id': 'bh_account_100102',
                'account_journal_early_pay_discount_loss_account_id': 'bh_account_501107',
                'account_journal_early_pay_discount_gain_account_id': 'bh_account_400304',
                'default_cash_difference_income_account_id': 'bh_account_400302',
                'default_cash_difference_expense_account_id': 'bh_account_500909',
                'deferred_expense_account_id': 'bh_account_100416',
                'deferred_revenue_account_id': 'bh_account_200401',
            },
        }

```

## File: models\__init__.py

```python
from . import template_bh

```

