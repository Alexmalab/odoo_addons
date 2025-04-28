# Odoo Module: l10n_be

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import demo

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Belgium - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/belgium.html',
    'version': '2.0',
    'icon': '/account/static/description/l10n.png',
    'countries': ['be'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Belgium in Odoo.
==============================================================================

After installing this module, the Configuration wizard for accounting is launched.
    * We have the account templates which can be helpful to generate Charts of Accounts.
    * On that particular wizard, you will be asked to pass the name of the company,
      the chart template to follow, the no. of digits to generate, the code for your
      account and bank account, currency to create journals.

Thus, the pure copy of Chart Template is generated.

Wizards provided by this module:
--------------------------------
    * Partner VAT Intra: Enlist the partners with their related VAT and invoiced
      amounts. Prepares an XML file format.

        **Path to access:** Invoicing/Reporting/Legal Reports/Belgium Statements/Partner VAT Intra
    * Periodical VAT Declaration: Prepares an XML file for Vat Declaration of
      the Main company of the User currently Logged in.

        **Path to access:** Invoicing/Reporting/Legal Reports/Belgium Statements/Periodical VAT Declaration
    * Annual Listing Of VAT-Subjected Customers: Prepares an XML file for Vat
      Declaration of the Main company of the User currently Logged in Based on
      Fiscal year.

        **Path to access:** Invoicing/Reporting/Legal Reports/Belgium Statements/Annual Listing Of VAT-Subjected Customers

    """,
    'author': 'Noviat, Odoo S.A.',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_tax_report_data.xml',
        'data/l10n_be_sequence_data.xml',
        'data/menuitem_data.xml',
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
    <record id="tax_report_vat" model="account.report">
        <field name="name">VAT Return</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.be"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_vat_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_title_operations" model="account.report.line">
                <field name="name">Operations</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_title_operations_sortie" model="account.report.line">
                        <field name="name">II Outgoing</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="tax_report_line_00" model="account.report.line">
                                <field name="name">00 - Operations subject to a special regulation</field>
                                <field name="code">c00</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_00_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">00</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_01" model="account.report.line">
                                <field name="name">01 - Operations subject to 6% VAT</field>
                                <field name="code">c01</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_01_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">01</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_02" model="account.report.line">
                                <field name="name">02 - Operations subject to 12% VAT</field>
                                <field name="code">c02</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_02_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">02</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_03" model="account.report.line">
                                <field name="name">03 - Operations subject to 21% VAT</field>
                                <field name="code">c03</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_03_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">03</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_44" model="account.report.line">
                                <field name="name">44 - Intra-Community services</field>
                                <field name="code">c44</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_44_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">44</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_45" model="account.report.line">
                                <field name="name">45 - Operations subject to VAT due by the co-contractor</field>
                                <field name="code">c45</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_45_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">45</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_title_operations_sortie_46" model="account.report.line">
                                <field name="name">46 - Exempted intra-Community deliveries and ABC sales</field>
                                <field name="code">c46</field>
                                <field name="aggregation_formula">c46L.balance + c46T.balance</field>
                                <field name="foldable" eval="True"/>
                                <field name="children_ids">
                                    <record id="tax_report_line_46L" model="account.report.line">
                                        <field name="name">46L - Exempted intra-Community deliveries</field>
                                        <field name="code">c46L</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_46L_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">46L</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_46T" model="account.report.line">
                                        <field name="name">46T - ABC sales</field>
                                        <field name="code">c46T</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_46T_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">46T</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_47" model="account.report.line">
                                <field name="name">47 - Other exempted operations and operations carried out abroad</field>
                                <field name="code">c47</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_47_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">47</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_title_operations_sortie_48" model="account.report.line">
                                <field name="name">48 - Credit notes for operations in grids [44] and [46]</field>
                                <field name="code">c48</field>
                                <field name="aggregation_formula">c48s44.balance + c48s46L.balance + c48s46T.balance</field>
                                <field name="foldable" eval="True"/>
                                <field name="children_ids">
                                    <record id="tax_report_line_48s44" model="account.report.line">
                                        <field name="name">48s44 - Credit notes for operations in grid [44]</field>
                                        <field name="code">c48s44</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_48s44_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">48s44</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_48s46L" model="account.report.line">
                                        <field name="name">48s46L - Credit notes for operations in grid [46L]</field>
                                        <field name="code">c48s46L</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_48s46L_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">48s46L</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_48s46T" model="account.report.line">
                                        <field name="name">48s46T - Credit notes for operations in grid [46T]</field>
                                        <field name="code">c48s46T</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_48s46T_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">48s46T</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_49" model="account.report.line">
                                <field name="name">49 - Credit notes for other operations in part II</field>
                                <field name="code">c49</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_49_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">49</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_title_operations_entree" model="account.report.line">
                        <field name="name">III Incoming</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="tax_report_line_81" model="account.report.line">
                                <field name="name">81 - Trade goods, raw materials and consumables</field>
                                <field name="code">c81</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_81_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">81</field>
                                    </record>
                                    <record id="tax_report_line_81_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_line_81_balance_unbound" model="account.report.expression">
                                        <field name="label">balance_unbound</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c81._applied_carryover_balance + c81.tag</field>
                                    </record>
                                    <record id="tax_report_line_81_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c81.balance_unbound</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_81_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c81.balance_unbound</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_82" model="account.report.line">
                                <field name="name">82 - Services and miscellaneous goods</field>
                                <field name="code">c82</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_82_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">82</field>
                                    </record>
                                    <record id="tax_report_line_82_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_line_82_balance_unbound" model="account.report.expression">
                                        <field name="label">balance_unbound</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c82._applied_carryover_balance + c82.tag</field>
                                    </record>
                                    <record id="tax_report_line_82_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c82.balance_unbound</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_82_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c82.balance_unbound</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_83" model="account.report.line">
                                <field name="name">83 - Investment goods</field>
                                <field name="code">c83</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_83_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">83</field>
                                    </record>
                                    <record id="tax_report_line_83_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_line_83_balance_unbound" model="account.report.expression">
                                        <field name="label">balance_unbound</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c83._applied_carryover_balance + c83.tag</field>
                                    </record>
                                    <record id="tax_report_line_83_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c83.balance_unbound</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_83_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c83.balance_unbound</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_84" model="account.report.line">
                                <field name="name">84 - Credit notes for operations in grids [86] and [88]</field>
                                <field name="code">c84</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_84_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">84</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_85" model="account.report.line">
                                <field name="name">85 - Credit notes received relating to other operations in part III</field>
                                <field name="code">c85</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_85_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">85</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_86" model="account.report.line">
                                <field name="name">86 - Intra-Community acquisitions and ABC sales</field>
                                <field name="code">c86</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_86_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">86</field>
                                    </record>
                                    <record id="tax_report_line_86_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_line_86_balance_unbound" model="account.report.expression">
                                        <field name="label">balance_unbound</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c86._applied_carryover_balance + c86.tag</field>
                                    </record>
                                    <record id="tax_report_line_86_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c86.balance_unbound</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_86_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c86.balance_unbound</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_87" model="account.report.line">
                                <field name="name">87 - Other operations subject to VAT</field>
                                <field name="code">c87</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_87_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">87</field>
                                    </record>
                                    <record id="tax_report_line_87_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_line_87_balance_unbound" model="account.report.expression">
                                        <field name="label">balance_unbound</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c87._applied_carryover_balance + c87.tag</field>
                                    </record>
                                    <record id="tax_report_line_87_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c87.balance_unbound</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_87_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c87.balance_unbound</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_88" model="account.report.line">
                                <field name="name">88 - Intra-Community services with reverse charge</field>
                                <field name="code">c88</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_88_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">88</field>
                                    </record>
                                    <record id="tax_report_line_88_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_line_88_balance_unbound" model="account.report.expression">
                                        <field name="label">balance_unbound</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c88._applied_carryover_balance + c88.tag</field>
                                    </record>
                                    <record id="tax_report_line_88_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c88.balance_unbound</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_88_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c88.balance_unbound</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_taxes" model="account.report.line">
                <field name="name">Taxes</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_title_taxes_dues" model="account.report.line">
                        <field name="name">IV Due</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="tax_report_line_54" model="account.report.line">
                                <field name="name">54 - VAT on operations in grids [01], [02] and [03]</field>
                                <field name="code">c54</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_54_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">54</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_55" model="account.report.line">
                                <field name="name">55 - VAT on operations in grids [86] and [88]</field>
                                <field name="code">c55</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_55_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">55</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_56" model="account.report.line">
                                <field name="name">56 - VAT on operations in grid [87], with the exception of imports with reverse charge</field>
                                <field name="code">c56</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_56_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">56</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_57" model="account.report.line">
                                <field name="name">57 - VAT on import with reverse charge</field>
                                <field name="code">c57</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_57_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">57</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_61" model="account.report.line">
                                <field name="name">61 - Various VAT regularizations in favor of the State</field>
                                <field name="code">c61</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_61_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">61</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_63" model="account.report.line">
                                <field name="name">63 - VAT to be paid back on credit notes received</field>
                                <field name="code">c63</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_63_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">63</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_title_taxes_deductibles" model="account.report.line">
                        <field name="name">V Deductible</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="tax_report_line_59" model="account.report.line">
                                <field name="name">59 - Deductible VAT</field>
                                <field name="code">c59</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_59_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">59</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_62" model="account.report.line">
                                <field name="name">62 - Various VAT regularizations in favor of the declarant</field>
                                <field name="code">c62</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_62_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">62</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_64" model="account.report.line">
                                <field name="name">64 - VAT to be recovered on credit notes issued</field>
                                <field name="code">c64</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_64_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">64</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_title_taxes_soldes" model="account.report.line">
                        <field name="name">VI Balance</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="tax_report_line_71" model="account.report.line">
                                <field name="name">71 - Taxes due to the State</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_71_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(c54.balance + c55.balance + c56.balance + c57.balance + c61.balance + c63.balance) - (c59.balance + c62.balance + c64.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_72" model="account.report.line">
                                <field name="name">72 - Amount owed by the State</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_72_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(c59.balance + c62.balance + c64.balance) - (c54.balance + c55.balance + c56.balance + c57.balance + c61.balance + c63.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_be_sequence_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!--
    Sequences for declarantnum will be used in wizard for "Listing of VAT Customers"..in creating xml file
        -->
    <record model="ir.sequence" id="seq_declarantnum">
        <field name="name">Declarantnum</field>
        <field name="code">declarantnum</field>
        <field name="padding">5</field>
        <field name="company_id" eval="False"/>
    </record>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_be_statements_menu" name="Belgium" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\template\account.account-be.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@fr","name@nl","name@de"
"a120","Revaluation Surpluses on Intangible Fixed Assets","120","equity","","False","Plus-values de réévaluation sur immobilisations incorporelles","Herwaarderingsmeerwaarden op immateriële vaste activa","Neubewertungsrücklagen auf immaterielle Anlagewerte"
"a121","Revaluation Surpluses on Tangible Fixed Assets","121","equity","","False","Plus-values de réévaluation sur immobilisations corporelles","Herwaarderingsmeerwaarden op materiële vaste activa","Neubewertungsrücklagen auf Sachanlagen"
"a122","Revaluation Surpluses on Financial Fixed Assets","122","equity","","False","Plus-values de réévaluation sur immobilisations financières","Herwaarderingsmeerwaarden op financiële vaste activa","Neubewertungsrücklagen auf Finanzanlagen"
"a124","Write-Back of Amounts Written Down on Current Investments","124","equity","","False","Reprises de réductions de valeur sur placements de trésorerie","Terugneming van waardeverminderingen op geldbeleggingen","Rücknahmen von Wertminderungen auf Geldanlagen"
"a132","Untaxed Reserves","132","equity","","False","Réserves immunisées","Belastingvrije reserves","Steuerfreie Rücklagen"
"a140","Profit Brought Forward","140","equity","","False","Bénéfice reporté","Overgedragen winst","Gewinnvortrag"
"a141","Loss Brought Forward (-)","141","equity","","False","Perte reportée (-)","Overgedragen verlies (-)","Verlustvortrag (-)"
"a160","Provisions for Pensions and Similar Obligations","160","liability_non_current","","False","Provisions pour pensions et obligations similaires","Voorzieningen voor pensioenen en soortgelijke verplichtingen","Rückstellungen für Pensionen und ähnliche Verpflichtungen"
"a161","Provisions for Taxation","161","liability_non_current","","False","Provisions pour charges fiscales","Voorzieningen voor belastingen","Rückstellungen für Steuern"
"a162","Provisions for Major Repairs and Maintenance","162","liability_non_current","","False","Provisions pour grosses réparations et gros entretien","Voorzieningen voor grote herstellings- en onderhoudswerken","Rückstellungen für große Reparaturen und Instandhaltungsarbeiten"
"a163","Provisions for Environmental Obligations","163","liability_non_current","","False","Provisions pour obligations environnementales","Voorzieningen voor milieuverplichtingen","Rückstellungen für Umweltschutzverpflichtungen"
"a164","Provisions for Other Liabilities and Charges","164","liability_non_current","","False","Provisions pour autres risques et charges","Voorzieningen voor overige risico's en kosten","Rückstellungen für sonstige Risiken und Aufwendungen"
"a172","Leasing and Other Similar Obligations (> 1 Year)","172","liability_non_current","","False","Dettes de location-financement et assimilées (> 1 an)","Leasingschulden en soortgelijke schulden (> 1 jaar)","Verbindlichkeiten aufgrund von Leasing und ähnlichen Verträgen (> 1 Jahr)"
"a1730","Credit Institutions - Current Account Payable (> 1 Year)","1730","liability_non_current","","False","Etablissements de crédit - Dettes en compte (> 1 an)","Kredietinstellingen - Schulden op rekening (> 1 jaar)","Kreditinstitute - Kontoverbindlichkeiten (> 1 Jahr)"
"a1731","Credit Institutions - Promissory Notes (> 1 Year)","1731","liability_non_current","","False","Etablissements de crédit - Promesses (> 1 an)","Kredietinstellingen - Promessen (> 1 jaar)","Kreditinstitute - Solawechsel (> 1 Jahr)"
"a1732","Credit Institutions - Bank Acceptances (> 1 Year)","1732","liability_non_current","","False","Etablissements de crédit - Crédits d'acceptation (> 1 an)","Kredietinstellingen - Acceptkredieten (> 1 jaar)","Kreditinstitute - Akzeptkredite (> 1 Jahr)"
"a174","Other Loans (> 1 Year)","174","liability_non_current","","False","Autres emprunts (> 1 an)","Overige leningen (> 1 jaar)","Sonstige Anleihen (> 1 Jahr)"
"a1750","Suppliers (> 1 Year)","1750","liability_non_current","","False","Fournisseurs (> 1 an)","Leveranciers (> 1 jaar)","Lieferanten (> 1 Jahr)"
"a1751","Bills of Exchange Payable (> 1 Year)","1751","liability_non_current","","False","Effets à payer (> 1 an)","Te betalen wissels (> 1 jaar)","Verbindlichkeiten aus Wechseln (> 1 Jahr)"
"a176","Advance Payments on Contracts in Progress (> 1 Year)","176","liability_non_current","","False","Acomptes sur commandes (> 1 an)","Vooruitbetalingen op bestellingen (> 1 jaar)","Erhaltene Anzahlungen auf Bestellungen (> 1 Jahr)"
"a178","Guarantees Received in Cash (> 1 Year)","178","liability_non_current","","False","Cautionnements reçus en numéraire (> 1 an)","Borgtochten ontvangen in contanten (> 1 jaar)","In Geldmittel erhaltene Kautionen (> 1 Jahr)"
"a201","Loan Issue Expenses","201","asset_non_current","","False","Frais d'émission d'emprunts","Kosten bij uitgifte van leningen","Kosten der Emission von Anleihen"
"a202","Other Formation Expenses","202","asset_non_current","","False","Autres frais d'établissement","Overige oprichtingskosten","Andere Errichtungs- und Erweiterungsaufwendungen"
"a204","Restructuring Costs","204","asset_non_current","","False","Frais de restructuration","Herstructureringskosten","Restrukturierungskosten"
"a210000","Research and Development Costs - Acquisition Value","210000","asset_non_current","","False","Frais de recherche et de développement - Valeur d'acquisition","Kosten van onderzoek en ontwikkeling - Aanschaffingswaarde","Forschungs- und Entwicklungskosten - Anschaffungswert"
"a210008","Research and Development Costs - Capital Profits - Recorded","210008","asset_non_current","","False","Frais de recherche et de développement - Plus-values actées","Kosten van onderzoek en ontwikkeling - Geboekte meerwaarden","Forschungs- und Entwicklungskosten - Gebuchte Mehrwerte"
"a210009","Research and Development Costs - Depreciations and Amounts Written Down - Recorded","210009","asset_non_current","","False","Frais de recherche et de développement - Amortissements ou réductions de valeur actés","Kosten van onderzoek en ontwikkeling - Geboekte afschrijvingen of waardeverminderingen","Forschungs- und Entwicklungskosten - Gebuchte Abschreibungen oder Wertminderungen"
"a211000","Concessions, Patents, Licences, Know-How, Brands and Similar Rights - Acquisition Value","211000","asset_non_current","","False","Concessions, brevets, licences, savoir-faire, marques et droits similaires - Valeur d'acquisition","Concessies, octrooien, licenties, know-how, merken en soortgelijke rechten - Aanschaffingswaarde","Konzessionen, Patente, Lizenzen, Know-how, Warenzeichen und ähnliche Rechte - Anschaffungswert"
"a211008","Concessions, Patents, Licences, Know-How, Brands and Similar Rights - Capital Profits - Recorded","211008","asset_non_current","","False","Concessions, brevets, licences, savoir-faire, marques et droits similaires - Plus-values actées","Concessies, octrooien, licenties, know-how, merken en soortgelijke rechten - Geboekte meerwaarden","Konzessionen, Patente, Lizenzen, Know-how, Warenzeichen und ähnliche Rechte - Gebuchte Mehrwerte"
"a211009","Concessions, Patents, Licences, Know-How, Brands and Similar Rights - Depreciations and Amounts Written Down - Recorded","211009","asset_non_current","","False","Concessions, brevets, licences, savoir-faire, marques et droits similaires - Amortissements ou réductions de valeur actés","Concessies, octrooien, licenties, know-how, merken en soortgelijke rechten - Geboekte afschrijvingen of waardeverminderingen","Konzessionen, Patente, Lizenzen, Know-how, Warenzeichen und ähnliche Rechte - Gebuchte Abschreibungen oder Wertminderungen"
"a212000","Goodwill - Acquisition Value","212000","asset_non_current","","False","Goodwill - Valeur d'acquisition","Goodwill - Aanschaffingswaarde","Goodwill - Anschaffungswert"
"a212008","Goodwill - Capital Profits - Recorded","212008","asset_non_current","","False","Goodwill - Plus-values actées","Goodwill - Geboekte meerwaarden","Goodwill - Gebuchte Mehrwerte"
"a212009","Goodwill - Depreciations and Amounts Written Down - Recorded","212009","asset_non_current","","False","Goodwill - Amortissements ou réductions de valeur actés","Goodwill - Geboekte afschrijvingen of waardeverminderingen","Goodwill - Gebuchte Abschreibungen oder Wertminderungen"
"a213","Advance Payments for Intangible Fixed Assets","213","asset_non_current","","False","Acomptes versés pour immobilisations incorporelles","Vooruitbetalingen voor immateriële vaste activa","Geleistete Anzahlungen für immaterielle Anlagewerte"
"a220000","Land - Acquisition Value","220000","asset_fixed","","False","Terrains - Valeur d'acquisition","Terreinen - Aanschaffingswaarde","Grundstücke - Anschaffungswert"
"a220008","Land - Capital Profits - Recorded","220008","asset_fixed","","False","Terrains - Plus-values actées","Terreinen - Geboekte meerwaarden","Grundstücke - Gebuchte Mehrwerte"
"a220009","Land - Depreciations and Amounts Written Down - Recorded","220009","asset_fixed","","False","Terrains - Amortissements ou réductions de valeur actés","Terreinen - Geboekte afschrijvingen of waardeverminderingen","Grundstücke - Gebuchte Abschreibungen oder Wertminderungen"
"a221000","Buildings - Acquisition Value","221000","asset_fixed","","False","Constructions - Valeur d'acquisition","Gebouwen - Aanschaffingswaarde","Bauten - Anschaffungswert"
"a221008","Buildings - Capital Profits - Recorded","221008","asset_fixed","","False","Constructions - Plus-values actées","Gebouwen - Geboekte meerwaarden","Bauten - Gebuchte Mehrwerte"
"a221009","Buildings - Depreciations and Amounts Written Down - Recorded","221009","asset_fixed","","False","Constructions - Amortissements ou réductions de valeur actés","Gebouwen - Geboekte afschrijvingen of waardeverminderingen","Bauten - Gebuchte Abschreibungen oder Wertminderungen"
"a222000","Developed Land - Acquisition Value","222000","asset_fixed","","False","Terrains bâtis - Valeur d'acquisition","Bebouwde terreinen - Aanschaffingswaarde","Bebaute Grundstücke - Anschaffungswert"
"a222008","Developed Land - Capital Profits - Recorded","222008","asset_fixed","","False","Terrains bâtis - Plus-values actées","Bebouwde terreinen - Geboekte meerwaarden","Bebaute Grundstücke - Gebuchte Mehrwerte"
"a222009","Developed Land - Depreciations and Amounts Written Down - Recorded","222009","asset_fixed","","False","Terrains bâtis - Amortissements ou réductions de valeur actés","Bebouwde terreinen - Geboekte afschrijvingen of waardeverminderingen","Bebaute Grundstücke - Gebuchte Abschreibungen oder Wertminderungen"
"a223000","Other Rights to Immovable Property - Acquisition Value","223000","asset_fixed","","False","Autres droits réels sur des immeubles - Valeur d'acquisition","Overige zakelijke rechten op onroerende goederen - Aanschaffingswaarde","Sonstige dingliche Rechte an unbeweglichen Gütern - Anschaffungswert"
"a223008","Other Rights to Immovable Property - Capital Profits - Recorded","223008","asset_fixed","","False","Autres droits réels sur des immeubles - Plus-values actées","Overige zakelijke rechten op onroerende goederen - Geboekte meerwaarden","Sonstige dingliche Rechte an unbeweglichen Gütern - Gebuchte Mehrwerte"
"a223009","Other Rights to Immovable Property - Depreciations and Amounts Written Down - Recorded","223009","asset_fixed","","False","Autres droits réels sur des immeubles - Amortissements ou réductions de valeur actés","Overige zakelijke rechten op onroerende goederen - Geboekte afschrijvingen of waardeverminderingen","Sonstige dingliche Rechte an unbeweglichen Gütern - Gebuchte Abschreibungen oder Wertminderungen"
"a230000","Plant, Machinery and Equipment - Acquisition Value","230000","asset_fixed","","False","Installations, machines et outillage - Valeur d'acquisition","Installaties, machines en uitrusting - Aanschaffingswaarde","Anlagen, Maschinen und Betriebsausstattung - Anschaffungswert"
"a230008","Plant, Machinery and Equipment - Capital Profits - Recorded","230008","asset_fixed","","False","Installations, machines et outillage - Plus-values actées","Installaties, machines en uitrusting - Geboekte meerwaarden","Anlagen, Maschinen und Betriebsausstattung - Gebuchte Mehrwerte"
"a230009","Plant, Machinery and Equipment - Depreciations and Amounts Written Down - Recorded","230009","asset_fixed","","False","Installations, machines et outillage - Amortissements ou réductions de valeur actés","Installaties, machines en uitrusting - Geboekte afschrijvingen of waardeverminderingen","Anlagen, Maschinen und Betriebsausstattung - Gebuchte Abschreibungen oder Wertminderungen"
"a240000","Furniture and Vehicles - Acquisition Value","240000","asset_fixed","","False","Mobilier et matériel roulant - Valeur d'acquisition","Meubilair en rollend materieel - Aanschaffingsprijs","Geschäftsausstattung und Fuhrpark - Anschaffungswert"
"a240008","Furniture and Vehicles - Capital Profits - Recorded","240008","asset_fixed","","False","Mobilier et matériel roulant - Plus-values actées","Meubilair en rollend materieel - Geboekte meerwaarden","Geschäftsausstattung und Fuhrpark - Gebuchte Mehrwerte"
"a240009","Furniture and Vehicles - Depreciations and Amounts Written Down - Recorded","240009","asset_fixed","","False","Mobilier et matériel roulant - Amortissements ou moins-values actés","Meubilair en rollend materieel - Geboekte afschrijvingen of waardeverminderingen","Geschäftsausstattung und Fuhrpark - Gebuchte Abschreibungen oder Wertminderungen"
"a250","Land and Buildings in Leasing or Other Similar Rights","250","asset_fixed","","False","Terrains et constructions détenues en location-financement et droits similaires","Terreinen en gebouwen in leasing of op grond van een soortgelijk recht","Aufgrund von Leasing und ähnliche Rechten gehaltene Grundstücke und Bauten"
"a251","Plant, Machinery and Equipment in Leasing or Other Similar Rights","251","asset_fixed","","False","Installations, machines et outillage détenues en location-financement et droits similaires","Installaties, machines en uitrusting in leasing of op grond van een soortgelijk recht","Aufgrund von Leasing und ähnliche Rechten gehaltene Anlagen, Maschinen und Betriebsausstattung"
"a252","Furniture and Vehicles in Leasing or Other Similar Rights","252","asset_fixed","","False","Mobilier et matériel roulant détenues en location-financement et droits similaires","Meubilair en rollend materieel in leasing of op grond van een soortgelijk recht","Aufgrund von Leasing und ähnliche Rechten gehaltene Geschäftsausstattung und Fuhrpark"
"a26","Other Tangible Fixed Assets","26","asset_fixed","","False","Autres immobilisations corporelles","Overige materiële vaste activa","Sonstige Sachanlagen"
"a27","Tangible Fixed Assets Under Construction and Advance Payments","27","asset_fixed","","False","Immobilisations corporelles en cours et acomptes versés","Vaste activa in aanbouw en vooruitbetalingen","Anlagen im Bau und geleistete Anzahlungen"
"a2800","Participating Interests in Affiliated Enterprises - Acquisition Value","2800","asset_non_current","","False","Participations dans des entreprises liées - Valeur d'acquisition","Deelnemingen in verbonden ondernemingen - Aanschaffingswaarde","Beteiligungen an verbundenen Unternehmen - Anschaffungswert"
"a2801","Participating Interests in Affiliated Enterprises - Uncalled Amounts (-)","2801","asset_non_current","","False","Participations dans des entreprises liées - Montants non appelés (-)","Deelnemingen in verbonden ondernemingen - Nog te storten bedragen (-)","Beteiligungen an verbundenen Unternehmen - Nicht eingeforderte Beträge (-)"
"a2808","Participating Interests in Affiliated Enterprises - Capital Profits - Recorded","2808","asset_non_current","","False","Participations dans des entreprises liées - Plus-values actées","Deelnemingen in verbonden ondernemingen - Geboekte meerwaarden","Beteiligungen an verbundenen Unternehmen - Gebuchte Mehrwerte"
"a2809","Participating Interests in Affiliated Enterprises - Amounts Written Down - Recorded (-)","2809","asset_non_current","","False","Participations dans des entreprises liées - Réductions de valeur actées (-)","Deelnemingen in verbonden vennootschappen - Geboekte waardeverminderingen (-)","Beteiligungen an verbundenen Unternehmen - Gebuchte Wertminderungen (-)"
"a2810","Amounts Receivable From Affiliated Enterprises - Current Account","2810","asset_non_current","","False","Créances sur des entreprises liées - Créances en compte","Vorderingen op verbonden ondernemingen - Vorderingen op rekening","Forderungen gegen verbundene Unternehmen - Kontoforderungen"
"a2811","Amounts Receivable From Affiliated Enterprises - Bills Receivable","2811","asset_non_current","","False","Créances sur des entreprises liées - Effets à recevoir","Vorderingen op verbonden ondernemingen - Te innen wissels","Forderungen gegen verbundene Unternehmen - Besitzwechsel"
"a2812","Amounts Receivable From Affiliated Enterprises - Fixed-Income Securities","2812","asset_non_current","","False","Créances sur des entreprises liées - Titres à revenu fixe","Vorderingen op verbonden ondernemingen - Vastrentende effecten","Forderungen gegen verbundene Unternehmen - Festverzinsliche Wertpapiere"
"a2817","Amounts Receivable From Affiliated Enterprises - Doubtful Amounts","2817","asset_non_current","","False","Créances sur des entreprises liées - Créances douteuses","Vorderingen op verbonden ondernemingen - Dubieuze debiteuren","Forderungen gegen verbundene Unternehmen - Zweifelhafte Forderungen"
"a2819","Amounts Receivable From Affiliated Enterprises - Amounts Written Down - Recorded (-)","2819","asset_non_current","","False","Créances sur des entreprises liées - Réductions de valeur actées (-)","Vorderingen op verbonden ondernemingen - Geboekte waardeverminderingen (-)","Forderungen gegen verbundene Unternehmen - Gebuchte Wertminderungen (-)"
"a2820","Participating Interests in Other Enterprises Linked by Participating Interests - Acquisition Value","2820","asset_non_current","","False","Participations dans des entreprises avec lesquelles il existe un lien de participation - Valeur d'acquisition","Deelnemingen in ondernemingen waarmee een deelnemingsverhouding bestaat - Aanschaffingswaarde","Beteiligungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Anschaffungswert"
"a2821","Participating Interests in Other Enterprises Linked by Participating Interests - Uncalled Amounts (-)","2821","asset_non_current","","False","Participations dans des entreprises avec lesquelles il existe un lien de participation - Montants non appelés (-)","Deelnemingen in ondernemingen waarmee een deelnemingsverhouding bestaat - Nog te storten bedragen (-)","Beteiligungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Nicht eingeforderte Beträge (-)"
"a2828","Participating Interests in Other Enterprises Linked by Participating Interests - Capital Profits - Recorded","2828","asset_non_current","","False","Participations dans des entreprises avec lesquelles il existe un lien de participation - Plus-values actées","Deelnemingen in ondernemingen waarmee een deelnemingsverhouding bestaat - Geboekte meerwaarden","Beteiligungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Gebuchte Mehrwerte"
"a2829","Participating Interests in Other Enterprises Linked by Participating Interests - Amounts Written Down - Recorded (-)","2829","asset_non_current","","False","Participations dans des entreprises avec lesquelles il existe un lien de participation - Réductions de valeur actées (-)","Deelnemingen in ondernemingen waarmee een deelnemingsverhouding bestaat - Geboekte waardeverminderingen (-)","Beteiligungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Gebuchte Wertminderungen (-)"
"a2830","Amounts Receivable From Other Enterprises Linked by Participating Interests - Current Account","2830","asset_non_current","","False","Créances sur des entreprises avec lesquelles il existe un lien de participation - Créances en compte","Vorderingen op ondernemingen waarmee een deelnemingsverhouding bestaat - Vorderingen op rekening","Forderungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Kontoforderungen"
"a2831","Amounts Receivable From Other Enterprises Linked by Participating Interests - Bills Receivable","2831","asset_non_current","","False","Créances sur des entreprises avec lesquelles il existe un lien de participation - Effets à recevoir","Vorderingen op ondernemingen waarmee een deelnemingsverhouding bestaat - Te innen wissels","Forderungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Besitzwechsel"
"a2832","Amounts Receivable From Other Enterprises Linked by Participating Interests - Fixed-Income Securities","2832","asset_non_current","","False","Créances sur des entreprises avec lesquelles il existe un lien de participation - Titres à revenu fixe","Vorderingen op ondernemingen waarmee een deelnemingsverhouding bestaat - Vastrentende effecten","Forderungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Festverzinsliche Wertpapiere"
"a2837","Amounts Receivable From Other Enterprises Linked by Participating Interests - Doubtful Amounts","2837","asset_non_current","","False","Créances sur des entreprises avec lesquelles il existe un lien de participation - Créances douteuses","Vorderingen op ondernemingen waarmee een deelnemingsverhouding bestaat - Dubieuze debiteuren","Forderungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Zweifelhafte Forderungen"
"a2839","Amounts Receivable From Other Enterprises Linked by Participating Interests - Amounts Written Down - Recorded (-)","2839","asset_non_current","","False","Créances sur les entreprises avec lesquelles il existe un lien de participation - Réductions de valeur actées (-)","Vorderingen op ondernemingen waarmee een deelnemingsverhouding bestaat - Geboekte waardeverminderingen (-)","Forderungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht - Gebuchte Wertminderungen (-)"
"a2840","Other Shares - Acquisition Value","2840","asset_non_current","","False","Autres actions et parts - Valeur d'acquisition","Andere aandelen - Aanschaffingswaarde","Sonstige Aktien oder Anteile - Anschaffungswert"
"a2841","Other Shares - Uncalled Amounts (-)","2841","asset_non_current","","False","Autres actions et parts - Montants non appelés (-)","Andere aandelen - Nog te storten bedragen (-)","Sonstige Aktien oder Anteile - Nicht eingeforderte Beträge (-)"
"a2848","Other Shares - Capital Profits - Recorded","2848","asset_non_current","","False","Autres actions et parts - Plus-values actées","Andere aandelen - Geboekte meerwaarden","Sonstige Aktien oder Anteile - Gebuchte Mehrwerte"
"a2849","Other Shares - Amounts Written Down - Recorded (-)","2849","asset_non_current","","False","Autres actions et parts - Réductions de valeur actées (-)","Andere aandelen - Geboekte waardeverminderingen (-)","Sonstige Aktien oder Anteile - Gebuchte Wertminderungen (-)"
"a2850","Other Amounts Receivable - Current Account","2850","asset_non_current","","False","Autres créances - Créances en compte","Overige vorderingen - Vorderingen op rekening","Sonstige Forderungen - Kontoforderungen"
"a2851","Other Amounts Receivable - Bills Receivable","2851","asset_non_current","","False","Autres créances - Effets à recevoir","Overige vorderingen - Te innen wissels","Sonstige Forderungen - Besitzwechsel"
"a2852","Other Amounts Receivable - Fixed-Income Securities","2852","asset_non_current","","False","Autres créances - Titres à revenu fixe","Overige vorderingen - Vastrentende effecten","Sonstige Forderungen - Festverzinsliche Wertpapiere"
"a2857","Other Amounts Receivable - Doubtful Amounts","2857","asset_non_current","","False","Autres créances - Créances douteuses","Overige vorderingen - Dubieuze debiteuren","Sonstige Forderungen - Zweifelhafte Forderungen"
"a2859","Other Amounts Receivable - Amounts Written Down - Recorded (-)","2859","asset_non_current","","False","Autres créances - Réductions de valeur actées (-)","Overige vorderingen - Geboekte waardeverminderingen (-)","Sonstige Forderungen - Gebuchte Wertminderungen (-)"
"a288","Cash Guarantees Paid","288","asset_non_current","","False","Cautionnements versés en numéraire","Borgtochten betaald in contanten","Gezahlte Kautionen"
"a2900","Customers (> 1 Year)","2900","asset_non_current","","False","Clients (> 1 an)","Handelsdebiteuren (> 1 jaar)","Kunden (> 1 Jahr)"
"a2901","Trade Debtors - Bills Receivable (> 1 Year)","2901","asset_non_current","","False","Créances commerciales - Effets à recevoir (> 1 an)","Handelsvorderingen - Te innen wissels (> 1 jaar)","Forderungen aus Lieferungen und Leistungen - Besitzwechsel (> 1 Jahr)"
"a2906","Trade Debtors - Advance Payments (> 1 Year)","2906","asset_non_current","","False","Créances commerciales - Acomptes versés (> 1 an)","Handelsvorderingen - Vooruitbetalingen (> 1 jaar)","Forderungen aus Lieferungen und Leistungen - Geleistete Anzahlungen (> 1 Jahr)"
"a2907","Trade Debtors - Doubtful Amounts (> 1 Year)","2907","asset_non_current","","False","Créances commerciales - Créances douteuses (> 1 an)","Handelsvorderingen - Dubieuze debiteuren (> 1 jaar)","Forderungen aus Lieferungen und Leistungen - Zweifelhafte Forderungen (> 1 Jahr)"
"a2909","Trade Debtors - Amounts Written Down - Recorded (> 1 Year) (-)","2909","asset_non_current","","False","Créances commerciales - Réductions de valeur actées (> 1 an) (-)","Handelsvorderingen - Geboekte waardeverminderingen (> 1 jaar) (-)","Forderungen aus Lieferungen und Leistungen - Gebuchte Wertminderungen (> 1 Jahr) (-)"
"a2910","Current Account Receivable (> 1 Year)","2910","asset_non_current","","False","Créances en compte (> 1 an)","Vorderingen op rekening (> 1 jaar)","Kontoforderungen (> 1 Jahr)"
"a2911","Other Amounts Receivable - Bills Receivable (> 1 Year)","2911","asset_non_current","","False","Autres créances - Effets à recevoir (> 1 an)","Overige vorderingen - Te innen wissels (> 1 jaar)","Sonstige Forderungen - Besitzwechsel (> 1 Jahr)"
"a2919","Other Amounts Receivable - Amounts Written Down Recorded (> 1 Year) (-)","2919","asset_non_current","","False","Autres créances - Réductions de valeur actées (> 1 an) (-)","Overige vorderingen - Geboekte waardeverminderingen (> 1 jaar) (-)","Sonstige Forderungen - Gebuchte Wertminderungen (> 1 Jahr) (-)"
"a300","Raw Materials - Acquisition Value","300","asset_current","","False","Matières premières - Valeur d'acquisition","Grondstoffen - Aanschaffingswaarde","Rohstoffe - Anschaffungswert"
"a309","Raw Materials - Amounts Written Down - Recorded (-)","309","asset_current","","False","Matières premières - Réductions de valeur actées (-)","Grondstoffen - Geboekte waardeverminderingen (-)","Rohstoffe - Gebuchte Wertminderungen (-)"
"a310","Consumables - Acquisition Value","310","asset_current","","False","Approvisionnements et fournitures - Valeur d'acquisition","Hulpstoffen - Aanschaffingswaarde","Hilfs- und Betriebsstoffe - Anschaffungswert"
"a319","Consumables - Amounts Written Down - Recorded (-)","319","asset_current","","False","Approvisionnements et fournitures - Réductions de valeur actées (-)","Hulpstoffen - Geboekte waardeverminderingen (-)","Hilfs- und Betriebsstoffe - Gebuchte Wertminderungen (-)"
"a320","Work in Progress - Acquisition Value","320","asset_current","","False","En-cours de fabrication - Valeur d'acquisition","Goederen in bewerking - Aanschaffingswaarde","Unfertige Erzeugnisse - Anschaffungswert"
"a329","Work in Progress - Amounts Written Down - Recorded (-)","329","asset_current","","False","En-cours de fabrication - Réductions de valeur actées (-)","Goederen in bewerking - Geboekte waardeverminderingen (-)","Unfertige Erzeugnisse - Gebuchte Wertminderungen (-)"
"a330","Finished Goods - Acquisition Value","330","asset_current","","False","Produits finis - Valeur d'acquisition","Gereed product - Aanschaffingswaarde","Fertige Erzeugnisse - Anschaffungswert"
"a339","Finished Goods - Amounts Written Down - Recorded (-)","339","asset_current","","False","Produits finis - Réductions de valeur actées (-)","Gereed product - Geboekte waardeverminderingen (-)","Fertige Erzeugnisse - Gebuchte Wertminderungen (-)"
"a340","Goods Purchased for Resale - Acquisition Value","340","asset_current","","False","Marchandises - Valeur d'acquisition","Handelsgoederen - Aanschaffingswaarde","Waren - Anschaffungswert"
"a349","Goods Purchased for Resale - Amounts Written Down - Recorded (-)","349","asset_current","","False","Marchandises - Réductions de valeur actées (-)","Handelsgoederen - Geboekte waardeverminderingen (-)","Waren - Gebuchte Wertminderungen (-)"
"a350","Immovable Property Intended for Sale - Acquisition Value","350","asset_current","","False","Immeubles destinés à la vente - Valeur d'acquisition","Onroerende goederen bestemd voor verkoop - Aanschaffingswaarde","Zum Verkauf bestimmte unbewegliche Gegenstände - Anschaffungswert"
"a359","Immovable Property Intended for Sale - Amounts Written Down - Recorded (-)","359","asset_current","","False","Immeubles destinés à la vente - Réductions de valeur actées (-)","Onroerende goederen bestemd voor verkoop - Geboekte waardeverminderingen (-)","Zum Verkauf bestimmte unbewegliche Gegenstände - Gebuchte Wertminderungen (-)"
"a360","Advance Payments on Purchases for Stocks - Advance Payments","360","asset_current","","False","Acomptes versés sur achats pour stocks - Acomptes versés","Vooruitbetalingen op voorraadinkopen - Vooruitbetalingen","Geleistete Anzahlungen auf Vorräte - Geleistete Anzahlungen"
"a369","Advance Payments on Purchases for Stocks - Amounts Written Down - Recorded (-)","369","asset_current","","False","Acomptes versés sur achats pour stocks - Réductions de valeur actées (-)","Vooruitbetalingen op voorraadinkopen - Geboekte waardeverminderingen (-)","Geleistete Anzahlungen auf Vorräte - Gebuchte Wertminderungen (-)"
"a370","Contracts in Progress - Acquisition Value","370","asset_current","","False","Commandes en cours d'exécution - Valeur d'acquisition","Bestellingen in uitvoering - Aanschaffingswaarde","In Ausführung befindliche Bestellungen - Anschaffungswert"
"a371","Contracts in Progress - Profit Recognized","371","asset_current","","False","Commandes en cours d'exécution - Bénéfice pris en compte","Bestellingen in uitvoering - Toegerekende winst","In Ausführung befindliche Bestellungen - Zugewiesener Gewinn"
"a379","Contracts in Progress - Amounts Written Down - Recorded (-)","379","asset_current","","False","Commandes en cours d'exécution - Réductions de valeur actées (-)","Bestellingen in uitvoering - Geboekte waardeverminderingen (-)","In Ausführung befindliche Bestellungen - Gebuchte Wertminderungen (-)"
"a400","Customers","400","asset_receivable","","True","Clients","Handelsdebiteuren","Kunden"
"a4001","Customers (POS)","4001","asset_receivable","","True","Clients (POS)","Handelsdebiteuren (POS)","Kunden (POS)"
"a401","Trade Debtors - Bills Receivable","401","asset_current","","False","Créances commerciales - Effets à recevoir","Handelsvorderingen - Te innen wissels","Forderungen aus Lieferungen und Leistungen - Besitzwechsel"
"a404","Trade Debtors - Income Receivable","404","asset_receivable","","True","Créances commerciales - Produits à recevoir","Handelsvorderingen - Te innen opbrengsten","Forderungen aus Lieferungen und Leistungen - Zu erhaltene Erträge"
"a406","Trade Debtors - Advance Payments","406","asset_receivable","","True","Créances commerciales - Acomptes versés","Handelsvorderingen - Vooruitbetalingen","Forderungen aus Lieferungen und Leistungen - Geleistete Anzahlungen"
"a407","Trade Debtors - Doubtful Amounts","407","asset_receivable","","True","Créances commerciales - Créances douteuses","Handelsvorderingen - Dubieuze debiteuren","Forderungen aus Lieferungen und Leistungen - Zweifelhalte Forderungen"
"a409","Trade Debtors - Amounts Written Down - Recorded (-)","409","asset_current","","False","Créances commerciales - Réductions de valeur actées (-)","Handelsvorderingen - Geboekte waardeverminderingen (-)","Forderungen aus Lieferungen und Leistungen - Gebuchte Wertminderungen (-)"
"a411","VAT Recoverable","411","asset_current","","False","TVA à récupérer","Terug te vorderen btw","Zurück zu erhaltende Mehrwertsteuer"
"a4112","VAT Recoverable: VAT Current Account (C/A)","4112","asset_current","","False","TVA à récupérer : Compte courant TVA (C/C)","Terug te vorderen btw: Rekening-courant btw (R/C)","Zurück zu erhaltende Mehrwertsteuer: Girokonto-MwSt"
"a4125","Other Belgian Taxes to Be Recovered","4125","asset_current","","False","Autres impôts et taxes belges à récupérer","Terug te vorderen andere Belgische belastingen","Zurück zu erhaltende sonstige belgische Steuern"
"a4128","Foreign Taxes to Be Recovered","4128","asset_current","","False","Impôts et taxes étrangers à récupérer","Terug te vorderen buitenlandse belastingen","Zurück zu erhaltende ausländische Steuern"
"a414","Income Receivable","414","asset_current","","False","Produits à recevoir","Te innen opbrengsten","Zu erhaltende Erträge"
"a416","Sundry Amounts Receivable","416","asset_current","","False","Créances diverses","Diverse vorderingen","Sonstige Forderungen"
"a417","Sundry Amounts Receivable - Doubtful Amounts","417","asset_current","","False","Créances diverses - Créances douteuses","Overige vorderingen - Dubieuze debiteuren","Sonstige Forderungen - Zweifelhafte Forderungen"
"a418","Sundry Amounts Receivable - Cash Guarantees Paid","418","asset_current","","False","Créances diverses - Cautionnements versés en numéraire","Overige vorderingen - Borgtochten betaald in contanten","Sonstige Forderungen - Gezahlte Kautionen"
"a419","Sundry Amounts Receivable - Amounts Written Down - Recorded (-)","419","asset_current","","False","Créances diverses - Réductions de valeur actées (-)","Overige vorderingen - Geboekte waardeverminderingen (-)","Sonstige Forderungen - Gebuchte Wertminderungen (-)"
"a422","Leasing and Other Similar Obligations (> 1 Year Falling Due < 1 Year)","422","liability_current","","False","Dettes de location-financement et dettes assimilées (> 1 an écheant < 1 an)","Leasingschulden en soortgelijke schulden (> 1 jaar vervallend < 1 jaar)","Verbindlichkeiten aufgrund von Leasing und ähnlichen Verträgen (> 1 Jahr fällig < 1 Jahr)"
"a4230","Credit Institutions - Current Account Payable (> 1 Year Falling Due < 1 Year)","4230","liability_current","","False","Etablissements de crédit - Dettes en compte (> 1 an écheant < 1 an)","Kredietinstellingen - Schulden op rekening (> 1 jaar vervallend < 1 jaar)","Kreditinstitute - Kontoverbindlichkeiten (> 1 Jahr fällig < 1 Jahr)"
"a4231","Credit Institutions - Promissory Notes (> 1 Year Falling Due < 1 Year)","4231","liability_current","","False","Etablissements de crédit - Promesses (> 1 an écheant < 1 an)","Kredietinstellingen - Promessen (> 1 jaar vervallend < 1 jaar)","Kreditinstitute - Solawechsel (> 1 Jahr fällig < 1 Jahr)"
"a4232","Credit Institutions - Bank Acceptances (> 1 Year Falling Due < 1 Year)","4232","liability_current","","False","Etablissements de crédit - Crédits d'acceptation (> 1 an écheant < 1 an)","Kredietinstellingen - Acceptkredieten (> 1 jaar vervallend < 1 jaar)","Kreditinstitute - Akzeptkredite (> 1 Jahr fällig < 1 Jahr)"
"a424","Other Loans (> 1 Year Falling Due < 1 Year)","424","liability_current","","False","Autres emprunts (> 1 an écheant < 1 an)","Overige leningen (> 1 jaar vervallend < 1 jaar)","Sonstige Anleihen (> 1 Jahr fällig < 1 Jahr)"
"a4250","Suppliers (> 1 Year Falling Due < 1 Year)","4250","liability_current","","False","Fournisseurs (> 1 an écheant < 1 an)","Leveranciers (> 1 jaar vervallend < 1 jaar)","Lieferanten (> 1 Jahr fällig < 1 Jahr)"
"a4251","Bills of Exchange Payable (> 1 Year Falling Due < 1 Year)","4251","liability_current","","False","Effets à payer (> 1 an écheant < 1 an)","Te betalen wissels (> 1 jaar vervallend < 1 jaar)","Verbindlichkeiten aus Wechseln (> 1 Jahr fällig < 1 Jahr)"
"a426","Advance Payments on Contracts in Progress (> 1 Year Falling Due < 1 Year)","426","liability_current","","False","Acomptes sur commandes (> 1 an écheant < 1 an)","Vooruitbetalingen op bestellingen (> 1 jaar vervallend < 1 jaar)","Anzahlungen auf Bestellungen (> 1 Jahr fällig < 1 Jahr)"
"a428","Guarantees Received in Cash (> 1 Year Falling Due < 1 Year)","428","liability_current","","False","Cautionnements reçus en numéraire (> 1 an écheant < 1 an)","Borgtochten ontvangen in contanten (> 1 jaar vervallend < 1 jaar)","In Geldmittel erhaltene Kautionen (> 1 Jahr fällig < 1 Jahr)"
"a430","Credit Institutions - Fixed-Term Loans","430","liability_current","","False","Etablissements de crédit - Emprunts en compte à terme fixe","Kredietinstellingen - Leningen op rekening met vaste termijn","Kreditinstitute - Kontokredite mit fester Laufzeit"
"a431","Credit Institutions - Promissory Notes","431","liability_current","","False","Etablissements de crédit - Promesses","Kredietinstellingen - Promessen","Kreditinstitute - Solawechsel"
"a432","Credit Institutions - Bank Acceptances","432","liability_current","","False","Etablissements de crédit - Crédits d'acceptation","Kredietinstellingen - Acceptkredieten","Kreditinstitute - Akzeptkredite"
"a433","Credit Institutions - Current Account Payable","433","liability_current","","False","Etablissements de crédit - Dettes en compte courant","Kredietinstellingen - Schulden in rekening-courant","Kreditinstitute - Kontokorrentverbindlichkeiten"
"a439","Other Loans","439","liability_current","","False","Autres emprunts","Overige leningen","Sonstige Anleihen"
"a440","Suppliers","440","liability_payable","","True","Fournisseurs","Leveranciers","Lieferanten"
"a441","Bills of Exchange Payable","441","liability_current","","False","Effets à payer","Te betalen wissels","Verbindlichkeiten aus Wechseln"
"a444","Invoices to Be Received","444","liability_payable","","True","Factures à recevoir","Te ontvangen facturen","Zu erhaltende Rechnungen"
"a4505","Estimated Other Belgian Taxes Payable","4505","liability_current","","False","Dettes fiscales estimées - Autres impôts et taxes belges","Geraamd bedrag der belastingschulden - Andere Belgische belastingen en taksen","Geschätzte Steuerschulden - Sonstige belgische Steuern und Abgabe"
"a4508","Estimated Foreign Taxes Payable","4508","liability_current","","False","Dettes fiscales estimées - Impôts et taxes étrangers","Geraamd bedrag der belastingschulden - Buitenlandse belastingen en taksen","Geschätzte Steuerschulden - Ausländische Steuern und Abgaben"
"a451","VAT Payable","451","liability_current","","False","TVA à payer","Te betalen btw","Zu zahlende Mehrwertsteuer"
"a451054","VAT Payable (Grid 54)","451054","liability_current","","False","TVA à payer (Grille 54)","Te betalen btw (Rooster 54)","Zu zahlende Mehrwertsteuer (Raster 54)"
"a451055","VAT Payable on Intra-Community Operations (Grid 55)","451055","liability_current","","False","TVA à payer sur opérations intracommunautaires (Grille 55)","Te betalen btw op intracommunautaire verrichtingen (Rooster 55)","Zu zahlende Mehrwertsteuer auf innergemeinschaftliche Umsätze (Raster 55)"
"a451056","VAT Payable on Transactions by Co-Contractor (Grid 56)","451056","liability_current","","False","TVA à payer sur opérations co-contractant (Grille 56)","Te betalen btw op verrichtingen medecontractant (Rooster 56)","Zu zahlende Mehrwertsteuer auf Umsätze des Vertragspartners (Raster 56)"
"a451057","VAT Payable: Deferred Payment on Import (Grid 57)","451057","liability_current","","False","TVA à payer : Report de paiement sur importations (Grille 57)","Te betalen btw: Uitgestelde betaling op invoer (Rooster 57)","Zu zahlende Mehrwertsteuer: Zahlungsaufschub bei der Einfuhr (Raster 57)"
"a451063","VAT Payable on Credit Notes (Grid 63)","451063","liability_current","","False","TVA à payer sur notes de crédit (Grille 63)","Te betalen btw op creditnota's (Rooster 63)","Zu zahlende Mehrwertsteuer auf Gutschriften (Raster 63)"
"a4512","VAT Payable: VAT Current Account (C/A)","4512","liability_current","","False","TVA à payer : Compte courant TVA (C/C)","Te betalen btw: Rekening-courant btw (R/C)","Zu zahlende Mehrwertsteuer: Girokonto-MwSt"
"a451800","VAT Payable - Insufficient Taxation","451800","liability_current","","False","TVA à payer - Taxation insuffisante","Te betalen btw - Ontoereikende heffing","Zu zahlende Mehrwertsteuer - Unzureichende Besteuerung"
"a451820","VAT Payable - Deduction Revisions","451820","liability_current","","False","TVA à payer - Révisions des déductions","Te betalen btw - Herzieningen aftrek","Zu zahlende Mehrwertsteuer - Abzugsrevisionen"
"a451830","VAT Payable - Regularizations","451830","liability_current","","False","TVA à payer - Régularisations","Te betalen btw - Regularisaties","Zu zahlende Mehrwertsteuer - Regularisierungen"
"a4525","Other Belgian Taxes Payable","4525","liability_current","","False","Autres impôts et taxes belges à payer","Andere te betalen Belgische belastingen en taksen","Sonstige zu zahlende belgische Steuern und Abgaben"
"a4528","Foreign Taxes Payable","4528","liability_current","","False","Impôts et taxes étrangers à payer","Te betalen buitenlandse belastingen en taksen","Zu zahlende ausländische Steuern und Abgaben"
"a453","Taxes Withheld","453","liability_current","","False","Précomptes retenus","Ingehouden voorheffingen","Einbehaltene Steuervorhabzüge"
"a454","National Social Security Office (NSSO)","454","liability_current","","False","Office National de Sécurité Sociale (ONSS)","Rijksdienst voor Sociale Zekerheid (RSZ)","Landesamt für Soziale Sicherheit (LSS)"
"a455","Remuneration","455","liability_current","","False","Rémunérations","Bezoldigingen","Arbeitsentgelte"
"a456","Holiday Pay","456","liability_current","","False","Pécules de vacances","Vakantiegeld","Urlaubsgeld"
"a459","Other Social Obligations","459","liability_current","","False","Autres dettes sociales","Andere sociale schulden","Sonstige Soziallasten"
"a46","Advance Payments on Contracts in Progress","460","liability_current","","False","Acomptes sur commandes","Vooruitbetalingen op bestellingen","Erhaltene Anzahlungen auf Bestellungen"
"a480","Debentures and Matured Coupons","480","liability_current","","False","Obligations et coupons échus","Vervallen obligaties en coupons","Fällige Schuldverschreibungen und Kupons"
"a488","Guarantees Received in Cash","488","liability_current","","False","Cautionnements reçus en numéraire","Borgtochten ontvangen in contanten","In Geldmitteln erhaltene Kautionen"
"a490","Deferred Charges","490","asset_current","","False","Charges à reporter","Over te dragen kosten","Vorzutragende Aufwendungen"
"a491","Accrued Income","491","asset_current","","False","Produits acquis","Verkregen opbrengsten","Erworbene Erträge"
"a492","Accrued Charges","492","liability_current","","False","Charges à imputer","Toe te rekenen kosten","Anzurechnende Aufwendungen"
"a493","Deferred Income","493","liability_current","","False","Produits à reporter","Over te dragen opbrengsten","Vorzutragende Erträge"
"a499","Suspense Accounts","499","liability_current","","False","Comptes d'attente","Wachtrekeningen","Wartekonten"
"a511","Shares - Uncalled Amounts (-)","511","asset_current","","False","Actions et parts - Montants non appelés (-)","Aandelen - Niet-opgevraagde bedragen (-)","Aktien und Anteile - Nicht eingeforderte Beträge (-)"
"a520","Fixed-Income Securities - Acquisition Value","520","asset_current","","False","Titres à revenu fixe - Valeur d'acquisition","Vastrentende effecten - Aanschaffingswaarde","Festverzinsliche Wertpapiere - Anschaffungswert"
"a529","Fixed-Income Securities - Amounts Written Down - Recorded (-)","529","asset_current","","False","Titres à revenu fixe - Réductions de valeur actées (-)","Vastrentende effecten - Geboekte waardeverminderingen (-)","Festverzinsliche Wertpapiere - Gebuchte Wertminderungen (-)"
"a530","Term Deposits at More Than One Year","530","asset_current","","False","Dépôts à terme de plus d'un an","Termijndeposito's op meer dan één jaar","Terminkonten mit einer Laufzeit von mehr als einem Jahr"
"a531","Term Deposits at More Than One Month and With a Maximum Term of One Year","531","asset_current","","False","Dépôts à terme de plus d'un mois et à un an au plus","Termijndeposito's op meer dan één maand en hoogstens één jaar","Terminkonten mit einer Laufzeit von mehr als einem Monat und bis zu einem Jahr"
"a532","Term Deposits With a Maximum Term of One Month","532","asset_current","","False","Dépôts à terme d'un mois au plus","Termijndeposito's op hoogstens één maand","Terminkonten mit einer Laufzeit bis zu einem Monat"
"a539","Term Deposits - Amounts Written Down - Recorded (-)","539","asset_current","","False","Dépôts à terme - Réductions de valeur actées (-)","Termijndeposito's - Geboekte waardeverminderingen (-)","Terminkonten - Gebuchte Wertminderungen (-)"
"a54","Receivable Securities Fallen Due","54","asset_current","","False","Valeurs échues à l'encaissement","Te incasseren vervallen waarden","Inkassowerte"
"a550","Credit Institutions","550","asset_cash","","False","Etablissements de crédit","Kredietinstellingen","Kreditinstitute"
"a570","Cash in Hand - Cash","570","asset_cash","","False","Caisses - Espèces","Kassen - Contanten","Kassenbestand - Bargeld"
"a578","Cash in Hand - Stamps","578","asset_cash","","False","Caisses - Timbres","Kassen - Zegels","Kassenbestand - Wertmarken"
"a58","Internal Transfers of Funds","58","asset_current","","True","Virements internes","Interne overboekingen","Interne Geldtransferkonten"
"a600","Purchases of Raw Materials","600","expense","account.account_tag_operating","False","Achats de matières premières","Aankopen van grondstoffen","Käufe von Rohstoffen"
"a601","Purchases of Consumables","601","expense","account.account_tag_operating","False","Achats de fournitures","Aankopen van hulpstoffen","Käufe von Hilfs- und Betriebsstoffen"
"a602","Purchases of Services, Work and Studies","602","expense","account.account_tag_operating","False","Achats de services, travaux et études","Aankopen van diensten, werk en studies","Käufe von Dienstleistungen, Arbeiten und Studien"
"a603","Subcontracting","603","expense","account.account_tag_operating","False","Sous-traitances générales","Algemene onderaannemingen","Einsätze von Unterlieferanten"
"a604","Purchases of Goods for Resale","604","expense","account.account_tag_operating","False","Achats de marchandises","Aankopen van handelsgoederen","Kauf von Waren"
"a605","Purchases of Immovable Property for Resale","605","expense","","False","Achats d'immeubles destinés à la vente","Aankopen van onroerende goederen bestemd voor verkoop","Kauf von für den Verkauf bestimmten unbeweglichen Gütern"
"a608","Discounts, Allowance and Rebates Received (-)","608","expense","","False","Remises, ristournes et rabais (-)","Ontvangen kortingen, ristorno's en rabatten (-)","Erhaltene Preisnachlässe, Rückvergütungen und Rabatte (-)"
"a6090","Decrease (Increase) in Stocks of Raw Materials","6090","expense","","False","Variations des stocks de matières premières","Voorraadwijzigingen van grondstoffen","Bestandsveränderungen der Rohstoffe"
"a6091","Decrease (Increase) in Stocks of Consumables","6091","expense","","False","Variations des stocks de fournitures","Voorraadwijzigingen van hulpstoffen","Bestandsveränderungen der Hilfs- und Betriebsstoffe"
"a6094","Decrease (Increase) in Stocks of Goods Purchased for Resale","6094","expense","","False","Variations des stocks de marchandises","Voorraadwijzigingen van handelsgoederen","Bestandsveränderungen der Waren"
"a6095","Decrease (Increase) in Respect of Immovable Property for Resale","6095","expense","","False","Variations des stocks d'immeubles achetés destinés à la vente","Voorraadwijzigingen van gekochte onroerende goederen bestemd voor verkoop","Bestandsveränderungen der zum Verkauf bestimmten gekauften unbeweglichen Güter"
"a61","Services and Other Goods","61","expense","account.account_tag_operating","False","Services et biens divers","Diensten en diverse goederen","Übrige Lieferungen und Leistungen"
"a617","Hired Temporary Staff and Persons Placed at the Enterprise's Disposal","617","expense","","False","Personnel intérimaire et personnes mises à la disposition de l'entreprise","Uitzendkrachten en personen ter beschikking gesteld van de onderneming","Zeitarbeitpersonal und dem Unternehmen zur Verfügung gestellte Personen"
"a618","Remuneration, Premiums for Extra Statutory Insurance, Pensions of the Directors or the Management Staff Which Are Not Allowed Following the Contract","618","expense","","False","Rémunérations, primes pour assurances extralégales, pensions de retraite et de survie des administrateurs, gérants et associés actifs qui ne sont pas attribuées en vertu d'un contrat de travail","Bezoldigingen, premies voor buitenwettelijke verzekeringen, ouderdoms- en overlevingspensioenen van bestuurders, zaakvoerders en werkende vennoten, die niet worden toegekend uit hoofde van een arbeidsovereenkomst","Arbeitsentgelte, Prämien für außergesetzliche Versicherungen, Ruhestands- und Hinterbliebenenpensionen der Verwalter, Geschäftsführer und aktiven Gesellschafter, die nicht aufgrund eines Arbeitsvertrags zuerkannt werden"
"a6200","Remuneration and Direct Social Benefits - Directors or Managers","6200","expense","account.account_tag_operating","False","Rémunérations et avantages sociaux directs - Administrateurs ou gérants","Bezoldigingen en rechtstreekse sociale voordelen - Bestuurders of zaakvoerders","Arbeitsentgelte und direkte soziale Vorteile - Verwalter oder Geschäftsführer"
"a6201","Remuneration and Direct Social Benefits - Management Staff","6201","expense","account.account_tag_operating","False","Rémunérations et avantages sociaux directs - Personnel de direction","Bezoldigingen en rechtstreekse sociale voordelen - Directiepersoneel","Arbeitsentgelte und direkte soziale Vorteile - Führungskräfte"
"a6202","Remuneration and Direct Social Benefits - Salaried Employees","6202","expense","account.account_tag_operating","False","Rémunérations et avantages sociaux directs - Employés","Bezoldigingen en rechtstreekse sociale voordelen - Bedienden","Arbeitsentgelte und direkte soziale Vorteile - Angestellte"
"a6203","Remuneration and Direct Social Benefits - Hourly Employees","6203","expense","account.account_tag_operating","False","Rémunérations et avantages sociaux directs - Ouvriers","Bezoldigingen en rechtstreekse sociale voordelen - Arbeiders","Arbeitsentgelte und direkte soziale Vorteile - Arbeiter"
"a6204","Remuneration and Direct Social Benefits - Other Staff Members","6204","expense","account.account_tag_operating","False","Rémunérations et avantages sociaux directs - Autres membres du personnel","Bezoldigingen en rechtstreekse sociale voordelen - Andere personeelsleden","Arbeitsentgelte und direkte soziale Vorteile - Sonstige Personalmitglieder"
"a621","Employers' Contribution for Social Security","621","expense","account.account_tag_operating","False","Cotisations patronales pour assurances sociales","Werkgeversbijdragen voor sociale verzekeringen","Arbeitgeberbeiträge zur Sozialversicherung"
"a622","Employers' Premiums for Extra Statutory Insurance","622","expense","","False","Primes patronales pour assurances extra-légales","Werkgeverspremies voor bovenwettelijke verzekeringen","Arbeitgeberprämien für außergesetzliche Versicherungen"
"a623","Other Personnel Costs","623","expense","","False","Autres frais du personnel","Andere personeelskosten","Sonstige Personalaufwendungen"
"a6240","Retirement and Survivors' Pensions - Directors or Managers","6240","expense","","False","Pensions de retraite et de survie - Administrateurs ou gérants","Ouderdoms- en overlevingspensioenen - Bestuurders of zaakvoerders","Ruhestands- und Hinterbliebenenpensionen - Verwalter oder Geschäftsführer"
"a6241","Retirement and Survivors' Pensions - Personnel","6241","expense","","False","Pensions de retraite et de survie - Personnel","Ouderdoms- en overlevingspensioenen - Personeel","Ruhestands- und Hinterbliebenenpensionen - Personal"
"a6300","Depreciation of Formation Expenses","6300","expense","","False","Dotations aux amortissements sur frais d'établissement","Afschrijvingen op oprichtingskosten","Abschreibungen auf Errichtungsaufwendungen"
"a6301","Depreciation of Intangible Fixed Assets","6301","expense","account.account_tag_operating","False","Dotations aux amortissements sur immobilisations incorporelles","Afschrijvingen op immateriële vaste activa","Abschreibungen auf immaterielle Anlagewerte"
"a6302","Depreciation of Tangible Fixed Assets","6302","expense","account.account_tag_operating","False","Dotations aux amortissements sur immobilisations corporelles","Afschrijvingen op materiële vaste activa","Abschreibungen auf Sachanlagen"
"a6308","Amounts Written Down on Intangible Fixed Assets","6308","expense","","False","Dotations aux réductions de valeur sur immobilisations incorporelles","Waardeverminderingen op immateriële vaste activa","Wertminderungen auf immateriellen Anlagewerten"
"a6309","Amounts Written Down on Tangible Fixed Assets","6309","expense","","False","Dotations aux réductions de valeur sur immobilisations corporelles","Waardeverminderingen op materiële vaste activa","Wertminderungen auf Sachanlagen"
"a6310","Amounts Written Down on Stocks - Appropriations","6310","expense","account.account_tag_operating","False","Réductions de valeur sur stocks - Dotations","Waardeverminderingen op voorraden - Toevoegingen","Wertminderungen auf Vorräte - Zuführungen"
"a6311","Amounts Written Down on Stocks - Write-Backs (-)","6311","expense","","False","Réductions de valeur sur stocks - Reprises (-)","Waardeverminderingen op voorraden - Terugnemingen (-)","Wertminderungen auf Vorräte - Rücknahmen (-)"
"a6320","Amounts Written Down on Contracts in Progress - Appropriations","6320","expense","account.account_tag_operating","False","Réductions de valeur sur commandes en cours d'exécution - Dotations","Waardeverminderingen op bestellingen in uitvoering - Toevoegingen","Wertminderungen von in Ausführung befindlichen Bestellungen - Zuführungen"
"a6321","Amounts Written Down on Contracts in Progress - Write-Backs (-)","6321","expense","","False","Réductions de valeur sur commandes en cours d'exécution - Reprises (-)","Waardeverminderingen op bestellingen in uitvoering - Terugnemingen (-)","Wertminderungen von in Ausführung befindlichen Bestellungen - Rücknahmen (-)"
"a6330","Amounts Written Down on Trade Debtors (> 1 Year) - Appropriations","6330","expense","","False","Réductions de valeur sur créances commerciales (> 1 an) - Dotations","Waardeverminderingen op handelsvorderingen (> 1 jaar) - Toevoegingen","Wertminderungen von Forderungen aus Lieferungen und Leistungen (> 1 Jahr) - Zuführungen"
"a6331","Amounts Written Down on Trade Debtors (> 1 Year) - Write-Backs (-)","6331","expense","","False","Réductions de valeur sur créances commerciales (> 1 an) - Reprises (-)","Waardeverminderingen op handelsvorderingen (> 1 jaar) - Terugnemingen (-)","Wertminderungen von Forderungen aus Lieferungen und Leistungen (> 1 Jahr) - Rücknahmen (-)"
"a6340","Amounts Written Down on Trade Debtors - Appropriations","6340","expense","","False","Réductions de valeur sur créances commerciales - Dotations","Waardeverminderingen op handelsvorderingen - Toevoegingen","Wertminderungen von Forderungen aus Lieferungen und Leistungen - Zuführungen"
"a6341","Amounts Written Down on Trade Debtors - Write-Backs (-)","6341","expense","","False","Réductions de valeur sur créances commerciales - Reprises (-)","Waardeverminderingen op handelsvorderingen - Terugnemingen (-)","Wertminderungen von Forderungen aus Lieferungen und Leistungen - Rücknahmen (-)"
"a6350","Provisions for Pensions and Similar Obligations - Appropriations","6350","expense","","False","Provisions pour pensions et obligations similaires - Dotations","Voorzieningen voor pensioenen en soortgelijke verplichtingen - Toevoegingen","Rückstellungen für Pensionen und ähnliche Verpflichtungen - Zuführungen"
"a6351","Provisions for Pensions and Similar Obligations - Uses and Write-Backs (-)","6351","expense","","False","Provisions pour pensions et obligations similaires - Utilisations et reprises (-)","Voorzieningen voor pensioenen en soortgelijke verplichtingen - Bestedingen en terugnemingen (-)","Rückstellungen für Pensionen und ähnliche Verpflichtungen - Verbrauch und Auflösungen (-)"
"a6360","Provisions for Major Repairs and Maintenance - Appropriations","6360","expense","","False","Provisions pour grosses réparations et gros entretien - Dotations","Voorzieningen voor grote herstellings- en onderhoudswerken - Toevoegingen","Rückstellungen für große Reparaturen und große Instandhaltungsarbeiten - Zuführungen"
"a6361","Provisions for Major Repairs and Maintenance - Uses and Write-Backs (-)","6361","expense","","False","Provisions pour grosses réparations et gros entretien - Utilisations et reprises (-)","Voorzieningen voor grote herstellings- en onderhoudswerken - Bestedingen en terugnemingen (-)","Rückstellungen für große Reparaturen und große Instandhaltungsarbeiten - Verbrauch und Auflösungen (-)"
"a6370","Provisions for Environmental Obligations - Appropriations","6370","expense","","False","Provisions pour obligations environnementales - Dotations","Voorzieningen voor milieuverplichtingen - Toevoegingen","Rückstellungen für Umweltschutzverpflichtungen - Zuführungen"
"a6371","Provisions for Environmental Obligations - Uses and Write-Backs (-)","6371","expense","","False","Provisions pour obligations environnementales - Utilisations et reprises (-)","Voorzieningen voor milieuverplichtingen - Bestedingen en terugnemingen (-)","Rückstellungen für Umweltschutzverpflichtungen - Verbrauch und Auflösungen (-)"
"a640","Business Taxes","640","expense","account.account_tag_operating","False","Charges fiscales d'exploitation","Bedrijfsbelastingen","Betriebliche Steueraufwendungen"
"a641","Capital Losses on Ordinary Disposal of Fixed Assets","641","expense","","False","Moins-values sur réalisations courantes d'immobilisations corporelles","Minderwaarden bij de courante realisatie van materiële vaste activa","Verluste aus dem normalen Abgang von Sachanlagen"
"a642","Capital Losses on Disposal of Trade Debtors","642","expense","","False","Moins-values sur réalisations de créances commerciales","Minderwaarden op de realisatie van handelsvorderingen","Minderwerte bei Realisierung von Forderungen aus Lieferungen und Leistungen"
"a649","Operating Charges Carried to Assets as Restructuring Costs (-)","649","expense","","False","Charges d'exploitation portées à l'actif au titre de frais de restructuration (-)","Als herstructureringskosten geactiveerde bedrijfskosten (-)","Betriebliche Aufwendungen, die als Restrukturierungskosten aktiviert wurden (-)"
"a6500","Interest, Commission and Other Charges Relating to Debts","6500","expense","","False","Intérêts, commissions et frais afférents aux dettes","Rente, commissies en kosten verbonden aan schulden","Zinsen, Provisionen und Kosten der Verbindlichkeiten"
"a6501","Depreciation of Loan Issue Expenses","6501","expense","","False","Amortissement des frais d'émission d'emprunts","Afschrijving van kosten bij uitgifte van leningen","Abschreibungen auf Kosten der Emissionen von Anleihen"
"a6502","Capitalized Interests (-)","6502","expense","","False","Intérêts intercalaires portés à l'actif (-)","Geactiveerde intercalaire interesten (-)","Aktivierte Fremdkapitalzinsen (-)"
"a6510","Depreciations on Current Assets - Appropriations","6510","expense","","False","Réductions de valeur sur actifs circulants - Dotations","Waardeverminderingen op vlottende activa - Toevoegingen","Wertminderungen von Gegenständen des Umlaufvermögens - Zuführungen"
"a6511","Depreciations on Current Assets - Write-Backs (-)","6511","expense","","False","Réductions de valeur sur actifs circulants - Reprises (-)","Waardeverminderingen op vlottende activa - Terugnemingen (-)","Wertminderungen von Gegenständen des Umlaufvermögens - Rücknahmen (-)"
"a652","Capital Losses on Disposal of Current Assets","652","expense","","False","Moins-values sur réalisation d'actifs circulants ","Minderwaarden bij de realisatie van vlottende activa","Verluste aus dem Abgang von Gegenständen des Umlaufvermögens"
"a653","Discount Costs on Amounts Receivable","653","expense","account.account_tag_financing","False","Charges d'escompte de créances","Discontokosten op vorderingen","Skontoaufwands aus Forderungen"
"a654","Exchange Results","654","expense","account.account_tag_financing","False","Différences de change","Wisselresultaten","Wechselkursdifferenzen"
"a655","Results From the Conversion of Foreign Currencies","655","expense","","False","Écarts de conversion des devises","Resultaten uit de omrekening van vreemde valuta","Differenzen aus der Umrechnung von Fremdwährungen"
"a6560","Provisions of a Financial Nature - Appropriations","6560","expense","account.account_tag_financing","False","Provisions à caractère financier - Dotations","Voorzieningen met financieel karakter - Toevoegingen","Rückstellungen mit finanziellem Charakter - Zuführungen"
"a6561","Provisions of a Financial Nature - Uses and Write-Backs (-)","6561","expense","","False","Provisions à caractère financier - Utilisations et reprises (-)","Voorzieningen met financieel karakter - Bestedingen en terugnemingen (-)","Rückstellungen mit finanziellem Charakter - Verbrauch und Auflösungen (-)"
"a657000","Financial Discounts Allowed","657000","expense","account.account_tag_financing","False","Escomptes financiers accordés","Toegekende financiële kortingen","Gewährte finanzielle Rabatte"
"a657100","Negative Payment Differences","657100","expense","account.account_tag_financing","False","Différences de paiement négatives","Negatieve betalingsverschillen","Negative Zahlungsdifferenzen"
"a659","Financial Charges Carried to Assets as Restructuring Costs (-)","659","expense","","False","Charges financières portées à l'actif au titre de frais de restructuration (-)","Als herstructureringskosten geactiveerde financiële kosten (-)","Finanzaufwendungen, die als Restrukturierungskosten aktiviert wurden (-)"
"a6600","Non-Recurring Depreciation and Amounts Written Down on Formation Expenses","6600","expense","account.account_tag_investing","False","Amortissements et réductions de valeur non récurrents (dotations) sur frais d'établissement","Niet-recurrente afschrijvingen en waardeverminderingen (toevoeging) op oprichtingskosten","Nicht wiederkehrende Abschreibungen und Wertminderungen (Zuführungen) auf Errichtungsaufwendungen"
"a6601","Non-Recurring Depreciation and Amounts Written Down on Intangible Fixed Assets","6601","expense","","False","Amortissements et réductions de valeur non récurrents (dotations) sur immobilisations incorporelles","Niet-recurrente afschrijvingen en waardeverminderingen (toevoeging) op immateriële vaste activa","Nicht wiederkehrende Abschreibungen und Wertminderungen (Zuführungen) auf immaterielle Anlagewerte"
"a6602","Non-Recurring Depreciation and Amounts Written Down on Tangible Fixed Assets","6602","expense","","False","Amortissements et réductions de valeur non récurrents (dotations) sur immobilisations corporelles","Niet-recurrente afschrijvingen en waardeverminderingen (toevoeging) op materiële vaste activa","Nicht wiederkehrende Abschreibungen und Wertminderungen (Zuführungen) auf Sachanlagen"
"a661","Amounts Written Off Financial Fixed Assets (Appropriations)","661","expense","","False","Réductions de valeur sur immobilisations financières (dotation)","Waardeverminderingen op financiële vaste activa (toevoeging)","Wertminderungen auf Finanzanlagen (Zuführungen)"
"a66200","Provisions for Non-Recurring Operating Liabilities and Charges - Appropriations","66200","expense","","False","Provisions pour risques et charges d'exploitation non récurrents - Dotations","Voorzieningen voor niet-recurrente bedrijfsrisico's en -kosten - Toevoegingen","Rückstellungen für nicht wiederkehrende Betriebsrisiken und Aufwendungen - Zuführungen"
"a66201","Provisions for Non-Recurring Operating Liabilities and Charges - Uses (-)","66201","expense","","False","Provisions pour risques et charges d'exploitation non récurrents - Utilisations (-)","Voorzieningen voor niet-recurrente bedrijfsrisico's en -kosten - Bestedingen (-)","Rückstellungen für nicht wiederkehrende Betriebsrisiken und Aufwendungen - Verbrauch (-)"
"a66210","Provisions for Non-Recurring Financial Liabilities and Charges - Appropriations","66210","expense","","False","Provisions pour risques et charges financiers non récurrents - Dotations","Voorzieningen voor niet-recurrente financiële risico's en kosten - Toevoegingen","Rückstellungen für nicht wiederkehrende finanzielle Risiken und Aufwendungen - Zuführungen"
"a66211","Provisions for Non-Recurring Financial Liabilities and Charges - Uses (-)","66211","expense","","False","Provisions pour risques et charges financiers non récurrents - Utilisations (-)","Voorzieningen voor niet-recurrente financiële risico's en kosten - Bestedingen (-)","Rückstellungen für nicht wiederkehrende finanzielle Risiken und Aufwendungen - Verbrauch (-)"
"a6630","Capital Losses on Disposal of Intangible and Tangible Fixed Assets","6630","expense","","False","Moins-values sur réalisation d'immobilisations incorporelles et corporelles","Minderwaarden op de realisatie van immateriële en materiële vaste activa","Verluste aus dem Abgang von immateriellen Anlagewerten und Sachanlagen"
"a6631","Capital Losses on Disposal of Financial Fixed Assets","6631","expense","","False","Moins-values sur réalisation d'immobilisations financières","Minderwaarden op de realisatie van financiële vaste activa","Verluste aus dem Abgang von Gegenständen der Finanzanlagen"
"a664","Other Non-Recurring Operating Charges","664","expense","","False","Autres charges d'exploitation non récurrentes","Andere niet-recurrente bedrijfskosten","Sonstige nicht wiederkehrende betriebliche Aufwendungen"
"a668","Other Non-Recurring Financial Charges","668","expense","","False","Autres charges financières non récurrentes","Andere niet-recurrente financiële kosten","Sonstige nicht wiederkehrende Finanzaufwendungen"
"a6690","Non-Recurring Operating Charges Carried to Assets as Restructuring Costs (-)","6690","expense","","False","Charges d'exploitation non récurrentes portées à l'actif au titre de frais de restructuration (-)","Als herstructureringskosten geactiveerde niet-recurrente bedrijfskosten (-)","Nicht wiederkehrende betriebliche Aufwendungen, die als Restrukturierungskosten aktiviert wurden (-)"
"a6691","Non-Recurring Financial Charges Carried to Assets as Restructuring Costs (-)","6691","expense","","False","Charges financières non récurrentes portées à l'actif au titre de frais de restructuration (-)","Als herstructureringskosten geactiveerde niet-recurrente financiële kosten (-)","Nicht wiederkehrende Finanzaufwendungen, die als Restrukturierungskosten aktiviert wurden (-)"
"a6700","Income Taxes Paid and Withholding Taxes Due or Paid on the Result of the Current Period","6700","expense","account.account_tag_operating","False","Impôts et précomptes dus ou versés sur le résultat de l'exercice","Verschuldigde of gestorte belastingen en voorheffingen op het resultaat van het boekjaar","Geschuldete oder gezahlte Steuern und Steuervorabzüge auf das Ergebnis des Geschäftsjahres"
"a6701","Excess of Income Tax Prepayments and Withholding Taxes Paid Recorded Under Assets on the Result of the Current Period (-)","6701","expense","","False","Excédent de versements d'impôts et de précomptes porté à l'actif sur le résultat de l'exercice (-)","Geactiveerde overschotten van betaalde belastingen en voorheffingen op het resultaat van het boekjaar (-)","Aktivierte Überschüsse von gezahlten Steuern und Steuervorabzügen auf das Ergebnis des Geschäftsjahres (-)"
"a6702","Estimated Taxes Payable on the Result of the Current Period","6702","expense","","False","Charges fiscales estimées sur le résultat de l'exercice","Geraamde belastingen op het resultaat van het boekjaar","Geschätzte Steuern auf das Ergebnis des Geschäftsjahres"
"a6710","Additional Income Taxes Due or Paid on the Result of Prior Periods","6710","expense","","False","Suppléments d'impôts dus ou versés sur le résultat d'exercices antérieurs","Verschuldigde of gestorte belastingsupplementen op het resultaat van vorige boekjaren","Geschuldete oder gezahlte Steuernachforderungen auf das Ergebnis vorhergehender Geschäftsjahre"
"a6711","Estimated Additional Taxes on the Result of Prior Periods","6711","expense","","False","Suppléments d'impôts estimés sur le résultat d'exercices antérieurs","Geraamde belastingsupplementen op het resultaat van vorige boekjaren","Geschätzte Steuernachforderungen auf das Ergebnis vorhergehender Geschäftsjahre"
"a6712","Additional Charges for Income Taxes Provided For on the Result of Prior Periods","6712","expense","","False","Provisions fiscales constituées sur le résultat d'exercices antérieurs","Gevormde fiscale voorzieningen op het resultaat van vorige boekjaren","Gebildete Steuerrückstellungen auf das Ergebnis vorhergehender Geschäftsjahre"
"a672","Foreign Income Taxes on the Result of the Current Period","672","expense","","False","Impôts étrangers sur le résultat de l'exercice","Buitenlandse belastingen op het resultaat van het boekjaar","Ausländische Steuern auf das Ergebnis des Geschäftsjahres"
"a673","Foreign Income Taxes on the Result of Prior Periods","673","expense","","False","Impôts étrangers sur le résultat d'exercices antérieurs","Buitenlandse belastingen op het resultaat van vorige boekjaren","Ausländische Steuern auf das Ergebnis vorhergehender Geschäftsjahre"
"a680","Transfer to Deferred Taxes","680","expense","","False","Transfert aux impôts différés","Overboeking naar de uitgestelde belastingen","Zuführung zu den aufgeschobenen Steuern"
"a689","Transfer to Untaxed Reserves","689","expense","","False","Transfert aux réserves immunisées","Overboeking naar de belastingvrije reserves","Einstellung in die steuerfreien Rücklagen"
"a690","Loss of the Preceding Period Brought Forward","690","expense","","False","Perte reportée de l'exercice précédent","Overgedragen verlies van het vorige boekjaar","Verlusvortrag aus dem Vorjahr"
"a7000","Sales in Belgium (Trade Goods)","7000","income","account.account_tag_operating","False","Ventes en Belgique (marchandises)","Verkopen in België (handelsgoederen)","Verkäufe in Belgien (Waren)"
"a7001","Sales in the EU (Trade Goods)","7001","income","account.account_tag_operating","False","Ventes dans l'UE (marchandises)","Verkopen in de EU (handelsgoederen)","Verkäufe in der EU (Waren)"
"a7002","Sales for Export (Trade Goods)","7002","income","account.account_tag_operating","False","Ventes à l'exportation (marchandises)","Verkopen voor export (handelsgoederen)","Verkäufe für den Export (Waren)"
"a7010","Sales in Belgium (Finished Goods)","7010","income","account.account_tag_operating","False","Ventes en Belgique (produits finis)","Verkopen in België (gereed product)","Verkäufe in Belgien (Fertige Erzeugnisse)"
"a7011","Sales in the EU (Finished Goods)","7011","income","account.account_tag_operating","False","Ventes dans l'UE (produits finis)","Verkopen in de EU (gereed product)","Verkäufe in der EU (Fertige Erzeugnisse)"
"a7012","Sales for Export (Finished Goods)","7012","income","account.account_tag_operating","False","Ventes à l'exportation (produits finis)","Verkopen voor export (gereed product)","Verkäufe für den Export (Fertige Erzeugnisse)"
"a7050","Services in Belgium","7050","income","account.account_tag_operating","False","Prestations de services en Belgique","Dienstprestaties in België","Dienstleistungen in Belgien"
"a7051","Services in the EU","7051","income","account.account_tag_operating","False","Prestations de services dans l'UE","Dienstprestaties in de EU","Dienstleistungen in der EU"
"a7052","Services Outside of the EU","7052","income","account.account_tag_operating","False","Prestations de services hors l'UE","Dienstprestaties buiten de EU","Dienstleistungen außerhalb der EU"
"a708","Discounts, Allowances and Rebates Allowed (-)","708","income","","False","Remises, ristournes et rabais accordés (-)","Toegekende kortingen, ristorno's en rabatten (-)","Gewährte Preisnachlässe, Rückvergütungen und Rabatte (-)"
"a709000","Rounding of Payments in Euros","709000","income","","False","Arrondissement des paiements en Euros","Afronding van betalingen in euro","Rundung von Zahlungen in Euro"
"a712","Increase (Decrease) in Stocks of Work in Progress","712","income","","False","Variations des stocks d'en-cours de fabrication","Wijzigingen in de voorraad goederen in bewerking","Bestandsveränderung der Vorräte an unfertigen Erzeugnisse"
"a713","Increase (Decrease) in Stocks of Finished Goods","713","income","","False","Variations des stocks de produits finis","Wijzigingen in de voorraad gereed product","Bestandsveränderung der Vorräte an fertigen Erzeugnisse"
"a715","Increase (Decrease) in Stocks of Immovable Property Intended for Resale","715","income","","False","Variations des stocks des immeubles construits destinés à la vente","Wijzigingen in de voorraad onroerende goederen bestemd voor verkoop","Bestandsveränderung der Vorräte an zum Verkauf bestimmten unbeweglichen Gegenständen"
"a7170","Increase (Decrease) in Contracts in Progress - Acquisition Value","7170","income","","False","Variations des commandes en cours d'exécution - Valeur d'acquisition","Wijzigingen in de bestellingen in uitvoering - Aanschaffingswaarde","Bestandsveränderung an in Ausführung befindlichen Bestellungen - Anschaffungswert"
"a7171","Increase (Decrease) in Contracts in Progress - Profit Recognized","7171","income","","False","Variations des commandes en cours d'exécution - Bénéfice pris en compte","Wijzigingen in de bestellingen in uitvoering - Toegerekende winst","Bestandsveränderung an in Ausführung befindlichen Bestellungen - Zugewiesener Gewinn"
"a72","Produced Fixed Assets","72","income","","False","Production immobilisée","Geproduceerde vaste activa","Andere aktivierte Eigenleistungen"
"a741","Capital Profits on Ordinary Disposal of Tangible Fixed Assets","741","income","","False","Plus-values sur réalisations courantes d'immobilisations corporelles","Meerwaarden op de courante realisatie van materiële vaste activa","Erträge aus dem normalen Abgang von Sachanlagen"
"a742","Capital Profits on Disposal of Trade Debtors","742","income","","False","Plus-values sur réalisation de créances commerciales","Meerwaarden op de realisatie van handelsvorderingen","Mehrwerte bei Realisierung von Forderungen aus Lieferungen und Leistungen"
"a743","Miscellaneous Operating Income","743","income","","False","Produits d'exploitation divers","Diverse bedrijfsopbrengsten","Sonstige betriebliche Erträge"
"a750","Income From Financial Fixed Assets","750","income","account.account_tag_financing","False","Produits des immobilisations financières","Opbrengsten uit financiële vaste activa","Erträge aus Finanzanlagen"
"a751","Income From Current Assets","751","income","account.account_tag_financing","False","Produits des actifs circulants","Opbrengsten uit vlottende activa","Erträge aus Gegenständen des Umlaufvermögens"
"a752","Capital Profits on Disposal of Current Assets","752","income","account.account_tag_financing","False","Plus-values sur réalisation d'actifs circulants","Meerwaarden op de realisatie van vlottende activa","Erträge aus dem Abgang von Gegenständen des Umlaufvermögens"
"a754","Exchange Results","754","income","account.account_tag_financing","False","Différences de change","Wisselresultaten","Wechselkursdifferenzen"
"a755","Results From the Conversion of Foreign Currencies","755","income","account.account_tag_financing","False","Écarts de conversion des devises","Resultaten uit de omrekening van vreemde valuta","Differenzen aus der Umrechnung von Fremdwährungen"
"a756","Miscellaneous Financial Income","756","income","account.account_tag_financing","False","Produits financiers divers","Diverse financiële opbrengsten","Sonstige Finanzerträge"
"a757000","Obtained Financial Discounts","757000","income","account.account_tag_financing","False","Escomptes financiers obtenus","Verkregen financiële kortingen","Erlangte finanzielle Rabatte"
"a757100","Positive Payment Differences","757100","income","account.account_tag_financing","False","Différences de paiement positives","Positieve betalingsverschillen","Positive Zahlungsdifferenzen"
"a7600","Write-Back of Depreciation and Amounts Written Down on Intangible Fixed Assets","7600","income","account.account_tag_investing","False","Reprises d'amortissements et de réductions de valeur sur immobilisations incorporelles","Terugneming van afschrijvingen en waardeverminderingen op immateriële vaste activa","Rücknahme von Abschreibungen oder Wertminderungen auf immaterielle Anlagewerte"
"a7601","Write-Back of Depreciation and Amounts Written Down on Tangible Fixed Assets","7601","income","","False","Reprises d'amortissements et de réductions de valeur sur immobilisations corporelles","Terugneming van afschrijvingen en waardeverminderingen op materiële vaste activa","Rücknahme von Abschreibungen oder Wertminderungen auf Sachanlagen"
"a761","Write-Back of Amounts Written Down Financial Fixed Assets","761","income","","False","Reprises de réductions de valeur sur immobilisations financières","Terugneming van waardeverminderingen op financiële vaste activa","Rücknahme von Wertminderungen auf Finanzanlagen"
"a7620","Write-Back of Provisions for Non-Recurring Operating Liabilities and Charges","7620","income","","False","Reprises de provisions pour risques et charges d'exploitation non récurrents","Terugneming van voorzieningen voor niet-recurrente bedrijfsrisico's en -kosten","Auflösung von Rückstellungen für nicht wiederkehrende Betriebsrisiken und Aufwendungen"
"a7621","Write-Back of Provisions for Non-Recurring Financial Liabilities and Charges","7621","income","","False","Reprises de provisions pour risques et charges financiers non récurrents","Terugneming van voorzieningen voor niet-recurrente financiële risico's en kosten","Auflösung von Rückstellungen für nicht wiederkehrende finanzielle Risiken und Aufwendungen"
"a7630","Capital Profit on Disposal of Intangible and Tangible Fixed Assets","7630","income","","False","Plus-values sur réalisation d'immobilisations incorporelles et corporelles","Meerwaarden op de realisatie van immateriële en materiële vaste activa","Mehrwerte aus dem Abgang von immateriellen Anlagewerten und Sachanlagen"
"a7631","Capital Profit on Disposal of Financial Fixed Assets","7631","income","","False","Plus-values sur réalisation d'immobilisations financières","Meerwaarden op de realisatie van financiële vaste activa","Mehrwerte aus dem Abgang von Finanzanlagen"
"a764","Other Non-Recurring Operating Income","764","income","","False","Autres bénéfices d'exploitation non récurrents","Andere niet-recurrente bedrijfsopbrengsten","Sonstige nicht wiederkehrende betriebliche Erträge"
"a769","Other Non-Recurring Financial Income","769","income","","False","Autres produits financiers non récurrents","Andere niet-recurrente financiële opbrengsten","Sonstige nicht wiederkehrende Finanzerträge"
"a780","Transfer From Deferred Taxes","780","income","account.account_tag_operating","False","Prélèvement sur les impôts différés","Onttrekking aan de uitgestelde belastingen","Auflösung von aufgeschobenen Steuern"
"a789","Transfer From Untaxed Reserves","789","income","account.account_tag_operating","False","Prélèvement sur les réserves immunisées","Onttrekking aan de belastingvrije reserves","Entnahmen aus den steuerfreien Rücklagen"

```

## File: data\template\account.account-be_asso.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@fr","name@nl","name@de"
"a100","Association or Foundation Funds","100","equity","","False","Fonds de l'association ou de la fondation","Fondsen van de vereniging of stichting","Vermögen der Vereinigung oder Stiftung"
"a130","Funds for Investments","130","equity","","False","Fonds affectés pour investissements","Fondsen bestemd voor investeringen","Für Investitionen bestimmtes Vermögen"
"a131","Funds for Social Liabilities","131","equity","","False","Fonds affectés pour passif social","Fondsen bestemd voor sociaal passief","Für Sozialverbindlichkeiten bestimmtes Vermögen"
"a139","Other Allocated Funds and Other Reserves","139","equity","","False","Autres fonds affectés et autres réserves","Andere bestemde fondsen en andere reserves","Anderes zweckgebundenes Vermögen und andere Rücklagen"
"a15","Capital Subsidies","15","equity","","False","Subsides en capital","Kapitaalsubsidies","Kapitalsubventionen"
"a167","Provisions for Subsidies and Legacies to Reimburse and Gifts With a Recovery Right","167","liability_non_current","","False","Provisions pour remboursement de subsides, legs et dons avec droit de reprise","Voorzieningen voor terug te betalen subsidies, legaten en schenkingen met terugnemingsrecht","Rückstellungen für zurückzuzahlende Subventionen, Legate und Schenkungen mit Rücknahmerecht"
"a168","Deferred Taxes","168","liability_non_current","","False","Impôts différés","Uitgestelde belastingen","Aufgeschobene Steuern"
"a170","Subordinated Loans (> 1 Year)","170","liability_non_current","","False","Emprunts subordonnés (> 1 an)","Achtergestelde leningen (> 1 jaar)","Nachrangige Anleihen (> 1 Jahr)"
"a171","Unsubordinated Debentures (> 1 Year)","171","liability_non_current","","False","Emprunts obligataires non subordonnés (> 1 an)","Niet achtergestelde obligatieleningen (> 1 jaar)","Nicht nachrangige Schuldverschreibungsanleihen (> 1 Jahr)"
"a1790","Other Amounts Payable - Interest-Bearing (> 1 Year)","1790","liability_non_current","","False","Autres dettes - Productives d'intérêts (> 1 an)","Overige schulden - Rentedragend (> 1 jaar)","Sonstige Verbindlichkeiten - Verzinslich (> 1 Jahr)"
"a1791","Other Amounts Payable - Non Interest-Bearing or Linked to an Abnormally Low Interest Rate (> 1 Year)","1791","liability_non_current","","False","Autres dettes - Non productives d'intérêts ou assorties d'un intérêt anormalement faible (> 1 an)","Overige schulden - Niet-rentedragend of gekoppeld aan een abnormaal lage rente (> 1 jaar)","Sonstige Verbindlichkeiten - Unverzinslich oder mit einem ungewöhnlich niedrigen Zinssatz (> 1 Jahr)"
"a200","Formation Expenses","200","asset_non_current","","False","Frais de constitution","Kosten van oprichting","Errichtungsaufwendungen"
"a2912","Other Amounts Receivable - Subsidies to Be Received (> 1 Year)","2912","asset_non_current","","False","Autres créances - Subsides à recevoir (> 1 an)","Overige vorderingen - Te ontvangen subsidies (> 1 jaar)","Sonstige Forderungen - Besitzwechsel (> 1 Jahr)"
"a2915","Other Amounts Receivable - Non Interest-Bearing or Linked to an Abnormally Low Interest Rate (> 1 Year)","2915","asset_non_current","","False","Autres créances - Créances non productives d'intérêts ou assorties d'un intérêt anormalement faible (> 1 an)","Overige vorderingen - Niet-rentedragende vorderingen of gekoppeld aan een abnormaal lage rente (> 1 jaar)","Sonstige Forderungen - Unverzinsliche Forderungen oder Forderungen mit einem ungewöhnlich niedrigen Zinssatz (> 1 Jahr)"
"a2916","Other Amounts Receivable - Doubtful Amounts (> 1 Year)","2916","asset_non_current","","False","Autres créances - Créances douteuses (> 1 an)","Overige vorderingen - Dubieuze debiteuren (> 1 jaar)","Sonstige Forderungen - Zweifelhafte Forderungen (> 1 Jahr)"
"a413","Subsidies to Be Received","413","asset_current","","False","Subsides à recevoir","Te ontvangen subsidies","Zu erhaltende Subventionen"
"a415","Non Interest-Bearing Amounts Receivable or Linked to an Abnormally Low Interest Rate","415","asset_current","","False","Créances non productives d'intérêts ou assorties d'un intérêt anormalement faible","Niet-rentedragende vorderingen of gekoppeld aan een abnormaal lage rente","Unverzinsliche Forderungen oder Forderungen mit einem ungewöhnlich niedrigen Zinssatz"
"a420","Subordinated Loans (> 1 Year Falling Due < 1 Year)","420","liability_current","","False","Emprunts subordonnés (> 1 an écheant < 1 an)","Achtergestelde leningen (> 1 jaar vervallend < 1 jaar)","Nachrangige Anleihen (> 1 Jahr fällig < 1 Jahr)"
"a421","Unsubordinated Debentures (> 1 Year Falling Due < 1 Year)","421","liability_current","","False","Emprunts obligataires non subordonnés (> 1 an écheant < 1 an)","Niet achtergestelde obligatieleningen (> 1 jaar vervallend < 1 jaar)","Nicht nachrangige Schuldverschreibungsanleihen (> 1 Jahr fällig < 1 Jahr)"
"a4290","Other Amounts Payable - Interest-Bearing (> 1 Year Falling Due < 1 Year)","4290","liability_current","","False","Autres dettes - Productives d'intérêts (> 1 an écheant < 1 an)","Overige schulden - Rentedragend (> 1 jaar vervallend < 1 jaar)","Sonstige Verbindlichkeiten - Verzinslich (> 1 Jahr fällig < 1 Jahr)"
"a4291","Other Amounts Payable - Non Interest-Bearing or Linked to an Abnormally Low Interest Rate (> 1 Year Falling Due < 1 Year)","4291","liability_current","","False","Autres dettes - Non productives d'intérêts ou assorties d'un intérêt anormalement faible (> 1 an écheant < 1 an)","Overige schulden - Niet-rentedragend of gekoppeld aan een abnormaal lage rente (> 1 jaar vervallend < 1 jaar)","Sonstige Verbindlichkeiten - Unverzinslich oder mit einem ungewöhnlich niedrigen Zinssatz (> 1 Jahr fällig < 1 Jahr)"
"a4890","Sundry Amounts Payable - Interest-Bearing","4890","liability_current","","False","Autres dettes - Productives d'intérêts","Andere diverse schulden - Rentedragend","Sonstige Verbindlichkeiten - Verzinslich"
"a4891","Sundry Amounts Payable - Non Interest-Bearing or Linked to an Abnormally Low Interest Rate","4891","liability_current","","False","Autres dettes - Non productives d'intérêts ou assorties d'un intérêt anormalement faible","Andere diverse schulden - Niet-rentedragend of gekoppeld aan een abnormaal lage rente","Sonstige Verbindlichkeiten - Unverzinslich oder mit einem ungewöhnlich niedrigen Zinssatz"
"a500","Current Investments Other Than Shares, Fixed-Income Investments and Term Deposits - Acquisition Value","500","asset_current","","False","Placements de trésorerie autres que actions et parts, titres à revenu fixe et dépôts à terme - Valeur d'acquisition","Geldbeleggingen andere dan aandelen, vastrentende effecten en termijndeposito's - Aanschaffingswaarde","Geldanlagen außer Anteilen, festverzinslichen Wertpapieren und Terminkonten - Anschaffungswert"
"a509","Current Investments Other Than Shares, Fixed-Income Investments and Term Deposits - Amounts Written Down - Recorded (-)","509","asset_current","","False","Placements de trésorerie autres que actions et parts, titres à revenu fixe et dépôts à terme - Réductions de valeurs actées (-)","Geldbeleggingen andere dan aandelen, vastrentende effecten en termijndeposito's - Geboekte waardeverminderingen (-)","Geldanlagen außer Anteilen, festverzinslichen Wertpapieren und Terminkonten - Gebuchte Wertminderungen (-)"
"a510","Shares - Acquisition Value","510","asset_current","","False","Actions et parts - Valeur d'acquisition","Aandelen - Aanschaffingswaarde","Anteile - Anschaffungswert"
"a519","Shares - Amounts Written Down - Recorded (-)","519","asset_current","","False","Actions et parts - Réductions de valeur actées (-)","Aandelen - Geboekte waardeverminderingen (-)","Anteile - Gebuchte Wertminderungen (-)"
"a6380","Provisions for Subsidies and Legacies to Reimburse and Gifts With a Recovery Right - Appropriations","6380","expense","","False","Provisions pour subsides et legs à rembourser et pour dons avec droit de reprise - Dotations","Voorzieningen voor terug te betalen subsidies en legaten en voor schenkingen met terugnemingsrecht - Toevoegingen","Rückstellungen für zurückzuzahlende Subventionen und Legate und für Schenkungen mit Rücknahmerecht - Zuführungen"
"a6381","Provisions for Subsidies and Legacies to Reimburse and Gifts With a Recovery Right - Uses and Write-Backs (-)","6381","expense","","False","Provisions pour subsides et legs à rembourser et pour dons avec droit de reprise - Utilisations et reprises (-)","Voorzieningen voor terug te betalen subsidies en legaten en voor schenkingen met terugnemingsrecht - Bestedingen en terugnemingen (-)","Rückstellungen für zurückzuzahlende Subventionen und Legate und für Schenkungen mit Rücknahmerecht - Verbrauch und Auflösungen (-)"
"a6390","Provisions for Other Liabilities and Charges - Appropriations","6390","expense","","False","Provisions pour autres risques et charges - Dotations","Voorzieningen voor andere risico's en kosten - Toevoegingen","Rückstellungen für sonstige Risiken und Aufwendungen - Zuführungen"
"a6391","Provisions for Other Liabilities and Charges - Uses and Write-Backs (-)","6391","expense","","False","Provisions pour autres risques et charges - Utilisations et reprises (-)","Voorzieningen voor andere risico's en kosten - Bestedingen en terugnemingen (-)","Rückstellungen für sonstige Risiken und Aufwendungen - Verbrauch und Auflösungen (-)"
"a643","Gifts","643","expense","","False","Dons","Schenkingen","Schenkungen"
"a644","Miscellaneous Operating Charges","644","expense","","False","Charges d'exploitation diverses","Diverse bedrijfskosten","Sonstige betriebliche Aufwendungen"
"a691","Transfer to Allocated Funds and Other Reserves","691","expense","","False","Transfert aux fonds affectés et autres réserves","Overboeking naar de bestemde fondsen en andere reserves","Einstellung in das zweckgebundene Vermögen und die sonstigen Rücklagen"
"a692","Positive Result to be Carried Forward","692","expense","","False","Résultat positif à reporter","Over te dragen positief resultaat","Gewinnvortrag auf neue Rechnung"
"a730","Membership Fees","730","income","","False","Cotisations","Lidgelden","Beiträge"
"a731","Gifts","731","income","","False","Dons","Schenkingen","Schenkungen"
"a732","Legacies","732","income","","False","Legs","Legaten","Legate"
"a733","Subsidies","733","income","","False","Subsides","Subsidies","Subventionen"
"a77","Adjustment of Income Taxes","77","income","account.account_tag_operating","False","Régularisation d'impôts","Regularisering van belastingen","Steuererstattungen"
"a790","Positive Result From the Preceding Period Brought Forward","790","income","","False","Résultat positif de l'exercice antérieur reporté","Overgedragen positief resultaat van het vorige boekjaar","Gewinnvortrag aus dem Vorjahr"
"a791","Other Reserves","791","income","","False","Autres réserves","Andere reserves","Sonstige Rücklagen"
"a792","Negative Result to Be Carried Forward","792","income","","False","Résultat négatif à reporter","Over te dragen negatief resultaat","Verlustvortrag auf neue Rechnung"

```

## File: data\template\account.account-be_comp.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@fr","name@nl","name@de"
"a100","Issued Capital","100","equity","","False","Capital souscrit","Geplaatst kapitaal","Gezeichnetes Kapital"
"a101","Uncalled Capital (-)","101","equity","","False","Capital non appelé (-)","Niet opgevraagd kapitaal (-)","Nicht eingefordertes Kapital (-)"
"a1100","Contribution Available Excluding Capital - Share Premium","1100","equity","","False","Apport disponible hors capital - Prime d'émission","Beschikbare inbreng buiten kapitaal - Uitgiftepremie","Verfügbare Einlage außerhalb des Kapitals - Agio"
"a1109","Contribution Available Excluding Capital - Other","1109","equity","","False","Apport disponible hors capital - Autres","Beschikbare inbreng buiten kapitaal - Andere","Verfügbare Einlage außerhalb des Kapitals - Sonstige"
"a1110","Contribution Not Available Excluding Capital - Share Premium","1110","equity","","False","Apport indisponible hors capital - Prime d'émission","Onbeschikbare inbreng buiten kapitaal - Uitgiftepremie","Nicht verfügbare Einlage außerhalb des Kapitals - Agio"
"a1119","Contribution Not Available Excluding Capital - Other","1119","equity","","False","Apport indisponible hors capital - Autres","Onbeschikbare inbreng buiten kapitaal - Andere","Nicht verfügbare Einlage außerhalb des Kapitals - Sonstige"
"a123","Revaluation Surpluses on Stocks","123","equity","","False","Plus-values de réévaluation sur stocks","Herwaarderingsmeerwaarden op voorraden","Neubewertungsrücklagen auf Vorräte"
"a130","Legal Reserve","130","equity","","False","Réserves légales","Wettelijke reserves","Gesetzliche Rücklage"
"a1311","Reserves Not Available Statutorily","1311","equity","","False","Réserves statutairement indisponibles","Statutair onbeschikbare reserves","Satzungsgemäße nicht verfügbare Rücklagen"
"a1312","Reserves Not Available in Respect of Own Shares Held","1312","equity","","False","Réserve pour actions propres","Reserve voor eigen aandelen","Rücklagen für eigene Aktien oder Anteile"
"a1313","Financial Support","1313","equity","","False","Soutien financier","Financiële steunverlening","Finanzielle Unterstützung"
"a1319","Other Reserves Not Available","1319","equity","","False","Autres réserves indisponibles","Andere onbeschikbare reserves","Sonstige nicht verfügbare Rücklagen"
"a133","Available Reserves","133","equity","","False","Réserves disponibles","Beschikbare reserves","Verfügbare Rücklagen"
"a15","Investment Grants","15","equity","","False","Subsides en capital","Kapitaalsubsidies","Kapitalsubventionen"
"a1680","Deferred Taxes on Capital Subsidies","1680","liability_non_current","","False","Impôts différés afférents à des subsides en capital","Uitgestelde belastingen op kapitaalsubsidies","Aufgeschobene Steuern auf Kapitalsubventionen"
"a1681","Deferred Taxes on Gain on Disposal of Intangible Fixed Assets","1681","liability_non_current","","False","Impôts différés afférents à des plus-values réalisées sur immobilisations incorporelles","Uitgestelde belastingen op gerealiseerde meerwaarden op immateriële vaste activa","Aufgeschobene Steuern auf realisierte Mehrwerte auf immaterielle Anlagewerte"
"a1682","Deferred Taxes on Gain on Disposal of Tangible Fixed Assets","1682","liability_non_current","","False","Impôts différés afférents à des plus-values réalisées sur immobilisations corporelles","Uitgestelde belastingen op gerealiseerde meerwaarden op materiële vaste activa","Aufgeschobene Steuern auf realisierte Mehrwerte auf Sachanlagen"
"a1687","Deferred Taxes on Gain on Disposal of Securities Issued by Belgian Public Authorities","1687","liability_non_current","","False","Impôts différés afférents à des plus-values réalisées sur titres émis par le secteur public belge","Uitgestelde belastingen op gerealiseerde meerwaarden op effecten die zijn uitgegeven door de Belgische openbare sector","Aufgeschobene Steuern auf realisierte Mehrwerte auf Wertpapiere, die vom belgischen öffentlichen Sektor ausgegeben wurden"
"a1688","Foreign Deferred Taxes","1688","liability_non_current","","False","Impôts différés étrangers","Buitenlandse uitgestelde belastingen","Ausländische aufgeschobene Steuern"
"a1700","Subordinated Loans - Convertible Bonds (> 1 Year)","1700","liability_non_current","","False","Emprunts subordonnés - Convertibles (> 1 an)","Achtergestelde leningen - Converteerbaar (> 1 jaar)","Nachrangige Anleihen - Wandelanleihen (> 1 Jahr)"
"a1701","Subordinated Loans - Non-Convertible Bonds (> 1 Year)","1701","liability_non_current","","False","Emprunts subordonnés - Non convertibles (> 1 an)","Achtergestelde leningen - Niet converteerbaar (> 1 jaar)","Nachrangige Anleihen - Nicht wandelbare Anleihen (> 1 Jahr)"
"a1710","Unsubordinated Debentures - Convertible (> 1 Year)","1710","liability_non_current","","False","Emprunts obligataires non subordonnés - Convertibles (> 1 an)","Niet-achtergestelde obligatieleningen - Converteerbaar (> 1 jaar)","Nicht nachrangige Anleihen - Wandelanleihen (> 1 Jahr)"
"a1711","Unsubordinated Debentures - Non-Convertible (> 1 Year)","1711","liability_non_current","","False","Emprunts obligataires non subordonnés - Non convertibles (> 1 an)","Niet-achtergestelde obligatieleningen - Niet converteerbaar (> 1 jaar)","Nicht nachrangige Anleihen - Nicht wandelbare Anleihen (> 1 Jahr)"
"a179","Other Amounts Payable (> 1 Year)","179","liability_non_current","","False","Dettes diverses (> 1 an)","Overige schulden (> 1 jaar)","Sonstige Verbindlichkeiten (> 1 Jahr)"
"a19","Advance to Shareholders on the Distribution of Net Assets (-)","19","liability_non_current","","False","Acompte aux associés sur le partage de l'actif net (-)","Voorschot aan de vennoten op de verdeling van het netto-actief (-)","Vorschuss an die Gesellschafter auf der Verteilung des Nettoaktiva (-)"
"a200","Formation or Capital Increase or Contribution Increase Expenses","200","asset_non_current","","False","Frais de constitution, d'augmentation de capital ou d'augmentation de l'apport","Kosten van oprichting, kapitaalverhoging of verhoging van de inbreng","Errichtungsaufwendungen und Kapital- und Einlagenerhöhungskosten"
"a2917","Other Amounts Receivable - Doubtful Amounts Receivable (> 1 Year)","2917","asset_non_current","","False","Autres créances - Créances douteuses (> 1 an)","Overige vorderingen - Dubieuze debiteuren (> 1 jaar)","Sonstige Forderungen - Zweifelhafte Forderungen (> 1 Jahr)"
"a410","Called Up Capital or Contribution, Unpaid","410","asset_current","","False","Capital ou apport appelé, non versé","Opgevraagd, niet gestort kapitaal of inbreng","Eingefordertes, noch nicht eingezahltes Kapital oder Einlagen"
"a4120","Belgian Income Taxes to Be Recovered","4120","asset_current","","False","Impôts belges sur le résultat à récupérer","Terug te vorderen Belgische winstbelastingen","Zurückzuerstattende Belgische Ertragsteuern"
"a4200","Subordinated Loans - Convertible Bonds (> 1 Year Falling Due < 1 Year)","4200","liability_current","","False","Emprunts subordonnés - Convertibles (> 1 an écheant < 1 an)","Achtergestelde leningen - Converteerbaar (> 1 jaar vervallend < 1 jaar)","Nachrangige Anleihen - Wandelanleihen (> 1 Jahr fällig < 1 Jahr)"
"a4201","Subordinated Loans - Non-Convertible Bonds (> 1 Year Falling Due < 1 Year)","4201","liability_current","","False","Emprunts subordonnés - Non convertibles (> 1 an écheant < 1 an)","Achtergestelde leningen - Niet converteerbaar (> 1 jaar vervallend < 1 jaar)","Nachrangige Anleihen - Nicht wandelbare Anleihen (> 1 Jahr fällig < 1 Jahr)"
"a4210","Unsubordinated Debentures - Convertible (> 1 Year Falling Due < 1 Year)","4210","liability_current","","False","Emprunts obligataires non subordonnés - Convertibles (> 1 an écheant < 1 an)","Niet-achtergestelde obligatieleningen - Converteerbaar (> 1 jaar vervallend < 1 jaar)","Nicht nachrangige Anleihen - Wandelanleihen (> 1 Jahr fällig < 1 Jahr)"
"a4211","Unsubordinated Debentures - Non-Convertible (> 1 Year Falling Due < 1 Year)","4211","liability_current","","False","Emprunts obligataires non subordonnés - Non convertibles (> 1 an écheant < 1 an)","Niet-achtergestelde obligatieleningen - Niet converteerbaar (> 1 jaar vervallend < 1 jaar)","Nicht nachrangige Anleihen - Nicht wandelbare Anleihen (> 1 Jahr fällig < 1 Jahr)"
"a429","Other Amounts Payable (> 1 Year Falling Due < 1 Year)","429","liability_current","","False","Dettes diverses (> 1 an écheant < 1 an)","Overige schulden (> 1 jaar vervallend < 1 jaar)","Sonstige Verbindlichkeiten (> 1 Jahr fällig < 1 Jahr)"
"a4500","Estimated Belgian Income Taxes Payable","4500","liability_current","","False","Dettes fiscales estimées - Impôts belges sur le résultat","Geraamd bedrag der belastingschulden - Belgische winstbelastingen","Geschätzte Steuerschulden - Belgische Ertragsteuern"
"a4520","Belgian Income Taxes Payable","4520","liability_current","","False","Impôts belges à payer sur le résultat","Te betalen Belgische winstbelastingen","Zu zahlende belgische Ertragsteuern"
"a470","Dividends and Director's Fees Relating to Prior Financial Periods","470","liability_current","","False","Dividendes et tantièmes d'exercices antérieurs","Dividenden en tantièmes over vorige boekjaren","Dividenden und Tantiemen vorhergehender Geschäftsjahre"
"a471","Dividends - Current Financial Period","471","liability_current","","False","Dividendes de l'exercice","Dividenden over het boekjaar","Dividenden des Geschäftsjahres"
"a472","Director's Fees - Current Financial Period","472","liability_current","","False","Tantièmes de l'exercice","Tantièmes over het boekjaar","Tantiemen des Geschäftsjahres"
"a473","Other Beneficiaries","473","liability_current","","False","Autres allocataires","Andere rechthebbenden","Sonstige Berechtigte"
"a489","Sundry Amounts Payable","489","liability_current","","False","Autres dettes diverses","Andere diverse schulden","Sonstige Verbindlichkeiten"
"a50","Own Shares","50","asset_current","","False","Actions propres","Eigen aandelen","Eigene Anteile"
"a5100","Shares - Acquisition Value","5100","asset_current","","False","Actions et parts - Valeur d'acquisition","Aandelen - Aanschaffingswaarde","Aktien und Anteile - Anschaffungswert"
"a5101","Current Investments Other Than Fixed-Income Investments - Acquisition Value","5101","asset_current","","False","Placements de trésorerie autres que placements à revenu fixe - Valeur d'acquisition","Geldbeleggingen andere dan vastrentende beleggingen - Aanschaffingswaarde","Geldanlagen außer festverzinsliche Geldanlagen - Anschaffungswert"
"a5190","Shares - Amounts Written Down - Recorded (-)","5190","asset_current","","False","Actions et parts - Réductions de valeur actées (-)","Aandelen - Geboekte waardeverminderingen (-)","Aktien und Anteile - Gebuchte Wertminderungen (-)"
"a5191","Current Investments Other Than Fixed-Income Investments - Amounts Written Down - Recorded (-)","5191","asset_current","","False","Placements de trésorerie autres que placements à revenu fixe - Réductions de valeur actées (-)","Geldbeleggingen andere dan vastrentende beleggingen - Geboekte waardeverminderingen (-)","Geldanlagen außer festverzinsliche Geldanlagen - Gebuchte Wertminderungen (-)"
"a6380","Provisions for Liabilities and Charges - Appropriations","6380","expense","","False","Provisions pour autres risques et charges - Dotations","Voorzieningen voor andere risico's en kosten - Toevoegingen","Rückstellungen für sonstige Risiken und Aufwendungen - Zuführungen"
"a6381","Provisions for Liabilities and Charges - Uses and Write-Backs (-)","6381","expense","","False","Provisions pour autres risques et charges - Utilisations et reprises (-)","Voorzieningen voor andere risico's en kosten - Bestedingen en terugnemingen (-)","Rückstellungen für sonstige Risiken und Aufwendungen - Verbrauch und Auflösungen (-)"
"a643","Miscellaneous Operating Charges","643","expense","","False","Charges d'exploitation diverses","Diverse bedrijfskosten","Sonstige betriebliche Aufwendungen"
"a691","Appropriations to Contribution","691","expense","","False","Affectations à l'apport","Toevoeging aan de inbreng","Zuweisungen an die Einlagen"
"a6920","Appropriations to Legal Reserve","6920","expense","","False","Dotation à la réserve légale","Toevoeging aan de wettelijke reserve","Zuweisungen an die gesetzliche Rücklage"
"a6921","Appropriations to Other Reserves","6921","expense","","False","Dotation aux autres réserves","Toevoeging aan de overige reserves","Zuweisungen an die sonstigen Rücklagen"
"a693","Profits to Be Carried Forward","693","expense","","False","Bénéfices à reporter","Over te dragen winst","Gewinnvortrag auf neue Rechnung"
"a694","Compensation for Contributions","694","expense","","False","Rémunération de l'apport","Vergoeding van de inbreng","Vergütung der Einlage"
"a695","Directors' or Managers' Entitlements","695","expense","","False","Distribution aux administrateurs ou gérants","Uitkering aan bestuurders of zaakvoerders","Verteilung zugunsten der Verwalter oder Geschäftsführer"
"a696","Employees' Entitlements","696","expense","","False","Distribution aux employés","Uitkering aan werknemers","Verteilung zugunsten der Arbeitnehmer"
"a697","Other Beneficiaries' Entitlements","697","expense","","False","Distribution aux autres allocataires","Uitkering aan andere rechthebbenden","Verteilung zugunsten andere Berechtigte"
"a740","Operating Subsidies and Compensatory Notes","740","income","account.account_tag_operating","False","Subsides d'exploitation et montants compensatoires","Bedrijfssubsidies en compenserende bedragen","Betriebssubventionen und Ausgleichszahlungen"
"a753","Capital and Interest Subsidies","753","income","account.account_tag_financing","False","Subsides en capital et en intérêts","Kapitaal- en interestsubsidies","Kapital- und Zinssubventionen"
"a7710","Adjustment of Taxes Due or Paid","7710","income","account.account_tag_operating","False","Régularisations d'impôts dus ou versés","Regularisering van verschuldigde of betaalde belastingen","Erstattung geschuldeter oder gezahlter Ertragssteuern"
"a7711","Adjustment of Estimated Taxes","7711","income","account.account_tag_operating","False","Régularisations d'impôts estimés","Regularisering van geraamde belastingen","Erstattung geschätzter Steuern"
"a7712","Tax Provisions Written Back","7712","income","","False","Reprises de provisions fiscales","Terugneming van fiscale voorzieningen","Auflösung von Steuerrückstellungen"
"a773","Adjustment of Foreign Income Taxes","773","income","","False","Régularisations d'impôts étrangers sur le résultat","Regularisering van buitenlandse belastingen op het resultaat","Ausländische Ertragssteuern"
"a790","Profit From the Preceding Period Brought Forward","790","income","","False","Bénéfice reporté de l'exercice précédent","Overgedragen winst van het vorige boekjaar","Gewinnvortrag aus dem Vorjahr"
"a791","Transfer From Contributions","791","income","","False","Prélèvements sur l'apport","Onttrekking aan de inbreng","Entnahmen aus der Einlage"
"a792","Transfer From Reserves","792","income","","False","Prélèvements sur les réserves","Onttrekking aan de reserves","Entnahmen aus den Rücklagen"
"a793","Losses to Be Carried Forward","793","income","","False","Perte à reporter","Over te dragen verlies","Verlustvortrag auf neue Rechnung"
"a794","Shareholders' (or Owners') Contribution in Respect of Losses","794","income","","False","Intervention des associés (ou du propriétaire) dans la perte","Tussenkomst van vennoten (of van de eigenaar) in het verlies","Teilnahme der Gesellschafter (oder des Eigentümers) am Verlust"

```

## File: data\template\account.fiscal.position-be.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@fr","name@nl"
"fiscal_position_template_1","1","Belgium B2B","1","1","base.be","","","","","","Belgique B2B","België B2B"
"fiscal_position_template_5","2","EU B2C","1","","","base.europe","","","","","EU B2C","EU B2C"
"fiscal_position_template_3","3","Intra-Community","1","1","","base.europe","attn_VAT-OUT-00-S","attn_VAT-OUT-00-EU-S","","","Intracommunautaire","Intracommunautair"
"","","","","","","","attn_VAT-OUT-00-L","attn_VAT-OUT-00-EU-L","","","",""
"","","","","","","","attn_VAT-OUT-06-S","attn_VAT-OUT-00-EU-S","","","",""
"","","","","","","","attn_VAT-OUT-06-L","attn_VAT-OUT-00-EU-L","","","",""
"","","","","","","","attn_VAT-OUT-12-S","attn_VAT-OUT-00-EU-S","","","",""
"","","","","","","","attn_VAT-OUT-12-L","attn_VAT-OUT-00-EU-L","","","",""
"","","","","","","","attn_VAT-OUT-21-S","attn_VAT-OUT-00-EU-S","","","",""
"","","","","","","","attn_VAT-OUT-21-L","attn_VAT-OUT-00-EU-L","","","",""
"","","","","","","","attn_VAT-IN-V81-00","attn_VAT-IN-V81-00-EU","","","",""
"","","","","","","","attn_VAT-IN-V81-06","attn_VAT-IN-V81-06-EU","","","",""
"","","","","","","","attn_VAT-IN-V81-12","attn_VAT-IN-V81-12-EU","","","",""
"","","","","","","","attn_VAT-IN-V81-21","attn_VAT-IN-V81-21-EU","","","",""
"","","","","","","","attn_VAT-IN-V82-00-S","attn_VAT-IN-V82-00-EU-S","","","",""
"","","","","","","","attn_VAT-IN-V82-00-G","attn_VAT-IN-V82-00-EU-G","","","",""
"","","","","","","","attn_VAT-IN-V82-06-S","attn_VAT-IN-V82-06-EU-S","","","",""
"","","","","","","","attn_VAT-IN-V82-06-G","attn_VAT-IN-V82-06-EU-G","","","",""
"","","","","","","","attn_VAT-IN-V82-12-S","attn_VAT-IN-V82-12-EU-S","","","",""
"","","","","","","","attn_VAT-IN-V82-12-G","attn_VAT-IN-V82-12-EU-G","","","",""
"","","","","","","","attn_VAT-IN-V82-21-S","attn_VAT-IN-V82-21-EU-S","","","",""
"","","","","","","","attn_VAT-IN-V82-21-G","attn_VAT-IN-V82-21-EU-G","","","",""
"","","","","","","","attn_VAT-IN-V83-00","attn_VAT-IN-V83-00-EU","","","",""
"","","","","","","","attn_VAT-IN-V83-06","attn_VAT-IN-V83-06-EU","","","",""
"","","","","","","","attn_VAT-IN-V83-12","attn_VAT-IN-V83-12-EU","","","",""
"","","","","","","","attn_VAT-IN-V83-21","attn_VAT-IN-V83-21-EU","","","",""
"","","","","","","","","","a7000","a7001","",""
"","","","","","","","","","a7010","a7011","",""
"","","","","","","","","","a7050","a7051","",""
"fiscal_position_template_2","4","Import/Export","1","","","","attn_VAT-OUT-00-S","attn_VAT-OUT-00-ROW","","","Importation/Exportation","Import/Export"
"","","","","","","","attn_VAT-OUT-00-L","attn_VAT-OUT-00-ROW","","","",""
"","","","","","","","attn_VAT-OUT-06-S","attn_VAT-OUT-00-ROW","","","",""
"","","","","","","","attn_VAT-OUT-06-L","attn_VAT-OUT-00-ROW","","","",""
"","","","","","","","attn_VAT-OUT-12-S","attn_VAT-OUT-00-ROW","","","",""
"","","","","","","","attn_VAT-OUT-12-L","attn_VAT-OUT-00-ROW","","","",""
"","","","","","","","attn_VAT-OUT-21-S","attn_VAT-OUT-00-ROW","","","",""
"","","","","","","","attn_VAT-OUT-21-L","attn_VAT-OUT-00-ROW","","","",""
"","","","","","","","attn_VAT-IN-V81-06","attn_VAT-IN-V81-06-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V81-12","attn_VAT-IN-V81-12-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V81-21","attn_VAT-IN-V81-21-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-06-S","attn_VAT-IN-V82-06-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-06-G","attn_VAT-IN-V82-06-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-12-S","attn_VAT-IN-V82-12-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-12-G","attn_VAT-IN-V82-12-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-21-S","attn_VAT-IN-V82-21-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-21-G","attn_VAT-IN-V82-21-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V83-06","attn_VAT-IN-V83-06-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V83-12","attn_VAT-IN-V83-12-ROW-CC","","","",""
"","","","","","","","attn_VAT-IN-V83-21","attn_VAT-IN-V83-21-ROW-CC","","","",""
"","","","","","","","","","a7000","a7002","",""
"","","","","","","","","","a7010","a7012","",""
"","","","","","","","","","a7050","a7052","",""
"fiscal_position_template_4","5","Co-Contractant","","","","","attn_VAT-OUT-00-S","attn_VAT-OUT-00-CC","","","Cocontractant","Medecontractant"
"","","","","","","","attn_VAT-OUT-00-L","attn_VAT-OUT-00-CC","","","",""
"","","","","","","","attn_VAT-OUT-06-S","attn_VAT-OUT-00-CC","","","",""
"","","","","","","","attn_VAT-OUT-06-L","attn_VAT-OUT-00-CC","","","",""
"","","","","","","","attn_VAT-OUT-12-S","attn_VAT-OUT-00-CC","","","",""
"","","","","","","","attn_VAT-OUT-12-L","attn_VAT-OUT-00-CC","","","",""
"","","","","","","","attn_VAT-OUT-21-S","attn_VAT-OUT-00-CC","","","",""
"","","","","","","","attn_VAT-OUT-21-L","attn_VAT-OUT-00-CC","","","",""
"","","","","","","","attn_VAT-IN-V81-06","attn_VAT-IN-V81-06-CC","","","",""
"","","","","","","","attn_VAT-IN-V81-12","attn_VAT-IN-V81-12-CC","","","",""
"","","","","","","","attn_VAT-IN-V81-21","attn_VAT-IN-V81-21-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-06-S","attn_VAT-IN-V82-06-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-06-G","attn_VAT-IN-V82-06-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-12-S","attn_VAT-IN-V82-12-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-12-G","attn_VAT-IN-V82-12-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-21-S","attn_VAT-IN-V82-21-CC","","","",""
"","","","","","","","attn_VAT-IN-V82-21-G","attn_VAT-IN-V82-21-CC","","","",""
"","","","","","","","attn_VAT-IN-V83-06","attn_VAT-IN-V83-06-CC","","","",""
"","","","","","","","attn_VAT-IN-V83-12","attn_VAT-IN-V83-12-CC","","","",""
"","","","","","","","attn_VAT-IN-V83-21","attn_VAT-IN-V83-21-CC","","","",""

```

## File: data\template\account.group-be.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@fr","name@nl","name@de"
"be_group_1","1","","Equity, Provisions for Liabilities and Charges and Amounts Payable After More Than One Year","Fonds propres, provisions pour risques et charges, impôts différés et dettes à plus d'un an","Eigen vermogen, voorzieningen voor risico's en kosten, schulden op meer dan één jaar","Eigenkapital, Rückstellungen für Risiken und Aufwendungen sowie Verbindlichkeiten mit einer Restlaufzeit von mehr als einem Jahr"
"be_group_12","12","","Revaluation Surpluses","Plus-values de réévaluation","Herwaarderingsmeerwaarden","Neubewertungsrücklagen"
"be_group_120","120","","Revaluation Surpluses on Intangible Fixed Assets","Plus-values de réévaluation sur immobilisations incorporelles","Herwaarderingsmeerwaarden op immateriële vaste activa","Neubewertungsrücklagen auf immaterielle Anlagewerte"
"be_group_121","121","","Revaluation Surpluses on Tangible Fixed Assets","Plus-values de réévaluation sur immobilisations corporelles","Herwaarderingsmeerwaarden op materiële vaste activa","Neubewertungsrücklagen auf Sachanlagen"
"be_group_122","122","","Revaluation Surpluses on Financial Fixed Assets","Plus-values de réévaluation sur immobilisations financières","Herwaarderingsmeerwaarden op financiële vaste activa","Neubewertungsrücklagen auf Finanzanlagen"
"be_group_124","124","","Decrease in Amounts Written Down on Current Investments","Reprises de réductions de valeur sur placements de trésorerie","Terugnemingen van waardeverminderingen op geldbeleggingen","Rücknahmen von Wertminderungen auf Geldanlagen"
"be_group_132","132","","Untaxed Reserves","Réserves immunisées","Belastingvrije reserves","Steuerfreie Rücklagen"
"be_group_14","14","","Profit (Loss) Brought Forward (-)","Bénéfice (Perte) reporté(e) (-)","Overgedragen winst (verlies) (-)","Gewinnvortrag (Verlustvortrag) (-)"
"be_group_16","16","","Provisions and Deferred Taxes","Provisions et impôts différés","Voorzieningen en uitgestelde belastingen","Rückstellungen und Aufgeschobene Steuern"
"be_group_160","160","","Provisions for Pensions and Similar Obligations","Provisions pour pensions et obligations similaires","Voorzieningen voor pensioenen en soortgelijke verplichtingen","Rückstellungen für Pensionen und ähnliche Verpflichtungen"
"be_group_161","161","","Provisions for Taxation","Provisions pour charges fiscales","Voorzieningen voor belastingen","Rückstellungen für Steuern"
"be_group_162","162","","Provisions for Major Repairs and Maintenance","Provisions pour grosses réparations et gros entretien","Voorzieningen voor grote herstellings- en onderhoudswerken","Rückstellungen für Großreparaturen und Instandhaltungsarbeiten"
"be_group_163","163","","Provisions for Environmental Obligations","Provisions pour obligations environnementales","Voorzieningen voor milieuverplichtingen","Rückstellungen für Umweltschutzverpflichtungen"
"be_group_164","164","165","Provisions for Other Liabilities and Charges","Provisions pour autres risques et charges","Voorzieningen voor overige risico's en kosten","Rückstellungen für sonstige Risiken und Aufwendungen"
"be_group_168","168","","Deferred Taxes","Impôts différés","Uitgestelde belastingen","Aufgeschobene Steuern"
"be_group_17","17","","Amounts Payable After More Than One Year","Dettes à plus d'un an","Schulden op meer dan één jaar","Verbindlichkeiten mit einer Restlaufzeit von mehr als einem Jahr"
"be_group_170","170","","Subordinated Loans (> 1 Year)","Emprunts subordonnés (> 1 an)","Achtergestelde leningen (> 1 jaar)","Nachrangige Anleihen (> 1 Jahr)"
"be_group_171","171","","Unsubordinated Debentures (> 1 Year)","Emprunts obligataires non subordonnés (> 1 an)","Niet-achtergestelde obligatieleningen (> 1 jaar)","Nicht nachrangige Anleihen (> 1 Jahr)"
"be_group_172","172","","Leasing and Other Similar Obligations (> 1 Year)","Dettes de location-financement et dettes assimilées (> 1 an)","Leasingschulden en soortgelijke schulden (> 1 jaar)","Verbindlichkeiten aufgrund von Leasing- und ähnlichen Verträgen (> 1 Jahr)"
"be_group_173","173","","Credit Institutions (> 1 Year)","Etablissements de crédit (> 1 an)","Kredietinstellingen (> 1 jaar)","Kreditinstitute (> 1 Jahr)"
"be_group_174","174","","Other Loans (> 1 Year)","Autres emprunts (> 1 an)","Overige leningen (> 1 jaar)","Sonstige Anleihen (> 1 Jahr)"
"be_group_175","175","","Trade Debts (> 1 Year)","Dettes commerciales (> 1 an)","Handelsschulden (> 1 jaar)","Verbindlichkeiten aus Lieferungen und Leistungen (> 1 Jahr)"
"be_group_176","176","","Advance Payments on Contract in Progress (> 1 Year)","Acomptes sur commandes (> 1 an)","Vooruitbetalingen op bestellingen (> 1 jaar)","Anzahlungen auf Bestellungen (> 1 Jahr)"
"be_group_178","178","","Guarantees Received in Cash (> 1 Year)","Cautionnements reçus en numéraire (> 1 an)","Borgtochten ontvangen in contanten (> 1 jaar)","In Barmitteln erhaltene Kautionen (> 1 Jahr)"
"be_group_179","179","","Other Amounts Payable (> 1 Year)","Autres dettes (> 1 an)","Overige schulden (> 1 jaar)","Sonstige Verbindlichkeiten (> 1 Jahr)"
"be_group_2","2","","Formation Expenses, Fixed Assets and Amounts Receivable After More Than One Year","Frais d'établissement, actifs immobilisés et créances à plus d'un an","Oprichtingskosten, vaste activa en vorderingen op meer dan één jaar","Errichtungs- und Erweiterungskosten, Anlagevermögen, Forderungen mit einer Restlaufzeit von mehr als einem Jahr"
"be_group_20","20","","Formation Expenses","Frais d'établissement","Oprichtingskosten","Errichtungs- und Erweiterungsaufwendungen"
"be_group_200","200","","Formation Expenses","Frais d'établissement","Oprichtingskosten","Errichtungs- und Erweiterungsaufwendungen"
"be_group_201","201","","Loan Issue Expenses","Frais d'émission d'emprunts","Kosten bij uitgifte van leningen","Emissionskosten von Anleihen"
"be_group_202","202","","Other Formation Expenses","Autres frais d'établissement","Overige oprichtingskosten","Andere Errichtungs- und Erweiterungsaufwendungen"
"be_group_204","204","","Restructuring Costs","Frais de restructuration","Herstructureringskosten","Restrukturierungskosten"
"be_group_21","21","","Intangible Fixed Assets","Immobilisations incorporelles","Immateriële vaste activa","Immaterielle Anlagewerte"
"be_group_210","210","","Research and Development Costs","Frais de recherche et de développement","Kosten van onderzoek en ontwikkeling","Forschungs- und Entwicklungskosten"
"be_group_211","211","","Concessions, Patents, Licences, Know-How, Brands and Similar Rights","Concessions, brevets, licences, savoir-faire, marques et droits similaires","Concessies, octrooien, licenties, know-how, merken en soortgelijke rechten","Konzessionen, Patente, Lizenzen, Know-how, Warenzeichen und ähnliche Rechte"
"be_group_212","212","","Goodwill","Goodwill","Goodwill","Goodwill"
"be_group_213","213","","Advance Payments for Intangible Fixed Assets","Acomptes versés pour immobilisations incorporelles","Vooruitbetalingen voor immateriële vaste activa","Geleistete Anzahlungen für immaterielle Anlagewerte"
"be_group_22","22","","Land and Buildings","Terrains et constructions","Terreinen en gebouwen","Grundstücke und Bauten"
"be_group_220","220","","Land","Terrains","Terreinen","Grundstücke"
"be_group_221","221","","Buildings","Constructions","Gebouwen","Bauten"
"be_group_222","222","","Developed Land","Terrains bâtis","Bebouwde terreinen","Bebaute Grundstücke"
"be_group_223","223","","Other Rights to Immovable Property","Autres droits réels sur des immeubles","Overige zakelijke rechten op onroerende goederen","Sonstige dingliche Rechte an unbeweglichen Gütern"
"be_group_23","23","","Plant, Machinery and Equipment","Installations, machines et outillage","Installaties, machines en uitrusting","Anlagen, Maschinen und Betriebsausstattung"
"be_group_24","24","","Furniture and Vehicles","Mobilier et matériel roulant","Meubilair en rollend materieel","Geschäftsausstattung und Fuhrpark"
"be_group_25","25","","Fixed Assets in Leasing or Other Similar Rights","Immobilisations détenues en location-financement et droits similaires","Vaste activa in leasing of op grond van een soortgelijk recht","Aufgrund von Leasing und ähnliche Rechte gehaltene Anlagen"
"be_group_250","250","","Land and Buildings in Leasing or Other Similar Rights","Terrains et constructions détenues en location-financement et droits similaires","Terreinen en gebouwen in leasing of op grond van een soortgelijk recht","Aufgrund von Leasing und ähnliche Rechte gehaltene Grundstücke und Bauten"
"be_group_251","251","","Plant, Machinery and Equipment in Leasing or Other Similar Rights","Installations, machines et outillage détenues en location-financement et droits similaires","Installaties, machines en uitrusting in leasing of op grond van een soortgelijk recht","Aufgrund von Leasing und ähnliche Rechte gehaltene Anlagen, Maschinen und Betriebsausstattung"
"be_group_252","252","","Furniture and Vehicles in Leasing or Other Similar Rights","Mobilier et matériel roulant détenues en location-financement et droits similaires","Meubilair en rollend materieel in leasing of op grond van een soortgelijk recht","Aufgrund von Leasing und ähnliche Rechte gehaltene Geschäftsausstattung und Fuhrpark"
"be_group_26","26","","Other Tangible Fixed Assets","Autres immobilisations corporelles","Overige materiële vaste activa","Sonstige Sachanlagen"
"be_group_27","27","","Tangible Fixed Assets Under Construction and Advance Payments","Immobilisations corporelles en cours et acomptes versés","Vaste activa in aanbouw en vooruitbetalingen","Sachanlagen im Bau und geleistete Anzahlungen"
"be_group_28","28","","Financial Fixed Assets","Immobilisations financières","Financiële vaste activa","Finanzanlagen"
"be_group_280","280","","Participating Interests in Affiliated Enterprises","Participations dans les sociétés liées","Deelnemingen in verbonden vennootschappen","Beteiligungen an verbundenen Gesellschaften"
"be_group_281","281","","Amounts Receivable From Affiliated Enterprises","Créances sur des entreprises liées","Vorderingen op verbonden ondernemingen","Forderungen an verbundenen Unternehmen"
"be_group_282","282","","Participating Interests in Other Enterprises Linked by Participating Interests","Participations dans les entreprises avec lesquelles il existe un lien de participation","Deelnemingen in ondernemingen waarmee een deelnemingsverhouding bestaat","Beteiligungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht"
"be_group_283","283","","Amounts Receivable From Other Enterprises Linked by Participating Interests","Créances sur les entreprises avec lesquelles il existe un lien de participation","Vorderingen op ondernemingen waarmee een deelnemingsverhouding bestaat","Forderungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht"
"be_group_284","284","","Other Shares","Autres actions et parts","Andere aandelen","Sonstige Aktien oder Anteile"
"be_group_285","285","","Other Amounts Receivable","Autres créances","Andere vorderingen","Sonstige Forderungen"
"be_group_288","288","","Cash Guarantees Paid","Cautionnements versés en numéraire","Borgtochten betaald in contanten","Gezahlte Kautionen"
"be_group_29","29","","Amounts Receivable After More Than One Year","Créances à plus d'un an","Vorderingen op meer dan één jaar","Forderungen mit einer Restlaufzeit von mehr als einem Jahr"
"be_group_290","290","","Trade Debtors (> 1 Year)","Créances commerciales (> 1 an)","Handelsvorderingen (> 1 jaar)","Forderungen aus Lieferungen und Leistungen (> 1 Jahr)"
"be_group_291","291","","Other Amounts Receivable (> 1 Year)","Autres créances (> 1 an)","Overige vorderingen (> 1 jaar)","Sonstige Forderungen (> 1 Jahr)"
"be_group_3","3","","Stocks and Contracts in Progress","Stocks et commandes en cours d'exécution","Voorraden en bestellingen in uitvoering","Vorräte und in Ausführung befindliche Bestellungen"
"be_group_30","30","","Raw Materials","Matières premières","Grondstoffen","Rohstoffe"
"be_group_300","300","","Raw Materials - Acquisition Value","Matières premières - Valeur d'acquisition","Grondstoffen - Aanschaffingswaarde","Rohstoffe - Anschaffungswert"
"be_group_309","309","","Raw Materials - Amounts Written Down Recorded (-)","Matières premières - Réductions de valeur actées (-)","Grondstoffen - Geboekte waardeverminderingen (-)","Rohstoffe - Gebuchte Wertminderungen (-)"
"be_group_31","31","","Consumables","Fournitures","Hulpstoffen","Hilfs- und Betriebsstoffe"
"be_group_310","310","","Consumables - Acquisition Value","Fournitures - Valeur d'acquisition","Hulpstoffen - Aanschaffingswaarde","Hilfs- und Betriebsstoffe - Anschaffungswert"
"be_group_319","319","","Consumables - Amounts Written Down Recorded (-)","Fournitures - Réductions de valeur actées (-)","Hulpstoffen - Geboekte waardeverminderingen (-)","Hilfs- und Betriebsstoffe - Gebuchte Wertminderungen (-)"
"be_group_32","32","","Work in Progress","En-cours de fabrication","Goederen in bewerking","Unfertige Erzeugnisse"
"be_group_320","320","","Work in Progress - Acquisition Value","En-cours de fabrication - Valeur d'acquisition","Goederen in bewerking - Aanschaffingswaarde","Unfertige Erzeugnisse - Anschaffungswert"
"be_group_329","329","","Work in Progress - Amounts Written Down Recorded (-)","En-cours de fabrication - Réductions de valeur actées (-)","Goederen in bewerking - Geboekte waardeverminderingen (-)","Unfertige Erzeugnisse - Gebuchte Wertminderungen (-)"
"be_group_33","33","","Finished Goods","Produits finis","Gereed product","Fertige Erzeugnisse"
"be_group_330","330","","Finished Goods - Acquisition Value","Produits finis - Valeur d'acquisition","Gereed product - Aanschaffingswaarde","Fertige Erzeugnisse - Anschaffungswert"
"be_group_339","339","","Finished Goods - Amounts Written Down Recorded (-)","Produits finis - Réductions de valeur actées (-)","Gereed product - Geboekte waardeverminderingen (-)","Fertige Erzeugnisse - Gebuchte Wertminderungen (-)"
"be_group_34","34","","Goods Purchased for Resale","Marchandises","Handelsgoederen","Waren"
"be_group_340","340","","Goods Purchased for Resale - Acquisition Value","Marchandises - Valeur d'acquisition","Handelsgoederen - Aanschaffingswaarde","Waren - Anschaffungswert"
"be_group_349","349","","Goods Purchased for Resale - Amounts Written Down Recorded (-)","Marchandises - Réductions de valeur actées (-)","Handelsgoederen - Geboekte waardeverminderingen (-)","Waren - Gebuchte Wertminderungen (-)"
"be_group_35","35","","Immovable Property Intended for Sale","Immeubles destinés à la vente","Onroerende goederen bestemd voor verkoop","Zum Verkauf bestimmte unbewegliche Gegenstände"
"be_group_350","350","","Immovable Property Intended for Sale - Acquisition Value","Immeubles destinés à la vente - Valeur d'acquisition","Onroerende goederen bestemd voor verkoop - Aanschaffingswaarde","Zum Verkauf bestimmte unbewegliche Gegenstände - Anschaffungswert"
"be_group_359","359","","Immovable Property Intended for Sale - Amounts Written Down Recorded (-)","Immeubles destinés à la vente - Réductions de valeur actées (-)","Onroerende goederen bestemd voor verkoop - Geboekte waardeverminderingen (-)","Zum Verkauf bestimmte unbewegliche Gegenstände - Gebuchte Wertminderungen (-)"
"be_group_36","36","","Advance Payments on Purchases for Stocks","Acomptes versés sur achats pour stocks","Vooruitbetalingen op voorraadinkopen","Geleistete Anzahlungen auf Vorräte"
"be_group_360","360","","Advance Payments on Purchases for Stocks - Advance Payments","Acomptes versés sur achats pour stocks - Acomptes versés","Vooruitbetalingen op voorraadinkopen - Vooruitbetalingen","Geleistete Anzahlungen auf Vorräte - Geleistete Anzahlungen"
"be_group_369","369","","Advance Payments on Purchases for Stocks - Amounts Written Down Recorded (-)","Acomptes versés sur achats pour stocks - Réductions de valeur actées (-)","Vooruitbetalingen op voorraadinkopen - Geboekte waardeverminderingen (-)","Geleistete Anzahlungen auf Vorräte - Gebuchte Wertminderungen (-)"
"be_group_37","37","","Contracts in Progress","Commandes en cours d'exécution","Bestellingen in uitvoering","In Ausführung befindliche Bestellungen"
"be_group_370","370","","Contracts in Progress - Acquisition Value","Commandes en cours d'exécution - Valeur d'acquisition","Bestellingen in uitvoering - Aanschaffingswaarde","In Ausführung befindliche Bestellungen - Anschaffungswert"
"be_group_371","371","","Contracts in Progress - Profit Recognized","Commandes en cours d'exécution - Bénéfice pris en compte","Bestellingen in uitvoering - Toegerekende winst","In Ausführung befindliche Bestellungen - Aktivierte Gewinnanteile"
"be_group_379","379","","Contracts in Progress - Amounts Written Down Recorded (-)","Commandes en cours d'exécution - Réductions de valeur actées (-)","Bestellingen in uitvoering - Geboekte waardeverminderingen (-)","In Ausführung befindliche Bestellungen - Gebuchte Wertminderungen (-)"
"be_group_4","4","","Amounts Receivable Within One Year","Créances à un an au plus","Vorderingen op ten hoogste één jaar","Forderungen mit einer Restlaufzeit bis zu einem Jahr"
"be_group_40","40","","Trade Debtors","Créances commerciales","Handelsvorderingen","Forderungen aus Lieferungen und Leistungen"
"be_group_400","400","","Customers","Clients","Handelsdebiteuren","Kunden"
"be_group_401","401","","Trade Debtors - Bills Receivable","Créances commerciales - Effets à recevoir","Handelsvorderingen - Te innen wissels","Forderungen aus Lieferungen und Leistungen - Besitzwechsel"
"be_group_404","404","","Trade Debtors - Income Receivable","Créances commerciales - Produits à recevoir","Handelsvorderingen - Te innen opbrengsten","Forderungen aus Lieferungen und Leistungen - Zu erhaltene Erträge"
"be_group_406","406","","Trade Debtors - Advance Payments","Créances commerciales - Acomptes versés","Handelsvorderingen - Vooruitbetalingen","Forderungen aus Lieferungen und Leistungen - Geleistete Anzahlungen"
"be_group_407","407","","Trade Debtors - Doubtful Amounts Receivable","Créances commerciales - Créances douteuses","Handelsvorderingen - Dubieuze debiteuren","Forderungen aus Lieferungen und Leistungen - Zweifelhalte Forderungen"
"be_group_409","409","","Trade Debtors - Amounts Written Down Recorded (-)","Créances commerciales - Réductions de valeur actées (-)","Handelsvorderingen - Geboekte waardeverminderingen (-)","Forderungen aus Lieferungen und Leistungen - Gebuchte Wertminderungen (-)"
"be_group_41","41","","Other Amounts Receivable","Autres créances","Overige vorderingen","Sonstige Forderungen"
"be_group_411","411","","VAT Recoverable","TVA à récupérer","Terug te vorderen btw","Zurückzuerhaltende Mehrwertsteuer"
"be_group_412","412","","Taxes and Withholding Taxed to Be Recovered","Impôts et précomptes à récupérer","Terug te vorderen belastingen en voorheffingen","Zurückzuerstattende Steuern und Steuervorhabzüge"
"be_group_414","414","","Income Receivable","Produits à recevoir","Te innen opbrengsten","Zu erhaltende Erträge"
"be_group_416","416","","Sundry Amounts Receivable","Créances diverses","Diverse vorderingen","Übrige Forderungen"
"be_group_417","417","","Sundry Amounts Receivable - Doubtful Amounts Receivable","Créances diverses - Créances douteuses","Overige vorderingen - Dubieuze debiteuren","Übrige Forderungen - Zweifelhafte Forderungen"
"be_group_418","418","","Sundry Amounts Receivable - Cash Guarantees Paid","Créances diverses - Cautionnements versés en numéraire","Overige vorderingen - Borgtochten betaald in contanten","Übrige Forderungen - Gezahlte Kautionen"
"be_group_419","419","","Sundry Amounts Receivable - Amounts Written Down Recorded (-)","Créances diverses - Réductions de valeur actées (-)","Overige vorderingen - Geboekte waardeverminderingen (-)","Übrige Forderungen - Gebuchte Wertminderungen (-)"
"be_group_42","42","","Current Portion of Amounts Payable After More Than One Year Falling Due Within One Year","Dettes à plus d'un an échéant dans l'année","Schulden op meer dan één jaar die binnen het jaar vervallen","Innerhalb eines Jahres fällig werdende Verbindlichkeiten mit einer ursprünglichen Laufzeit von mehr als einem Jahr"
"be_group_420","420","","Subordinated Loans (> 1 Year Falling Due < 1 Year)","Emprunts subordonnés (> 1 an écheant < 1 an)","Achtergestelde leningen (> 1 jaar vervallend < 1 jaar)","Nachrangige Anleihen (> 1 Jahr fällig < 1 Jahr)"
"be_group_421","421","","Unsubordinated Debentures (> 1 Year Falling Due < 1 Year)","Emprunts obligataires non subordonnés (> 1 an écheant < 1 an)","Niet-achtergestelde obligatieleningen (> 1 jaar vervallend < 1 jaar)","Nicht nachrangige Anleihen (> 1 Jahr fällig < 1 Jahr)"
"be_group_422","422","","Leasing and Other Similar Obligations (> 1 Year Falling Due < 1 Year)","Dettes de location-financement et dettes assimilées (> 1 an écheant < 1 an)","Leasingschulden en soortgelijke schulden (> 1 jaar vervallend < 1 jaar)","Verbindlichkeiten aufgrund von Leasing- und ähnlichen Verträgen (> 1 Jahr fällig < 1 Jahr)"
"be_group_423","423","","Credit Institutions (> 1 Year Falling Due < 1 Year)","Etablissements de crédit (> 1 an écheant < 1 an)","Kredietinstellingen (> 1 jaar vervallend < 1 jaar)","Kreditinstitute (> 1 Jahr fällig < 1 Jahr)"
"be_group_424","424","","Other Loans (> 1 Year Falling Due < 1 Year)","Autres emprunts (> 1 an écheant < 1 an)","Overige leningen (> 1 jaar vervallend < 1 jaar)","Sonstige Anleihen (> 1 Jahr fällig < 1 Jahr)"
"be_group_425","425","","Trade Debts (> 1 Year Falling Due < 1 Year)","Dettes commerciales (> 1 an écheant < 1 an)","Handelsschulden (> 1 jaar vervallend < 1 jaar)","Verbindlichkeiten aus Lieferungen und Leistungen (> 1 Jahr fällig < 1 Jahr)"
"be_group_426","426","","Advance Payments on Contract in Progress (> 1 Year Falling Due < 1 Year)","Acomptes sur commandes (> 1 an écheant < 1 an)","Vooruitbetalingen op bestellingen (> 1 jaar vervallend < 1 jaar)","Anzahlungen auf Bestellungen (> 1 Jahr fällig < 1 Jahr)"
"be_group_428","428","","Guarantees Received in Cash (> 1 Year Falling Due < 1 Year)","Cautionnements reçus en numéraire (> 1 an écheant < 1 an)","Borgtochten ontvangen in contanten (> 1 jaar vervallend < 1 jaar)","In Barmitteln erhaltene Kautionen (> 1 Jahr fällig < 1 Jahr)"
"be_group_429","429","","Other Amounts Payable (> 1 Year Falling Due < 1 Year)","Autres dettes (> 1 an écheant < 1 an)","Overige schulden (> 1 jaar vervallend < 1 jaar)","Sonstige Verbindlichkeiten (> 1 Jahr fällig < 1 Jahr)"
"be_group_43","43","","Financial Debts","Dettes financières","Financiële schulden","Finanzverbindlichkeiten"
"be_group_430","430","","Credit Institutions - Fixed Term Loans","Etablissements de crédit - Emprunts en compte à terme fixe","Kredietinstellingen - Leningen op rekening met vaste termijn","Kreditinstitute - Kontokredite mit fester Laufzeit"
"be_group_431","431","","Credit Institutions - Promissory Notes","Etablissements de crédit - Promesses","Kredietinstellingen - Promessen","Kreditinstitute - Solawechsel"
"be_group_432","432","","Credit Institutions - Bank Acceptances","Etablissements de crédit - Crédits d'acceptation","Kredietinstellingen - Acceptkredieten","Kreditinstitute - Akzeptkredite"
"be_group_433","433","","Credit Institutions - Current Account Payable","Etablissements de crédit - Dettes en compte courant","Kredietinstellingen - Schulden in rekening-courant","Kreditinstitute - Kontokorrentverbindlichkeiten"
"be_group_439","439","","Other Loans","Autres emprunts","Overige leningen","Sonstige Anleihen"
"be_group_44","44","","Trade Debts","Dettes commerciales","Handelsschulden","Verbindlichkeiten aus Lieferungen und Leistungen"
"be_group_440","440","","Suppliers","Fournisseurs","Leveranciers","Lieferanten"
"be_group_441","441","","Bills of Exchange Payable","Effets à payer","Te betalen wissels","Verbindlichkeiten aus Wechseln"
"be_group_444","444","","Invoices to Be Received","Factures à recevoir","Te ontvangen facturen","Zu erhaltende Rechnungen"
"be_group_45","45","","Taxes, Remuneration and Social Security","Dettes fiscales, salariales et sociales","Schulden met betrekking tot belastingen, bezoldigingen en sociale lasten","Verbindlichkeiten aufgrund von Steuern, Arbeitsentgelten und Soziallasten"
"be_group_450","450","","Estimated Taxes Payable","Dettes fiscales estimées","Geraamde belastingschulden","Geschätzte Steuerschulden"
"be_group_451","451","","VAT Payable","TVA à payer","Te betalen btw","Zu zahlende Mehrwertsteuer"
"be_group_452","452","","Taxes Payable","Impôts et taxes à payer","Te betalen belastingen en taksen","Zu zahlende Steuern und Abgaben"
"be_group_453","453","","Taxes Withheld","Précomptes retenus","Ingehouden voorheffingen","Einbehaltene Steuervorhabzüge"
"be_group_454","454","","National Social Security Office","Office National de Sécurité Sociale","Rijksdienst voor Sociale Zekerheid","Landesamt für Soziale Sicherheit"
"be_group_455","455","","Remuneration","Rémunérations","Bezoldigingen","Arbeitsentgelte"
"be_group_456","456","","Holiday Pay","Pécules de vacances","Vakantiegeld","Urlaubsgeld"
"be_group_459","459","","Other Social Obligations","Autres dettes sociales","Andere sociale schulden","Sonstige Soziallasten"
"be_group_46","46","","Advances on Contracts in Progress","Acomptes sur commandes","Vooruitbetalingen op bestellingen","Anzahlungen auf Bestellungen"
"be_group_48","48","","Miscellaneous Amounts Payable","Dettes diverses","Diverse schulden","Sonstige Verbindlichkeiten"
"be_group_480","480","","Debentures and Matured Coupons","Obligations et coupons échus","Vervallen obligaties en coupons","Fällige Schuldverschreibungen und Kupons"
"be_group_488","488","","Guarantees Received in Cash","Cautionnements reçus en numéraire","Borgtochten ontvangen in contanten","In Geldmitteln erhaltene Kautionen"
"be_group_489","489","","Sundry Amounts Payable","Autres dettes","Andere diverse schulden","Übrige Verbindlichkeiten"
"be_group_49","49","","Accruals and Deferred Income","Comptes de régularisation","Overlopende rekeningen","Rechnungsabgrenzungsposten"
"be_group_490","490","","Deferred Charges","Charges à reporter","Over te dragen kosten","Vorzutragende Aufwendungen"
"be_group_491","491","","Accrued Income","Produits acquis","Verkregen opbrengsten","Erworbene Erträge"
"be_group_492","492","","Accrued Charges","Charges à imputer","Toe te rekenen kosten","Anzurechnende Aufwendungen"
"be_group_493","493","","Deferred Income","Produits à reporter","Over te dragen opbrengsten","Vorzutragende Erträge"
"be_group_499","499","","Suspense Accounts","Comptes d'attente","Wachtrekeningen","Wartekonten"
"be_group_5","5","","Current Investments and Cash at Bank and in Hand","Placements de trésorerie et valeurs disponibles","Geldbeleggingen en liquide middelen","Geldanlagen und Flüssige Mittel"
"be_group_52","52","","Fixed-Income Securities","Titres à revenu fixe","Vastrentende effecten","Festverzinsliche Wertpapiere"
"be_group_520","520","","Fixed-Income Securities - Acquisition Value","Titres à revenu fixe - Valeur d'acquisition","Vastrentende effecten - Aanschaffingswaarde","Festverzinsliche Wertpapiere - Anschaffungswert"
"be_group_529","529","","Fixed-Income Securities - Amounts Written Down Recorded (-)","Titres à revenu fixe - Réductions de valeur actées (-)","Vastrentende effecten - Geboekte waardeverminderingen (-)","Festverzinsliche Wertpapiere - Gebuchte Wertminderungen (-)"
"be_group_53","53","","Fixed Term Deposits","Dépôts à terme","Termijnrekeningen","Terminkonten"
"be_group_530","530","","Fixed Term Deposit Over One Year","Dépôts à terme de plus d'un an","Termijnrekeningen op meer dan één jaar","Terminkonten mit einer Laufzeit von mehr als einem Jahr"
"be_group_531","531","","Fixed Term Deposit Between One Month and One Year","Dépôts à terme de plus d'un mois et à un an au plus","Termijnrekeningen op meer dan één maand en hoogstens één jaar","Terminkonten mit einer Laufzeit von mehr als einem Monat und bis zu einem Jahr"
"be_group_532","532","","Fixed Term Deposit Up to One Month","Dépôts à terme d'un mois au plus","Termijnrekeningen op hoogstens één maand","Terminkonten mit einer Laufzeit bis zu einem Monat"
"be_group_539","539","","Fixed Term Deposits - Amounts Written Down Recorded (-)","Dépôts à terme - Réductions de valeur actées (-)","Termijnrekeningen - Geboekte waardeverminderingen (-)","Terminkonten - Gebuchte Wertminderungen (-)"
"be_group_54","54","","Receivable Securities Fallen Due","Valeurs échues à l'encaissement","Te incasseren vervallen waarden","Zum Inkasso fällige Werte"
"be_group_55","55","","Cash at Bank - Credit Institutions","Etablissements de crédit","Kredietinstellingen","Kreditinstitute"
"be_group_550","550","559","Cash at Bank - Credit Institutions","Etablissements de crédit","Kredietinstellingen","Kreditinstitute"
"be_group_57","57","","Cash in Hand","Caisses","Kassen","Kassenbestand"
"be_group_570","570","577","Cash in Hand - Cash","Caisses - Espèces","Kassen - Contanten","Kassen - Bargeld"
"be_group_578","578","","Cash in Hand - Stamps","Caisses - Timbres","Kassen - Zegels","Kassen - Wertmarken"
"be_group_58","58","","Internal Transfers of Funds","Virements internes","Interne overboekingen","Interne Geldtransferkonten"
"be_group_6","6","","Expenses","Charges","Bedrijfskosten","Aufwendungen"
"be_group_60","60","","Goods for Resale, Raw Materials and Consumables","Approvisionnements et marchandises","Handelsgoederen, grond- en hulpstoffen","Waren, Roh-, Hilfs- und Betriebsstoffe"
"be_group_600","600","","Purchases of Raw Materials","Achats de matières premières","Aankopen van grondstoffen","Käufe von Rohstoffen"
"be_group_601","601","","Purchases of Consumables","Achats de fournitures","Aankopen van hulpstoffen","Käufe von Hilfs- und Betriebsstoffen"
"be_group_602","602","","Purchases of Services, Works and Studies","Achats de services, travaux et études","Aankopen van diensten, werk en studies","Käufe von Dienstleistungen, Arbeiten und Studien"
"be_group_603","603","","Subcontracting","Sous-traitances générales","Algemene onderaannemingen","Einsätze von Unterlieferanten"
"be_group_604","604","","Purchases of Goods for Resale","Achats de marchandises","Aankopen van handelsgoederen","Käufe von Handelswaren"
"be_group_605","605","","Purchases of Immovable Property for Resale","Achats d'immeubles destinés à la vente","Aankopen van onroerende goederen bestemd voor verkoop","Kauf von für den Verkauf bestimmten unbeweglichen Gegenständen"
"be_group_608","608","","Discounts, Allowance and Rebates Received (-)","Remises, ristournes et rabais (-)","Ontvangen kortingen, ristorno's en rabatten (-)","Erhaltene Preisnachlässe, Rückvergütungen und Rabatte (-)"
"be_group_609","609","","Decrease (Increase) in Stocks","Réduction (Augmentation) des stocks","Afname (Toename) van de voorraad","Abnahme (Zunahme) des Bestandes"
"be_group_61","61","","Services and Other Goods","Services et biens divers","Diensten en diverse goederen","Übrige Lieferungen und Leistungen"
"be_group_617","617","","Costs of Hired Temporary Staff and Persons Placed at the Enterprise's Disposal","Frais de personnel intérimaire et de personnes mises à la disposition de la société","Kosten met betrekking tot uitzendkrachten en personen ter beschikking gesteld van de vennootschap","Auf Zeitarbeitpersonal und der Gesellschaft zur Verfügung gestellte Personen bezüglichen Aufwand"
"be_group_618","618","","Remuneration, Premiums for Extra Statutory Insurance, Pensions of Directors, Managers or Active Partners Which Are Not Allowed Following the Contract","Rémunérations, primes pour assurances extralégales, pensions de retraite et de survie des administrateurs, gérants et associés actifs qui ne sont pas attribuées en vertu d'un contrat de travail","Bezoldigingen, premies voor buitenwettelijke verzekeringen, ouderdoms- en overlevingspensioenen van bestuurders, zaakvoerders en werkende vennoten, die niet worden toegekend uit hoofde van een arbeidsovereenkomst","Arbeitsentgelte, Prämien für außergesetzliche Versicherungen, Ruhestands- und Hinterbliebenenpensionen der Verwalter, Geschäftsführer oder aktiven Gesellschafter, die nicht aufgrund eines Arbeitsvertrags zuerkannt werden"
"be_group_62","62","","Remuneration, Social Security Costs and Pensions","Rémunérations, charges sociales et pensions","Bezoldigingen, sociale lasten en pensioenen","Arbeitsentgelte, Soziallasten und Pensionen"
"be_group_620","620","","Remuneration and Direct Social Benefits","Rémunérations et avantages sociaux directs","Bezoldigingen en rechtstreekse sociale voordelen","Arbeitsentgelte und direkte soziale Vorteile"
"be_group_621","621","","Employers' Contribution for Social Security","Cotisations patronales d'assurances sociales","Werkgeversbijdragen voor sociale verzekeringen","Arbeitgeberbeiträge zur Sozialversicherung"
"be_group_622","622","","Employers' Premiums for Extra Statutory Insurance","Primes patronales pour assurances extralégales","Werkgeverspremies voor bovenwettelijke verzekeringen","Arbeitgeberprämien für außergesetzliche Versicherungen"
"be_group_623","623","","Other Personnel Costs","Autres frais de personnel","Andere personeelskosten","Sonstige Personalaufwendungen"
"be_group_624","624","","Retirement and Survivors' Pensions","Pensions de retraite et de survie","Ouderdoms- en overlevingspensioenen","Ruhestands- und Hinterbliebenenpensionen"
"be_group_63","63","","Depreciation, Amounts Written Off and Provisions for Risks and Charges","Amortissements, réductions de valeur et provisions pour risques et charges","Afschrijvingen, waardeverminderingen en voorzieningen voor risico's en kosten","Abschreibungen, Wertminderungen sowie Rückstellungen für Risiken und Aufwendungen"
"be_group_630","630","","Depreciation, Amounts Written Off and Provisions for Risks","Amortissements, réductions de valeur et provisions pour risques","Afschrijvingen, waardeverminderingen en voorzieningen voor risico's","Abschreibungen, Wertminderungen sowie Rückstellungen für Risiken"
"be_group_631","631","","Amounts Written Off Stocks","Réductions de valeur sur stocks","Waardeverminderingen op voorraden","Wertminderungen von Vorräten"
"be_group_632","632","","Amounts Written Off Contracts in Progress","Réductions de valeur sur commandes en cours d'exécution","Waardeverminderingen op bestellingen in uitvoering","Wertminderungen von in Ausführung befindlichen Bestellungen"
"be_group_633","633","","Amounts Written Off Trade Debtors (> 1 Year)","Réductions de valeur sur créances commerciales (> 1 an)","Waardeverminderingen op handelsvorderingen (> 1 jaar)","Wertminderungen von Forderungen aus Lieferungen und Leistungen (> 1 Jahr)"
"be_group_634","634","","Amounts Written Off Trade Debtors","Réductions de valeur sur créances commerciales","Waardeverminderingen op handelsvorderingen","Wertminderungen von Forderungen aus Lieferungen und Leistungen"
"be_group_635","635","","Provisions for Pensions and Similar Obligations","Provisions pour pensions et obligations similaires","Voorzieningen voor pensioenen en soortgelijke verplichtingen","Rückstellungen für Pensionen und ähnliche Verpflichtungen"
"be_group_636","636","","Provisions for Major Repairs and Maintenance","Provisions pour grosses réparations et gros entretien","Voorzieningen voor grote herstellings- en onderhoudswerken","Rückstellungen für Großreparaturen und große Instandhaltungsarbeiten"
"be_group_637","637","","Provisions for Environmental Obligations","Provisions pour obligations environnementales","Voorzieningen voor milieuverplichtingen","Rückstellungen für Umweltschutzverpflichtungen"
"be_group_64","64","","Other Operating Charges","Autres charges d'exploitation","Andere bedrijfskosten","Sonstige betriebliche Aufwendungen"
"be_group_640","640","","Taxes Related to Operation","Charges fiscales d'exploitation","Bedrijfsbelastingen","Betriebliche Steuern"
"be_group_641","641","","Loss on Ordinary Disposal of Tangible Fixed Assets","Moins-values sur réalisations courantes d'immobilisations corporelles","Minderwaarden bij de courante realisatie van materiële vaste activa","Verluste aus dem normalen Abgang von Sachanlagen"
"be_group_642","642","","Loss on Ordinary Disposal of Trade Debtors","Moins-values sur réalisations courantes de créances commerciales","Minderwaarden bij de courante realisatie van handelsvorderingen","Minderwerte bei Realisierung von Forderungen aus Lieferungen und Leistungen"
"be_group_649","649","","Operating Charges Carried to Assets as Restructuring Costs (-)","Charges d'exploitation portées à l'actif au titre de frais de restructuration (-)","Als herstructureringskosten geactiveerde bedrijfskosten (-)","Auf der Aktivseite als Restrukturierungskosten ausgewiesene betriebliche Aufwendungen (-)"
"be_group_65","65","","Financial Charges","Charges financières","Financiële kosten","Finanzaufwendungen"
"be_group_650","650","","Debt Charges","Charges des dettes","Kosten van schulden","Aufwendungen für Verbindlichkeiten"
"be_group_651","651","","Amounts Written Off Current Assets","Réductions de valeur sur actifs circulants","Waardeverminderingen op vlottende activa","Wertminderungen von Gegenständen des Umlaufvermögens"
"be_group_652","652","","Capital Losses on Disposal of Current Assets","Moins-values sur réalisation d'actifs circulants ","Minderwaarden bij de realisatie van vlottende activa","Verluste aus dem Abgang von Gegenständen des Umlaufvermögens"
"be_group_653","653","","Discount Costs on Amounts Receivable","Charges d'escompte de créances","Discontokosten op vorderingen","Skontoaufwands auf Forderungen"
"be_group_654","654","","Exchange Differences","Différences de change","Wisselresultaten","Wechselkursdifferenzen"
"be_group_655","655","","Foreign Currency Translation Differences","Ecarts de conversion des devises","Resultaten uit de omrekening van vreemde valuta","Umrechnungsdifferenzen von Fremdwährungen"
"be_group_656","656","","Provisions of a Financial Nature","Provisions à caractère financier","Voorzieningen met financieel karakter","Rückstellungen mit finanziellem Charakter"
"be_group_657","657","658","Miscellaneous Financial Charges","Charges financières diverses","Diverse financiële kosten","Übrige Finanzaufwendungen"
"be_group_659","659","","Financial Charges Carried to Assets as Restructuring Costs (-)","Charges financières portées à l'actif au titre de frais de restructuration (-)","Als herstructureringskosten geactiveerde financiële kosten (-)","Finanzaufwendungen als Restrukturierungskosten ausgewiesene außerordentliche Aufwendungen (-)"
"be_group_66","66","","Non-Recurring Operating or Financial Charges","Charges d'exploitation et charges financières non récurrentes","Niet-recurrente bedrijfs- of financiële kosten","Nicht wiederkehrende betriebliche- oder Finanzaufwendungen"
"be_group_660","660","","Non-Recurring Depreciation of and Amounts Written Off","Amortissements et réductions de valeur non récurrents","Niet-recurrente afschrijvingen en waardeverminderingen","Nicht wiederkehrende Abschreibungen und Wertminderungen"
"be_group_661","661","","Amounts Written Off Financial Fixed Assets","Réductions de valeur sur immobilisations financières","Waardeverminderingen op financiële vaste activa","Wertminderungen auf Finanzanlagen"
"be_group_662","662","","Provisions for Non-Recurring Liabilities and Charges","Provisions pour risques et charges non récurrents","Voorzieningen voor niet-recurrente risico's en kosten","Rückstellungen für nicht wiederkehrende Risiken und Aufwendungen"
"be_group_663","663","","Capital Losses on Disposal of Fixed Assets","Moins-values sur réalisation d'actifs immobilisés","Minderwaarden bij de realisatie van vaste activa","Minderwerte aus dem Abgang von Gegenständen des Anlagevermögens"
"be_group_664","664","667","Other Non-Recurring Operating Charges","Autres charges d'exploitation non récurrentes","Andere niet-recurrente bedrijfskosten","Sonstige nicht wiederkehrende betriebliche Aufwendungen"
"be_group_668","668","","Other Non-Recurring Financial Charges","Autres charges financières non récurrentes","Andere niet-recurrente financiële kosten","Sonstige nicht wiederkehrende Finanzaufwendungen"
"be_group_669","669","","Non-Recurring Charges Carried to Assets as Restructuring Costs (-)","Charges non récurrentes portées à l'actif au titre de frais de restructuration (-)","Als herstructureringskosten geactiveerde niet-recurrente kosten (-)","Als Restrukturierungskosten ausgewiesene nicht wiederkehrende Aufwendungen (-)"
"be_group_67","67","","Income Taxes","Impôts sur le résultat","Belastingen op het resultaat","Steuern auf das Ergebnis"
"be_group_670","670","","Belgian Income Taxes on the Result of the Current Period","Impôts belges sur le résultat de l'exercice","Belgische belastingen op het resultaat van het boekjaar","Belgische Steuern auf das Ergebnis des Geschäftsjahres"
"be_group_671","671","","Belgian Income Taxes on the Result of Prior Periods","Impôts belges sur le résultat d'exercices antérieurs","Belgische belastingen op het resultaat van vorige boekjaren","Belgische Steuern auf das Ergebnis vorhergehender Geschäftsjahre"
"be_group_672","672","","Foreign Income Taxes on the Result of the Current Period","Impôts étrangers sur le résultat de l'exercice","Buitenlandse belastingen op het resultaat van het boekjaar","Ausländische Steuern auf das Ergebnis des Geschäftsjahres"
"be_group_673","673","","Foreign Income Taxes on the Result of Prior Periods","Impôts étrangers sur le résultat d'exercices antérieurs","Buitenlandse belastingen op het resultaat van vorige boekjaren","Ausländische Steuern auf das Ergebnis vorhergehender Geschäftsjahre"
"be_group_68","68","","Transfer to Deferred Taxes and Untaxed Reserves","Transfert aux impôts différés et aux réserves immunisées","Overboeking naar de uitgestelde belastingen en naar de belastingvrije reserves","Zuführung zu aufgeschobenen Steuern und Einstellung in die steuerbegünstigten Rücklagen"
"be_group_680","680","","Transfer to Deferred Taxes","Transfert aux impôts différés","Overboeking naar de uitgestelde belastingen","Zuführung zu aufgeschobenen Steuern"
"be_group_689","689","","Transfer to Untaxed Reserves","Transfert aux réserves immunisées","Overboeking naar de belastingvrije reserves","Einstellung in die steuerfreien Rücklagen"
"be_group_69","69","","Appropriation Account","Affectations et prélèvements","Resultaatverwerking","Ergebnisverwendung"
"be_group_690","690","","Loss of the Preceding Period Brought Forward","Perte reportée de l'exercice précédent","Overgedragen verlies van het vorige boekjaar","Verlusvortrag aus dem Vorjahr"
"be_group_7","7","","Income","Produits","Opbrengsten","Erträge"
"be_group_70","70","","Turnover","Chiffre d'affaires","Omzet","Umsatzerlöse"
"be_group_700","700","707","Sales and Services Rendered","Ventes et prestations de services","Verkopen en dienstprestaties","Verkäufe und Dienstleistungen"
"be_group_708","708","","Discounts, Allowances and Rebates Allowed (-)","Remises, ristournes et rabais accordés (-)","Toegekende kortingen, ristorno's en rabatten (-)","Preisnachlässe, Rückvergütungen und Rabatte (-)"
"be_group_71","71","","Increase (Decrease) in Stocks of Finished Goods and Work and Contracts in Progress","Augmentation (Réduction) des en-cours de fabrication, des produits finis et des commandes en cours d'exécution","Toename (Afname) in de voorraad goederen in bewerking en gereed product en in de bestellingen in uitvoering","Zunahme (Abnahme) des Bestandes an fertigen und unfertigen Erzeugnissen und an in Ausführung befindlichen Bestellungen"
"be_group_712","712","","Increase (Decrease) in Stocks of Work in Progress","Augmentation (Réduction) des stocks d'en-cours de fabrication","Toename (Afname) van de voorraad goederen in bewerking","Zunahme (Abnahme) des Bestandes an unfertigen Erzeugnisse"
"be_group_713","713","","Increase (Decrease) in Stocks of Finished Goods","Augmentation (Réduction) des stocks de produits finis","Toename (Afname) van de voorraad gereed product","Zunahme (Abnahme) des Bestandes an fertigen Erzeugnisse"
"be_group_715","715","","Increase (Decrease) in Stocks of Immovable Property Constructed for Resale","Augmentation (Réduction) des stocks des immeubles construits destinés à la vente","Toename (Afname) van de voorraad onroerende goederen bestemd voor verkoop","Zunahme (Abnahme) des Bestandes an zum Verkauf bestimmten, hergestellten unbeweglichen Gegenstände"
"be_group_717","717","","Increase (Decrease) in Contracts in Progress","Augmentation (Réduction) des commandes en cours d'exécution","Toename (Afname) van de bestellingen in uitvoering","Zunahme (Abnahme) der in Ausführung befindlichen Bestellungen"
"be_group_72","72","","Produced Fixed Assets","Production immobilisée","Geproduceerde vaste activa","Andere aktivierte Eigenleistungen"
"be_group_74","74","","Other Operating Income","Autres produits d'exploitation","Andere bedrijfsopbrengsten","Sonstige betriebliche Erträge"
"be_group_741","741","","Capital Profits on Ordinary Disposal of Tangible Fixed Assets","Plus-values sur réalisations courantes d'immobilisations corporelles","Meerwaarden bij de courante realisatie van materiële vaste activa","Erträge aus dem normalen Abgang von Sachanlagen"
"be_group_742","742","","Capital Profits on Disposal of Trade Debtors","Plus-values sur réalisation de créances commerciales","Meerwaarden bij de courante realisatie van handelsvorderingen","Mehrwerte bei Realisierung von Forderungen aus Lieferungen und Leistungen"
"be_group_743","743","749","Miscellaneous Operating Income","Produits d'exploitation divers","Diverse bedrijfsopbrengsten","Übrige betriebliche Erträge"
"be_group_75","75","","Financial Income","Produits financiers","Financiële opbrengsten","Finanzerträge"
"be_group_750","750","","Income From Financial Fixed Assets","Produits des immobilisations financières","Opbrengsten uit financiële vaste activa","Erträge aus Finanzanlagen"
"be_group_751","751","","Income From Current Assets","Produits des actifs circulants","Opbrengsten uit vlottende activa","Erträge aus Gegenständen des Umlaufvermögens"
"be_group_752","752","","Capital Profits on Disposal of Current Assets","Plus-values sur réalisation d'actifs circulants","Meerwaarden bij de realisatie van vlottende activa","Erträge aus dem Abgang von Gegenständen des Umlaufvermögens"
"be_group_754","754","","Exchange Differences","Différences de change","Wisselresultaten","Wechselkursdifferenzen"
"be_group_755","755","","Foreign Currency Translation Differences","Ecarts de conversion des devises","Resultaten uit de omrekening van vreemde valuta","Umrechnungsdifferenzen von Fremdwährungen"
"be_group_756","756","759","Miscellaneous Financial Income","Produits financiers divers","Diverse financiële opbrengsten","Übrige Finanzerträge"
"be_group_76","76","","Non-Recurring Operating or Financial Expenses","Produits d'exploitation ou financiers non récurrents","Niet-recurrente bedrijfs- of financiële opbrengsten","Nicht wiederkehrende betriebliche oder Finanzerträge"
"be_group_760","760","","Write-Back of Depreciation and of Amounts Written Off","Reprises d'amortissements et de réductions de valeur","Terugneming van afschrijvingen en van waardeverminderingen","Rücknahme von Abschreibungen und Wertminderungen"
"be_group_761","761","","Write-Back of Amounts Written Down Financial Fixed Assets","Reprises de réductions de valeur sur immobilisations financières","Terugneming van waardeverminderingen op financiële vaste activa","Rücknahme von Wertminderungen auf Finanzanlagen"
"be_group_762","762","","Write-Back of Provisions for Non-Recurring Liabilities and Charges","Reprises de provisions pour risques et charges non récurrents","Terugneming van voorzieningen voor niet-recurrente risico's en kosten","Auflösung von Rückstellungen für nicht wiederkehrende Risiken und Aufwendungen"
"be_group_763","763","","Capital Profits on Disposal of Fixed Assets","Plus-values sur réalisation d'actifs immobilisés","Meerwaarden bij de realisatie van vaste activa","Mehrwerte aus dem Abgang von Gegenständen des Anlagevermögens"
"be_group_764","764","768","Other Non-Recurring Operating Income","Autres produits d'exploitation non récurrents","Andere niet-recurrente bedrijfsopbrengsten","Sonstige nicht wiederkehrende betriebliche Erträge"
"be_group_769","769","","Other Non-Recurring Financial Income","Autres produits financiers non récurrents","Andere niet-recurrente financiële opbrengsten","Sonstige nicht wiederkehrende Finanzerträge"
"be_group_78","78","","Transfer From Deferred Taxes and Untaxed Reserves","Prélèvements sur les réserves immunisées et les impôts différés","Onttrekkingen aan de belastingvrije reserves en uitgestelde belastingen","Auflösung von aufgeschobenen Steuern und Entnahmen aus den steuerbegünstigten Rücklagen"
"be_group_780","780","","Transfer From Deferred Taxes","Prélèvement sur les impôts différés","Onttrekking aan de uitgestelde belastingen","Auflösung von aufgeschobenen Steuern"
"be_group_789","789","","Transfer From Untaxed Reserves","Prélèvement sur les réserves immunisées","Onttrekking aan de belastingvrije reserves","Entnahmen aus den steuerfreien Rücklagen"
"be_group_79","79","","Appropriation Account","Affectations et prélèvements","Resultaatverwerking","Ergebnisverwendung"
"be_group_792","792","","Withdrawals From Reserves","Prélèvements sur les réserves","Onttrekking aan de reserves","Entnahmen aus den Rücklagen"

```

## File: data\template\account.group-be_asso.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@fr","name@nl","name@de"
"be_group_10","10","","Association or Foundation Funds","Fonds de l'association ou de la fondation","Fondsen van de vereniging of stichting","Vermögen der Vereinigung oder Stiftung"
"be_group_13","13","","Reserves","Réserves","Reserves","Rücklagen"
"be_group_130","130","","Funds for Investments","Fonds affectés pour investissements","Fondsen bestemd voor investeringen","Für Investitionen bestimmtes Vermögen"
"be_group_131","131","","Funds for Social Liabilities","Fonds affectés pour passif social","Fondsen bestemd voor sociaal passief","Für Sozialverbindlichkeiten bestimmtes Vermögen"
"be_group_139","139","","Other Allocated Funds and Other Reserves","Autres fonds affectés et autres réserves","Andere bestemde fondsen en andere reserves","Anderes zweckgebundenes Vermögen und andere Rücklagen"
"be_group_15","15","","Capital Subsidies","Subsides en capital","Kapitaalsubsidies","Kapitalsubventionen"
"be_group_167","167","","Provisions for Subsidies and Legacies to Reimburse and Gifts With a Recovery Right","Provisions pour remboursement de subsides, legs et dons avec droit de reprise","Voorzieningen voor terug te betalen subsidies, legaten en schenkingen met terugnemingsrecht","Rückstellungen für zurückzuzahlende Subventionen, Legate und Schenkungen mit Rücknahmerecht"
"be_group_413","413","","Subsidies to Be Received","Subsides à recevoir","Te ontvangen subsidies","Zu erhaltende Subventionen"
"be_group_415","415","","Non Interest-Bearing Amounts Receivable or Linked to an Abnormally Low Interest Rate","Créances non productives d'intérêts ou assorties d'un intérêt anormalement faible","Niet-rentedragende vorderingen of gekoppeld aan een abnormaal lage rente","Unverzinsliche Forderungen oder Forderungen mit einem ungewöhnlich niedrigen Zinssatz"
"be_group_46","46","","Advances on Contracts in Progress","Acomptes sur commandes","Vooruitbetalingen op bestellingen","Anzahlungen auf Bestellungen"
"be_group_50","50","","Current Investments Other Than Shares, Fixed-Income Investments and Term Deposits","Placements de trésorerie autres que actions et parts, titres à revenu fixe et dépôts à terme","Geldbeleggingen andere dan aandelen, vastrentende effecten en termijndeposito's","Geldanlagen außer Anteilen, festverzinslichen Wertpapieren und Terminkonten"
"be_group_51","51","","Shares","Actions et parts","Aandelen","Anteile"
"be_group_510","510","","Shares - Acquisition Value","Actions et parts - Valeur d'acquisition","Aandelen - Aanschaffingswaarde","Anteile - Anschaffungswert"
"be_group_511","511","","Shares - Uncalled Amounts (-)","Actions et parts - Montants non appelés (-)","Aandelen - Niet-opgevraagde bedragen (-)","Anteile - Nicht eingeforderte Beträge (-)"
"be_group_519","519","","Shares - Amounts Written Down Recorded (-)","Actions et parts - Réductions de valeur actées (-)","Aandelen - Geboekte waardeverminderingen (-)","Anteile - Gebuchte Wertminderungen (-)"
"be_group_638","638","","Provisions for Subsidies and Legacies to Reimburse and Gifts With a Recovery Right","Provisions pour subsides et legs à rembourser et pour dons avec droit de reprise","Voorzieningen voor terug te betalen subsidies en legaten en voor schenkingen met terugnemingsrecht","Rückstellungen für große Reparaturen und große Wartungsarbeiten"
"be_group_639","639","","Provisions for Other Risks and Charges","Provisions pour autres risques et charges","Voorzieningen voor andere risico's en kosten","Rückstellungen für sonstige Risiken und Aufwendungen"
"be_group_643","643","","Gifts","Dons","Schenkingen","Schenkungen"
"be_group_644","644","648","Miscellaneous Operating Charges","Charges d'exploitation diverses","Diverse bedrijfskosten","Übrige betriebliche Aufwendungen"
"be_group_691","691","","Transfer to Allocated Funds and Other Reserves","Transfert aux fonds affectés et autres réserves","Overboeking naar de bestemde fondsen en andere reserves","Einstellung in das zweckgebundene Vermögen und die sonstigen Rücklagen"
"be_group_692","692","","Positive Result to be Carried Forward","Résultat positif à reporter","Over te dragen positief resultaat","Gewinnvortrag auf neue Rechnung"
"be_group_73","73","","Membership Fees, Gifts, Legacies and Subsidies","Cotisations, dons, legs et subsides","Lidgelden, schenkingen, legaten en subsidies","Beiträge, Schenkungen, Legate und Subventionen"
"be_group_730","730","","Membership Fees","Cotisations","Lidgelden","Beiträge"
"be_group_731","731","","Gifts","Dons","Schenkingen","Schenkungen"
"be_group_732","732","","Legacies","Legs","Legaten","Legate"
"be_group_733","733","","Subsidies","Subsides","Subsidies","Subventionen"
"be_group_741","741","","Capital Profits on Ordinary Disposal of Tangible Fixed Assets","Plus-values sur réalisations courantes d'immobilisations corporelles","Meerwaarden bij de courante realisatie van materiële vaste activa","Erträge aus dem normalen Abgang von Sachanlagen"
"be_group_742","742","","Capital Profits on Disposal of Trade Debtors","Plus-values sur réalisation de créances commerciales","Meerwaarden bij de courante realisatie van handelsvorderingen","Mehrwerte bei Realisierung von Forderungen aus Lieferungen und Leistungen"
"be_group_743","743","749","Miscellaneous Operating Income","Produits d'exploitation divers","Diverse bedrijfsopbrengsten","Übrige betriebliche Erträge"
"be_group_76","76","","Non-Recurring Operating or Financial Expenses","Produits d'exploitation ou financiers non récurrents","Niet-recurrente bedrijfs- of financiële opbrengsten","Nicht wiederkehrende betriebliche oder Finanzerträge"
"be_group_760","760","","Write-Back of Depreciation and of Amounts Written Off","Reprises d'amortissements et de réductions de valeur","Terugneming van afschrijvingen en van waardeverminderingen","Rücknahme von Abschreibungen und Wertminderungen"
"be_group_761","761","","Write-Back of Amounts Written Down Financial Fixed Assets","Reprises de réductions de valeur sur immobilisations financières","Terugneming van waardeverminderingen op financiële vaste activa","Rücknahme von Wertminderungen auf Finanzanlagen"
"be_group_762","762","","Write-Back of Provisions for Non-Recurring Liabilities and Charges","Reprises de provisions pour risques et charges non récurrents","Terugneming van voorzieningen voor niet-recurrente risico's en kosten","Auflösung von Rückstellungen für nicht wiederkehrende Risiken und Aufwendungen"
"be_group_763","763","","Capital Profits on Disposal of Fixed Assets","Plus-values sur réalisation d'actifs immobilisés","Meerwaarden bij de realisatie van vaste activa","Mehrwerte aus dem Abgang von Gegenständen des Anlagevermögens"
"be_group_764","764","768","Other Non-Recurring Operating Income","Autres produits d'exploitation non récurrents","Andere niet-recurrente bedrijfsopbrengsten","Sonstige nicht wiederkehrende betriebliche Erträge"
"be_group_769","769","","Other Non-Recurring Financial Income","Autres produits financiers non récurrents","Andere niet-recurrente financiële opbrengsten","Sonstige nicht wiederkehrende Finanzerträge"
"be_group_77","77","","Adjustment of Income Taxes","Régularisation d'impôts","Regularisering van belastingen","Steuererstattungen"
"be_group_780","780","","Transfer From Deferred Taxes","Prélèvement sur les impôts différés","Onttrekking aan de uitgestelde belastingen","Auflösung von aufgeschobenen Steuern"
"be_group_789","789","","Transfer From Untaxed Reserves","Prélèvement sur les réserves immunisées","Onttrekking aan de belastingvrije reserves","Entnahmen aus den steuerfreien Rücklagen"
"be_group_790","790","","Positive Result From the Preceding Period Brought Forward","Résultat positif de l'exercice antérieur reporté","Overgedragen positief resultaat van het vorige boekjaar","Gewinnvortrag aus dem Vorjahr"
"be_group_791","791","","Other Reserves","Autres réserves","Andere reserves","Sonstige Rücklagen"
"be_group_793","793","","Negative Result to Be Carried Forward","Résultat négatif à reporter","Over te dragen negatief resultaat","Verlustvortrag auf neue Rechnung"

```

## File: data\template\account.group-be_comp.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@fr","name@nl","name@de"
"be_group_10","10","","Capital","Capital","Kapitaal","Kapital"
"be_group_11","11","","Contribution Excluding Capital","Apport hors capital","Inbreng buiten kapitaal","Einlage ohne Kapital"
"be_group_110","110","","Contribution Available Excluding Capital","Apport disponible hors capital","Beschikbare inbreng buiten kapitaal","Verfügbare Einlage ohne Kapital"
"be_group_111","111","","Contribution Not Available Excluding Capital","Apport indisponible hors capital","Onbeschikbare inbreng buiten kapitaal","Nicht verfügbare Einlage ohne Kapital"
"be_group_123","123","","Revaluation Surpluses on Stocks","Plus-values de réévaluation sur stocks","Herwaarderingsmeerwaarden op voorraden","Neubewertungsrücklagen auf Vorräte"
"be_group_13","13","","Reserves","Réserves","Reserves","Rücklagen"
"be_group_130","130","","Legal Reserve","Réserve légale","Wettelijke reserve","Gesetzliche Rücklage"
"be_group_131","131","","Other Reserves Not Available","Autres réserves indisponibles","Andere onbeschikbare reserves","Sonstige nicht verfügbare Rücklagen"
"be_group_133","133","","Available Reserves","Réserves disponibles","Beschikbare reserves","Verfügbare Rücklagen"
"be_group_15","15","","Investment Grants","Subsides en capital","Kapitaalsubsidies","Kapitalsubventionen"
"be_group_19","19","","Advance to Associates on the Sharing Out of the Assets (-)","Avance aux associés sur la répartition de l'actif net (-)","Voorschot aan de vennoten op de verdeling van het netto-actief (-)","Vorschuss an die Gesellschafter auf der Verteilung des Nettoaktiva (-)"
"be_group_410","410","","Called Up Capital or Contribution, Unpaid","Capital ou apport appelé, non versé","Opgevraagd, niet gestort kapitaal of inbreng","Eingefordertes, noch nicht eingezahltes Kapital oder Einlage"
"be_group_47","47","","Debt From Appropriation Result","Dettes découlant de l'affectation du résultat","Schulden uit de bestemming van het resultaat","Verbindlichkeiten, die sich aus der Ergebnisverwendung ergeben"
"be_group_470","470","","Dividends and Director's Fees Relating to Prior Financial Periods","Dividendes et tantièmes d'exercices antérieurs","Dividenden en tantièmes over vorige boekjaren","Dividenden und Tantiemen vorhergehender Geschäftsjahre"
"be_group_471","471","","Dividends - Current Financial Period","Dividendes de l'exercice","Dividenden over het boekjaar","Dividenden für das Geschäftsjahr"
"be_group_472","472","","Director's Fees - Current Financial Period","Tantièmes de l'exercice","Tantièmes over het boekjaar","Tantiemen für das Geschäftsjahr"
"be_group_473","473","","Other Beneficiaries","Autres allocataires","Andere rechthebbenden","Sonstige Berechtigte"
"be_group_50","50","","Own Shares","Actions propres","Eigen aandelen","Eigene Anteile"
"be_group_51","51","","Shares and Current Investments Other Than Fixed Income Investments","Actions, parts et placements autres que placements à revenu fixe","Aandelen en geldbeleggingen andere dan vastrentende beleggingen","Anteile und Geldanlagen, andere als festverzinsliche Anlagen"
"be_group_510","510","","Shares and Current Investments Other Than Fixed Income Investments - Acquisition Value","Actions, parts et placements autres que placements à revenu fixe - Valeur d'acquisition","Aandelen en geldbeleggingen andere dan vastrentende beleggingen - Aanschaffingswaarde","Anteile und Geldanlagen, andere als festverzinsliche Anlagen - Anschaffungswert"
"be_group_511","511","","Shares and Current Investments Other Than Fixed Income Investments - Uncalled Amounts (-)","Actions, parts et placements autres que placements à revenu fixe - Montants non appelés (-)","Aandelen en geldbeleggingen andere dan vastrentende beleggingen - Niet-opgevraagde bedragen (-)","Anteile und Geldanlagen, andere als festverzinsliche Anlagen - Nicht eingeforderte Beträge (-)"
"be_group_519","519","","Shares and Current Investments Other Than Fixed Income Investments - Amounts Written Down Recorded (-)","Actions, parts et placements autres que placements à revenu fixe - Réductions de valeur actées (-)","Aandelen en geldbeleggingen andere dan vastrentende beleggingen - Geboekte waardeverminderingen (-)","Anteile und Geldanlagen, andere als festverzinsliche Anlagen - Gebuchte Wertminderungen (-)"
"be_group_637","637","","Provisions for Environmental Obligations","Provisions pour obligations environnementales","Voorzieningen voor milieuverplichtingen","Rückstellungen für Umweltschutzverpflichtungen"
"be_group_638","638","","Provisions for Other Risks and Charges","Provisions pour autres risques et charges","Voorzieningen voor andere risico's en kosten","Rückstellungen für sonstige Risiken und Aufwendungen"
"be_group_643","643","648","Miscellaneous Operating Charges","Charges d'exploitation diverses","Diverse bedrijfskosten","Übrige betriebliche Aufwendungen"
"be_group_691","691","","Appropriations to Contribution","Affectations à l'apport","Toevoeging aan de inbreng","Zuweisungen an der Einlage"
"be_group_692","692","","Appropriations to Reserves","Dotation aux réserves","Toevoeging aan de reserves","Zuweisungen an die Rücklagen"
"be_group_693","693","","Profits to Be Carried Forward","Bénéfice à reporter","Over te dragen winst","Gewinnvortrag auf neue Rechnung"
"be_group_694","694","","Compensation for Contributions","Rémunération de l'apport","Vergoeding van de inbreng","Vergütung der Einlage"
"be_group_695","695","","Directors' or Managers' Entitlements","Distribution aux administrateurs ou gérants","Uitkering aan bestuurders of zaakvoerders","Verteilung zugunsten der Verwalter oder Geschäftsführer"
"be_group_696","696","","Employees' Entitlements","Distribution aux employés","Uitkering aan werknemers","Verteilung zugunsten der Arbeitnehmer"
"be_group_697","697","","Other Beneficiaries' Entitlements","Distribution aux autres allocataires","Uitkering aan andere rechthebbenden","Verteilung zugunsten andere Berechtigte"
"be_group_740","740","","Operating Subsidies and Compensatory Amounts","Subsides d'exploitation et montants compensatoires","Exploitatiesubsidies en vanwege de overheid ontvangen compenserende bedragen","Betriebssubventionen und von der öffentlichen Hand erhaltene Ausgleichszahlungen"
"be_group_741","741","","Capital Profits on Ordinary Disposal of Tangible Fixed Assets","Plus-values sur réalisations courantes d'immobilisations corporelles","Meerwaarden bij de courante realisatie van materiële vaste activa","Erträge aus dem normalen Abgang von Sachanlagen"
"be_group_742","742","","Capital Profits on Disposal of Trade Debtors","Plus-values sur réalisation de créances commerciales","Meerwaarden bij de courante realisatie van handelsvorderingen","Mehrwerte bei Realisierung von Forderungen aus Lieferungen und Leistungen"
"be_group_743","743","749","Miscellaneous Operating Income","Produits d'exploitation divers","Diverse bedrijfsopbrengsten","Übrige betriebliche Erträge"
"be_group_753","753","","Capital and Interest Subsidies","Subsides en capital et en intérêts","Kapitaal- en interestsubsidies","Kapital- und Zinssubventionen"
"be_group_77","77","","Adjustment of Income Taxes and Write-Back of Tax Provisions","Régularisation d'impôts et reprise de provisions fiscales","Regularisering van belastingen en terugneming van voorzieningen voor belastingen","Steuererstattung und Auflösung von Steuerrückstellungen"
"be_group_771","771","","Adjustment of Belgian Income Taxes","Régularisations d'impôts belges sur le résultat","Regularisering van Belgische belastingen op het resultaat","Erstattung Belgischer Ertragsteuern"
"be_group_773","773","","Adjustment of Foreign Income Taxes","Régularisations d'impôts étrangers sur le résultat","Regularisering van buitenlandse belastingen op het resultaat","Erstattung Ausländischer Ertragsteuern"
"be_group_780","780","","Transfer From Deferred Taxes","Prélèvement sur les impôts différés","Onttrekking aan de uitgestelde belastingen","Auflösung von aufgeschobenen Steuern"
"be_group_789","789","","Transfer From Untaxed Reserves","Prélèvement sur les réserves immunisées","Onttrekking aan de belastingvrije reserves","Entnahmen aus den steuerfreien Rücklagen"
"be_group_790","790","","Profit From the Preceding Period Brought Forward","Bénéfice reporté de l'exercice précédent","Overgedragen winst van het vorige boekjaar","Gewinnvortrag aus dem Vorjahr"
"be_group_791","791","","Withdrawals From Contributions","Prélèvements sur l'apport","Onttrekking aan de inbreng","Entnahmen aus der Einlage"
"be_group_793","793","","Losses to Be Carried Forward","Perte à reporter","Over te dragen verlies","Verlustvortrag auf neue Rechnung"
"be_group_794","794","","Shareholders' (or Owners') Contribution in Respect of Losses","Intervention des associés (ou du propriétaire) dans la perte","Tussenkomst van de vennoten (of van de eigenaar) in het verlies","Teilnahme der Gesellschafter (oder des Eigentümers) am Verlust"

```

## File: data\template\account.tax-be.csv

```csv
"id","sequence","description","invoice_label","name","amount","amount_type","type_tax_use","tax_group_id","tax_scope","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@fr","name@nl"
"attn_VAT-OUT-21-L","10","21% VAT ","21%","21%","21.0","percent","sale","tax_group_tva_21","","","base","invoice","+03","","","21%","21%"
"","","","","","","","","","","","tax","invoice","+54","a451","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","+64","a451","","",""
"attn_VAT-OUT-21-S","11","21% VAT (Services)","21%","21% S","21.0","percent","sale","tax_group_tva_21","service","False","base","invoice","+03","","","",""
"","","","","","","","","","","","tax","invoice","+54","a451","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","+64","a451","","",""
"attn_VAT-OUT-12-S","20","12% VAT (Services)","12%","12% S","12.0","percent","sale","tax_group_tva_12","service","False","base","invoice","+02","","","",""
"","","","","","","","","","","","tax","invoice","+54","a451","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","+64","a451","","",""
"attn_VAT-OUT-12-L","21","12% VAT ","12%","12%","12.0","percent","sale","tax_group_tva_12","","","base","invoice","+02","","","12%","12%"
"","","","","","","","","","","","tax","invoice","+54","a451","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","+64","a451","","",""
"attn_VAT-OUT-06-S","30","6% VAT (Services)","6%","6% S","6.0","percent","sale","tax_group_tva_6","service","False","base","invoice","+01","","","",""
"","","","","","","","","","","","tax","invoice","+54","a451","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","+64","a451","","",""
"attn_VAT-OUT-06-L","31","6% VAT ","6%","6%","6.0","percent","sale","tax_group_tva_6","","","base","invoice","+01","","","",""
"","","","","","","","","","","","tax","invoice","+54","a451","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","+64","a451","","",""
"attn_VAT-OUT-00-S","40","6% VAT (Services)","0%","0% S","0.0","percent","sale","tax_group_tva_0","service","False","base","invoice","+00","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-OUT-00-L","41","0% VAT ","0%","0%",".00","percent","sale","tax_group_tva_0","","","base","invoice","+00","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-OUT-00-CC","50","0% Co-contracting ","0%","0% Cocont","0.0","percent","sale","tax_group_tva_0","","","base","invoice","+45","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-OUT-00-EU-S","60","0% Intra-Community (Services)","0%","0% EU S","0.0","percent","sale","tax_group_tva_0","service","","base","invoice","+44","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","+48s44","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-OUT-00-EU-L","61","0% Intra-Community (Goods)","0%","0% EU M","0.0","percent","sale","tax_group_tva_0","merch","","base","invoice","+46L","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","+48s46L","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-OUT-00-EU-T","62","0% Intra-Community (Triangular)","0%","0% EU T","0.0","percent","sale","tax_group_tva_0","","","base","invoice","+46T","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","+48s46T","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-OUT-00-ROW","70","0% Extra-Community","0%","0% EX","0.0","percent","sale","tax_group_tva_0","","","base","invoice","+47","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V81-21","110","21% VAT (Merchandise)","21%","21% M","21.0","percent","purchase","tax_group_tva_21","merch","","base","invoice","+81","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-81||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V81-12","120","12% VAT (Merchandise)","12%","12% M","12.0","percent","purchase","tax_group_tva_12","merch","","base","invoice","+81","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-81||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V81-06","130","6% VAT (Merchandise)","6%","6% M","6.0","percent","purchase","tax_group_tva_6","merch","","base","invoice","+81","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-81||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V81-00","140","0% VAT (Merchandise)","0%","0% M","0.0","percent","purchase","tax_group_tva_0","merch","","base","invoice","+81","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-81||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_TVA-21-inclus-dans-prix","150","21% VAT included (Services)","21%","21% S.TTC","21","percent","purchase","tax_group_tva_21","service","","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V82-21-S","210","21% VAT (Services)","21%","21% S","21.0","percent","purchase","tax_group_tva_21","service","","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V82-21-G","220","21% VAT (Goods)","21%","21% G","21.0","percent","purchase","tax_group_tva_21","consu","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V82-12-S","230","12% VAT (Services)","12%","12% S","12.0","percent","purchase","tax_group_tva_12","service","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V82-12-G","240","12% VAT (Goods)","12%","12% G","12.0","percent","purchase","tax_group_tva_12","consu","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V82-06-S","250","6% VAT (Services)","6%","6% S","6.0","percent","purchase","tax_group_tva_6","service","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V82-06-G","260","6% VAT (Goods)","6%","6% G","6.0","percent","purchase","tax_group_tva_6","consu","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V82-00-S","270","0% VAT (Services)","0%","0% S","0.0","percent","purchase","tax_group_tva_0","service","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V82-00-G","280","0% VAT (Goods)","0%","0% G","0.0","percent","purchase","tax_group_tva_0","consu","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-82||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V83-21","310","21% VAT (Investment Goods)","21%","21% IG","21.0","percent","purchase","tax_group_tva_21","invest","False","base","invoice","+83","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-83||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V83-12","320","12% VAT (Investment Goods)","12%","12% IG","12.0","percent","purchase","tax_group_tva_12","invest","False","base","invoice","+83","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-83||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V83-06","330","6% VAT (Investment Goods)","6%","6% IG","6.0","percent","purchase","tax_group_tva_6","invest","False","base","invoice","+83","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","base","refund","-83||+85","","","",""
"","","","","","","","","","","","tax","refund","+63","a411","","",""
"attn_VAT-IN-V83-00","340","0% VAT (Investment Goods)","0%","0% IG","0.0","percent","purchase","tax_group_tva_0","invest","False","base","invoice","+83","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-83||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V81-21-CC","410","21% VAT Co-contracting (Merchandise)","21%","21% M.Cocont","21.0","percent","purchase","tax_group_tva_21","merch","","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V81-12-CC","420","12% VAT Co-contracting (Merchandise)","12%","12% M.Cocont","12.0","percent","purchase","tax_group_tva_12","merch","False","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V81-06-CC","430","6% VAT Co-contracting (Merchandise)","6%","6% M.Cocont","6.0","percent","purchase","tax_group_tva_6","merch","False","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V81-00-CC","440","0% VAT Co-contracting (Merchandise)","0%","0% M.Cocont","0.0","percent","purchase","tax_group_tva_0","merch","False","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V82-21-CC","510","21% VAT Co-contracting (Services)","21%","21% S.Cocont","21.0","percent","purchase","tax_group_tva_21","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V82-12-CC","520","12% VAT Co-contracting (Services)","12%","12% S.Cocont","12.0","percent","purchase","tax_group_tva_12","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V82-06-CC","530","6% VAT Co-contracting (Services)","6%","6% S.Cocont","6.0","percent","purchase","tax_group_tva_6","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V82-00-CC","540","0% VAT Co-contracting (Services)","0%","0% S.Cocont","0.0","percent","purchase","tax_group_tva_0","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V83-21-CC","610","21% VAT Co-contracting (Investment Goods)","21%","21% IG.Cocont","21.0","percent","purchase","tax_group_tva_21","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V83-12-CC","620","12% VAT Co-contracting (Investment Goods)","12%","12% IG.Cocont","12.0","percent","purchase","tax_group_tva_12","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V83-06-CC","630","6% VAT Co-contracting (Investment Goods)","6%","6% IG.Cocont","6.0","percent","purchase","tax_group_tva_6","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451056","-100","",""
"attn_VAT-IN-V83-00-CC","640","0% VAT Co-contracting (Investment Goods)","0%","0% IG.Cocont","0.0","percent","purchase","tax_group_tva_0","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V82-CAR-EXC","720","21% VAT (Services) with 50% VAT deductibility","21%","21% D50","21.0","percent","purchase","tax_group_tva_21","service","","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+82","","50","",""
"","","","","","","","","","","","tax","invoice","+59","a411","50","",""
"","","","","","","","","","","","base","refund","+85||-82","","","",""
"","","","","","","","","","","","tax","refund","-82||+85","","50","",""
"","","","","","","","","","","","tax","refund","+63","a451","50","",""
"attn_VAT-IN-V82-D35","730","21% VAT (Services) with 35% VAT deductibility","21%","21% D35","21.0","percent","purchase","tax_group_tva_21","service","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+82","","65","",""
"","","","","","","","","","","","tax","invoice","+59","a411","35","",""
"","","","","","","","","","","","base","refund","+85||-82","","","",""
"","","","","","","","","","","","tax","refund","-82||+85","","65","",""
"","","","","","","","","","","","tax","refund","+63","a451","35","",""
"attn_VAT-IN-V82-D85","740","21% VAT (Services) with 85% VAT deductibility","21%","21% D85","21.0","percent","purchase","tax_group_tva_21","service","False","base","invoice","+82","","","",""
"","","","","","","","","","","","tax","invoice","+82","","15","",""
"","","","","","","","","","","","tax","invoice","+59","a411","85","",""
"","","","","","","","","","","","base","refund","+85||-82","","","",""
"","","","","","","","","","","","tax","refund","-82||+85","","15","",""
"","","","","","","","","","","","tax","refund","+63","a451","85","",""
"attn_VAT-IN-V81-21-EU","1110","21% VAT Intra-Community (Merchandise)","21%","21% EU M","21.0","percent","purchase","tax_group_tva_21","merch","","base","invoice","+81||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-81||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V81-12-EU","1120","12% VAT Intra-Community (Merchandise)","12%","12% EU M","12.0","percent","purchase","tax_group_tva_12","merch","False","base","invoice","+81||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-81||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V81-06-EU","1130","6% VAT Intra-Community (Merchandise)","6%","6% EU M","6.0","percent","purchase","tax_group_tva_6","merch","False","base","invoice","+81||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-81||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V81-00-EU","1140","0% VAT Intra-Community (Merchandise)","0%","0% EU M","0.0","percent","purchase","tax_group_tva_0","merch","False","base","invoice","+81||+86","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-81||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V82-21-EU-S","1210","21% VAT Intra-Community (Services)","21%","21% EU S","21.0","percent","purchase","tax_group_tva_21","service","False","base","invoice","+82||+88","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-88||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V82-21-EU-G","1220","21% VAT Intra-Community (Goods)","21%","21% EU G","21.0","percent","purchase","tax_group_tva_21","consu","False","base","invoice","+82||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V82-21-EU-G-D50","1223","21% VAT Intra-Community with 50% VAT deductibility (Goods)","21%","21% EU G D50","21.0","percent","purchase","tax_group_tva_21","consu","False","base","invoice","+82||+86","","","",""
"","","","","","","","","","","","tax","invoice","+82","","50","",""
"","","","","","","","","","","","tax","invoice","+59","a411","50","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","-82","","50","",""
"","","","","","","","","","","","tax","refund","-62","a451055","50","",""
"","","","","","","","","","","","tax","refund","-62","a411","-100","",""
"attn_VAT-IN-V82-21-EU-G-D35","1224","21% VAT Intra-Community with 35% VAT deductibility (Goods)","21%","21% EU G D35","21.0","percent","purchase","tax_group_tva_21","consu","False","base","invoice","+82||+86","","","",""
"","","","","","","","","","","","tax","invoice","+82","","65","",""
"","","","","","","","","","","","tax","invoice","+59","a411","35","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","-82","","65","",""
"","","","","","","","","","","","tax","refund","-62","a451055","35","",""
"","","","","","","","","","","","tax","refund","-62","a411","-100","",""
"attn_VAT-IN-V82-21-EU-S-D50","1225","21% VAT Intra-Community with 50% VAT deductibility (Services)","21%","21% EU S D50","21.0","percent","purchase","tax_group_tva_21","service","False","base","invoice","+82||+88","","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","tax","invoice","+59","a411","50","",""
"","","","","","","","","","","","tax","invoice","+82||+88","","50","",""
"","","","","","","","","","","","base","refund","-82||-88||+84","","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"","","","","","","","","","","","tax","refund","+62","a411","50","",""
"","","","","","","","","","","","tax","refund","-82||-88||+84","","50","",""
"attn_VAT-IN-V82-21-EU-G-D35-ALRD-IN-BE","1226","21% VAT Intra-Community with 35% VAT deductibility (Goods already in Belgium)","21%","21% EU G D35 already in Belgium","21.0","percent","purchase","tax_group_tva_21","consu","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","+82","","65","",""
"","","","","","","","","","","","tax","invoice","+59","a411","35","",""
"","","","","","","","","","","","tax","invoice","+56","a451056","-100","",""
"","","","","","","","","","","","base","refund","-82||+85||-87","","","",""
"","","","","","","","","","","","tax","refund","-82","","65","",""
"","","","","","","","","","","","tax","refund","-62","a451056","35","",""
"","","","","","","","","","","","tax","refund","-62","a411","-100","",""
"attn_VAT-IN-V82-12-EU-S","1230","12% VAT Intra-Community (Services)","12%","12% EU S","12.0","percent","purchase","tax_group_tva_12","service","False","base","invoice","+82||+88","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-88||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V82-12-EU-G","1240","12% VAT Intra-Community (Goods)","12%","12% EU G","12.0","percent","purchase","tax_group_tva_12","consu","False","base","invoice","+82||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V82-06-EU-S","1250","6% VAT Intra-Community (Services)","6%","6% EU S","6.0","percent","purchase","tax_group_tva_6","service","False","base","invoice","+82||+88","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-88||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V82-06-EU-G","1260","6% VAT Intra-Community (Goods)","6%","6% EU G","6.0","percent","purchase","tax_group_tva_6","consu","False","base","invoice","+82||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-82||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V82-00-EU-S","1270","0% VAT Intra-Community (Services)","0%","0% EU S","0.0","percent","purchase","tax_group_tva_0","service","False","base","invoice","+82||+88","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-82||-88||+84","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V83-21-EU","1310","21% VAT Intra-Community (Investment Goods)","21%","21% EU IG","21.0","percent","purchase","tax_group_tva_21","invest","False","base","invoice","+83||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-83||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V82-00-EU-G","1280","0% VAT Intra-Community (Goods)","0%","0% EU G","0.0","percent","purchase","tax_group_tva_0","consu","False","base","invoice","+82||+86","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-82||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V83-12-EU","1320","12% VAT Intra-Community (Investment Goods)","12%","12% EU IG","12.0","percent","purchase","tax_group_tva_12","invest","False","base","invoice","+83||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-83||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V83-06-EU","1330","6% VAT Intra-Community (Investment Goods)","6%","6% EU IG","6.0","percent","purchase","tax_group_tva_6","invest","False","base","invoice","+83||+86","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-55","a451055","-100","",""
"","","","","","","","","","","","base","refund","-83||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451055","-100","",""
"attn_VAT-IN-V83-00-EU","1340","0% VAT Intra-Community (Investment Goods)","0%","0% EU IG","0.0","percent","purchase","tax_group_tva_0","invest","False","base","invoice","+83||+86","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-83||-86||+84","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V81-21-ROW-CC","2110","21% Extra-Community (Merchandise)","21%","21% EX M","21.0","percent","purchase","tax_group_tva_21","merch","","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V81-12-ROW-CC","2120","12% Extra-Community (Merchandise)","12%","12% EX M","12.0","percent","purchase","tax_group_tva_12","merch","False","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V81-06-ROW-CC","2130","6% Extra-Community (Merchandise)","6%","6% EX M","6.0","percent","purchase","tax_group_tva_6","merch","False","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V81-00-ROW-CC","2140","0% Extra-Community (Merchandise)","0%","0% EX M","0.0","percent","purchase","tax_group_tva_0","merch","False","base","invoice","+81||+87","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-81||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V82-21-ROW-CC","2210","21% Extra-Community (Services)","21%","21% EX S","21.0","percent","purchase","tax_group_tva_21","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V82-12-ROW-CC","2220","12% Extra-Community (Services)","12%","12% EX S","12.0","percent","purchase","tax_group_tva_12","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V82-06-ROW-CC","2230","6% Extra-Community (Services)","6%","6% EX S","6.0","percent","purchase","tax_group_tva_6","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V82-00-ROW-CC","2240","0% Extra-Community (Services)","0%","0% EX S","0.0","percent","purchase","tax_group_tva_0","service","False","base","invoice","+82||+87","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-82||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
"attn_VAT-IN-V83-21-ROW-CC","2310","21% Extra-Community (Investment Goods)","21%","21% EX IG","21.0","percent","purchase","tax_group_tva_21","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V83-12-ROW-CC","2320","12% Extra-Community (Investment Goods)","12%","12% EX IG","12.0","percent","purchase","tax_group_tva_12","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V83-06-ROW-CC","2330","6% Extra-Community (Investment Goods)","6%","6% EX IG","6.0","percent","purchase","tax_group_tva_6","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","+59","a411","","",""
"","","","","","","","","","","","tax","invoice","-57","a451057","-100","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","a411","","",""
"","","","","","","","","","","","tax","refund","","a451057","-100","",""
"attn_VAT-IN-V83-00-ROW-CC","2340","0% Extra-Community (Investment Goods)","0%","0% EX IG","0.0","percent","purchase","tax_group_tva_0","invest","False","base","invoice","+83||+87","","","",""
"","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","base","refund","-83||-87||+85","","","",""
"","","","","","","","","","","","tax","refund","","","","",""
```

## File: data\template\account.tax.group-be.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","pos_receipt_label","name@fr","name@nl"
"tax_group_tva_21","VAT 21%","base.be","a4112","a4512","A","TVA 21%","BTW 21%"
"tax_group_tva_12","VAT 12%","base.be","a4112","a4512","B","TVA 12%","BTW 12%"
"tax_group_tva_6","VAT 6%","base.be","a4112","a4512","C","TVA 6%","BTW 6%"
"tax_group_tva_0","VAT 0%","base.be","a4112","a4512","D","TVA 0%","BTW 0%"

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[
        ('be', 'Belgium')
        ], ondelete={'be': lambda recs: recs.write({'invoice_reference_model': 'odoo'})})

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Noviat nv/sa (www.noviat.be). All rights reserved.

import random
import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError

"""
account.move object: add support for Belgian structured communication
"""


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_invoice_reference_be_partner(self):
        """ This computes the reference based on the belgian national standard
            “OGM-VCS”.
            For instance, if an invoice is issued for the partner with internal
            reference 'food buyer 654', the digits will be extracted and used as
            the data. This will lead to a check number equal to 72 and the
            reference will be '+++000/0000/65472+++'.
            If no reference is set for the partner, its id in the database will
            be used.
        """
        self.ensure_one()
        bbacomm = (re.sub(r'\D', '', self.partner_id.ref or '') or str(self.partner_id.id))[-10:].rjust(10, '0')
        base = int(bbacomm)
        mod = base % 97 or 97
        reference = '+++%s/%s/%s%02d+++' % (bbacomm[:3], bbacomm[3:7], bbacomm[7:], mod)
        return reference

    def _get_invoice_reference_be_invoice(self):
        """ This computes the reference based on the belgian national standard
            “OGM-VCS”.
            The data of the reference is the database id number of the invoice.
            For instance, if an invoice is issued with id 654, the check number
            is 72 so the reference will be '+++000/0000/65472+++'.
        """
        self.ensure_one()
        base = self.id
        bbacomm = str(base).rjust(10, '0')
        base = int(bbacomm)
        mod = base % 97 or 97
        reference = '+++%s/%s/%s%02d+++' % (bbacomm[:3], bbacomm[3:7], bbacomm[7:], mod)
        return reference

```

## File: models\account_tax.py

```python
from odoo import fields, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    tax_scope = fields.Selection(
        selection_add=[('merch', 'Merchandise'), ('invest', 'Investment')],
    )

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Noviat nv/sa (www.noviat.be). All rights reserved.

from odoo import api, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    @api.depends('vat', 'country_id')
    def _compute_company_registry(self):
        # OVERRIDE
        # If a belgian company has a VAT number then its company registry is its VAT Number (without country code).
        super()._compute_company_registry()
        for partner in self.filtered(lambda p: p._deduce_country_code() == 'BE' and p.vat):
            vat_country, vat_number = self._split_vat(partner.vat)
            if vat_country.isnumeric():
                vat_country = 'be'
                vat_number = partner.vat
            if vat_country == 'be' and self.simple_vat_check(vat_country, vat_number):
                partner.company_registry = vat_number

```

## File: models\template_be.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import Command, _, models

from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('be')
    def _get_be_template_data(self):
        return {
            'name': _('Base'),
            'visible': False,
            'code_digits': '6',
            'property_account_receivable_id': 'a400',
            'property_account_payable_id': 'a440',
            'property_account_expense_categ_id': 'a600',
            'property_account_income_categ_id': 'a7000',
        }

    @template('be', 'res.company')
    def _get_be_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.be',
                'bank_account_code_prefix': '550',
                'cash_account_code_prefix': '570',
                'transfer_account_code_prefix': '580',
                'account_default_pos_receivable_account_id': 'a4001',
                'income_currency_exchange_account_id': 'a754',
                'expense_currency_exchange_account_id': 'a654',
                'account_journal_suspense_account_id': 'a499',
                'account_journal_early_pay_discount_loss_account_id': 'a657000',
                'account_journal_early_pay_discount_gain_account_id': 'a757000',
                'account_sale_tax_id': 'attn_VAT-OUT-21-L',
                'account_purchase_tax_id': 'attn_VAT-IN-V81-21',
                'default_cash_difference_income_account_id': 'a757100',
                'default_cash_difference_expense_account_id': 'a657100',
                'transfer_account_id': 'a58',
            },
        }

    @template('be', 'account.journal')
    def _get_be_account_journal(self):
        return {
            'sale': {'refund_sequence': True},
            'purchase': {'refund_sequence': True},
        }

    @template('be', 'account.reconcile.model')
    def _get_be_reconcile_model(self):
        return {
            'escompte_template': {
                'name': 'Cash Discount',
                'line_ids': [
                    Command.create({
                        'account_id': 'a653',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Cash Discount Granted',
                    }),
                ],
                'name@fr': 'Escompte',
                'name@nl': 'Betalingskorting',
                'name@de': 'Skonto',
            },
            'frais_bancaires_htva_template': {
                'name': 'Bank Fees (No VAT)',
                'line_ids': [
                    Command.create({
                        'account_id': 'a6560',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                        'label': 'Bank Fees (No VAT)',
                    }),
                ],
                'name@fr': 'Frais bancaires (Hors TVA)',
                'name@nl': 'Bankkosten (Geen BTW)',
                'name@de': 'Bankgebühren (Ohne MwSt.)',
            },
        }

```

## File: models\template_be_asso.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _, models

from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('be_asso')
    def _get_be_asso_template_data(self):
        return {
            'name': _('Associations and Foundations'),
            'parent': 'be',
            'code_digits': '6',
        }

    @template('be_asso', 'res.company')
    def _get_be_asso_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.be',
                'bank_account_code_prefix': '550',
                'cash_account_code_prefix': '570',
                'transfer_account_code_prefix': '580',
            },
        }

```

## File: models\template_be_comp.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _, models

from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('be_comp')
    def _get_be_comp_template_data(self):
        return {
            'name': _('Companies'),
            'parent': 'be',
            'code_digits': '6',
            'sequence': 0,
        }

    @template('be_comp', 'res.company')
    def _get_be_comp_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.be',
                'bank_account_code_prefix': '550',
                'cash_account_code_prefix': '570',
                'transfer_account_code_prefix': '580',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_be
from . import template_be_comp
from . import template_be_asso
from . import account_journal
from . import account_move
from . import account_tax
from . import res_partner

```

