# Odoo Module: l10n_eg

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
    'name': "Egypt - Accounting",
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/egypt.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['eg'],
    'description': """
Egypt Accounting Module
==============================================================================
Egypt Accounting Basic Charts and Localization.

Activates:

- Chart of Accounts
- Taxes
- VAT Return
- Withholding Tax Report
- Schedule Tax Report
- Other Taxes Report
- Fiscal Positions
    """,
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.0',
    'depends': [
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_tax_report_data.xml',
        'views/account_tax.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/demo_partner.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report_vat_return" model="account.report">
        <field name="name">1. VAT Return</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.eg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_vat_return_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_vat_return_sale_base" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Base)</field>
                <field name="aggregation_formula">EG_STD_SALE_B.balance + EG_ZERO_SALE_B.balance + EG_EXM_SALE_B.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_vat_return_sale_base_fourteen" model="account.report.line">
                        <field name="name">1. Standard Rated 14% (Base)</field>
                        <field name="code">EG_STD_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_sale_base_fourteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1. VAT 14% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_sale_base_zero" model="account.report.line">
                        <field name="name">2. Zero Rated (Base)</field>
                        <field name="code">EG_ZERO_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_sale_base_zero_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Zero Rated (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_sale_base_exempt" model="account.report.line">
                        <field name="name">3. Exempt Sales (Base)</field>
                        <field name="code">EG_EXM_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_sale_base_exempt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Exempt Sales (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_return_sale_tax" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Tax)</field>
                <field name="aggregation_formula">EG_STD_SALE_T.balance + EG_ZERO_SALE_T.balance + EG_EXM_SALE_T.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_vat_return_sale_tax_fourteen" model="account.report.line">
                        <field name="name">1. Standard Rated 14% (Tax)</field>
                        <field name="code">EG_STD_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_sale_tax_fourteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1. VAT 14% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_sale_tax_zero" model="account.report.line">
                        <field name="name">2. Zero Rated (Tax)</field>
                        <field name="code">EG_ZERO_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_sale_tax_zero_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Zero Rated (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_sale_tax_exempt" model="account.report.line">
                        <field name="name">3. Exempt Sales (Tax)</field>
                        <field name="code">EG_EXM_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_sale_tax_exempt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Exempt Sales (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_return_expense_base" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Base)</field>
                <field name="aggregation_formula">EG_STD_PUR_B.balance + EG_ZERO_PUR_B.balance + EG_EXM_PUR_B.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_vat_return_expense_base_fourteen" model="account.report.line">
                        <field name="name">5. Standard Rated 14% Expenses (Base)</field>
                        <field name="code">EG_STD_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_expense_base_fourteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. VAT 14% Expenses (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_expense_base_zero" model="account.report.line">
                        <field name="name">6. Zero Rated (Base)</field>
                        <field name="code">EG_ZERO_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_expense_base_zero_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6. Zero Rated (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_expense_base_exempt" model="account.report.line">
                        <field name="name">7. Exempt Expenses (Base)</field>
                        <field name="code">EG_EXM_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_expense_base_exempt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Exempt Expenses (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_return_expense_tax" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Tax)</field>
                <field name="aggregation_formula">EG_STD_PUR_T.balance + EG_ZERO_PUR_T.balance + EG_EXM_PUR_T.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_vat_return_expense_tax_fourteen" model="account.report.line">
                        <field name="name">5. Standard Rated 14% Expenses (Tax)</field>
                        <field name="code">EG_STD_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_expense_tax_fourteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. VAT 14% Expenses (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_expense_tax_zero" model="account.report.line">
                        <field name="name">6. Zero Rated (Tax)</field>
                        <field name="code">EG_ZERO_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_expense_tax_zero_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6. Zero Rated (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_return_expense_tax_exempt" model="account.report.line">
                        <field name="name">7. Exempt Expenses (Tax)</field>
                        <field name="code">EG_EXM_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_return_expense_tax_exempt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Exempt Expenses (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_return_net" model="account.report.line">
                <field name="name">Net VAT Due</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_vat_return_net_1" model="account.report.line">
                        <field name="name">Total value of due tax for the period</field>
                        <field name="aggregation_formula">EG_STD_SALE_T.balance + EG_ZERO_SALE_T.balance + EG_EXM_SALE_T.balance</field>
                    </record>
                    <record id="tax_report_vat_return_net_2" model="account.report.line">
                        <field name="name">Total value of recoverable tax for the period</field>
                        <field name="aggregation_formula">EG_STD_PUR_T.balance + EG_ZERO_PUR_T.balance + EG_EXM_PUR_T.balance</field>
                    </record>
                    <record id="tax_report_vat_return_net_3" model="account.report.line">
                        <field name="name">Net VAT due (or reclaimed) for the period</field>
                        <field name="aggregation_formula">EG_STD_SALE_T.balance + EG_ZERO_SALE_T.balance + EG_EXM_SALE_T.balance - (EG_STD_PUR_T.balance + EG_ZERO_PUR_T.balance + EG_EXM_PUR_T.balance)</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
    <record id="tax_report_withholding_tax" model="account.report">
        <field name="name">2. Withholding Tax</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.eg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_withholding_tax_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_withholding_tax_sale_base" model="account.report.line">
                <field name="name">Withholding Tax on Sales (Base)</field>
                <field name="aggregation_formula">EG_H_SALE_B.balance + EG_O_SALE_B.balance + EG_T_SALE_B.balance + EG_F_SALE_B.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_withholding_tax_sale_base_half" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -0.5% (Base)</field>
                        <field name="code">EG_H_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_base_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Sales -0.5% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_sale_base_one" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -1% (Base)</field>
                        <field name="code">EG_O_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_base_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH on Sales -1% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_sale_base_three" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -3% (Base)</field>
                        <field name="code">EG_T_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_base_three_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH on Sales -3% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_sale_base_five" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -5% (Base)</field>
                        <field name="code">EG_F_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_base_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH on Sales -5% (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_withholding_tax_sale_tax" model="account.report.line">
                <field name="name">Withholding Tax on Sales (Tax)</field>
                <field name="aggregation_formula">EG_H_SALE_T.balance + EG_O_SALE_T.balance + EG_T_SALE_T.balance + EG_F_SALE_T.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_withholding_tax_sale_tax_half" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -0.5% (Tax)</field>
                        <field name="code">EG_H_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_tax_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Sales -0.5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_sale_tax_one" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -1% (Tax)</field>
                        <field name="code">EG_O_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_tax_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Sales -1% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_sale_tax_three" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -3% (Tax)</field>
                        <field name="code">EG_T_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_tax_three_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Sales -3% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_sale_tax_five" model="account.report.line">
                        <field name="name">Withholding Tax on Sales -5% (Tax)</field>
                        <field name="code">EG_F_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_sale_tax_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Sales -5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_withholding_tax_purchase_base" model="account.report.line">
                <field name="name">Withholding Tax on Purchases (Base)</field>
                <field name="aggregation_formula">EG_H_PUR_B.balance + EG_O_PUR_B.balance + EG_T_PUR_B.balance + EG_F_PUR_B.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_withholding_tax_purchase_base_half" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -0.5% (Base)</field>
                        <field name="code">EG_H_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_base_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -0.5% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_purchase_base_one" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -1% (Base)</field>
                        <field name="code">EG_O_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_base_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -1% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_purchase_base_three" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -3% (Base)</field>
                        <field name="code">EG_T_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_base_three_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -3% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_purchase_base_five" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -5% (Base)</field>
                        <field name="code">EG_F_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_base_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -5% (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_withholding_tax_purchase_tax" model="account.report.line">
                <field name="name">Withholding Tax on Purchases (Tax)</field>
                <field name="aggregation_formula">EG_H_PUR_T.balance + EG_O_PUR_T.balance + EG_T_PUR_T.balance + EG_F_PUR_T.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_withholding_tax_purchase_tax_half" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -0.5% (Tax)</field>
                        <field name="code">EG_H_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_tax_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -0.5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_purchase_tax_one" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -1% (Tax)</field>
                        <field name="code">EG_O_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_tax_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -1% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_purchase_tax_three" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -3% (Tax)</field>
                        <field name="code">EG_T_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_tax_three_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -3% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_withholding_tax_purchase_tax_five" model="account.report.line">
                        <field name="name">Withholding Tax on Purchases -5% (Tax)</field>
                        <field name="code">EG_F_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_withholding_tax_purchase_tax_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH Purchases -5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
    <record id="tax_report_schedule_tax" model="account.report">
        <field name="name">3. Schedule Tax</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.eg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_schedule_tax_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_schedule_tax_schedule_tax_sale_base" model="account.report.line">
                <field name="name">Schedule Tax on Sales (Base)</field>
                <field name="aggregation_formula">EG_H_SALE_SB.balance + EG_O_SALE_SB.balance + EG_F_SALE_SB.balance + EG_E_SALE_SB.balance + EG_T_SALE_SB.balance + EG_FF_SALE_SB.balance + EG_TY_SALE_SB.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_schedule_tax_schedule_tax_sale_base_half" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 0.5% (Base)</field>
                        <field name="code">EG_H_SALE_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_base_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 0.5% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_base_one" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 1% (Base)</field>
                        <field name="code">EG_O_SALE_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_base_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 1% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_base_five" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 5% (Base)</field>
                        <field name="code">EG_F_SALE_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_base_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 5% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_base_eight" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 8% (Base)</field>
                        <field name="code">EG_E_SALE_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_base_eight_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 8% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_base_ten" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 10% (Base)</field>
                        <field name="code">EG_T_SALE_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_base_ten_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 10% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_base_fifteen" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 15% (Base)</field>
                        <field name="code">EG_FF_SALE_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_base_fifteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 15% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_base_thirty" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 30% (Base)</field>
                        <field name="code">EG_TY_SALE_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_base_thirty_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 30% (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_schedule_tax_schedule_tax_sale_tax" model="account.report.line">
                <field name="name">Schedule Tax on Sales (Tax)</field>
                <field name="aggregation_formula">EG_H_SALE_ST.balance + EG_O_SALE_ST.balance + EG_F_SALE_ST.balance + EG_E_SALE_ST.balance + EG_T_SALE_ST.balance + EG_FF_SALE_ST.balance + EG_TY_SALE_ST.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_half" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 0.5% (Tax)</field>
                        <field name="code">EG_H_SALE_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_tax_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 0.5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_one" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 1% (Tax)</field>
                        <field name="code">EG_O_SALE_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_tax_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 1% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_five" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 5% (Tax)</field>
                        <field name="code">EG_F_SALE_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_tax_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_eight" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 8% (Tax)</field>
                        <field name="code">EG_E_SALE_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_tax_eight_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 8% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_ten" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 10% (Tax)</field>
                        <field name="code">EG_T_SALE_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_tax_ten_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 10% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_fifteen" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 15% (Tax)</field>
                        <field name="code">EG_FF_SALE_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_tax_fifteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_sale_tax_thirty" model="account.report.line">
                        <field name="name">Schedule Tax on Sales 30% (Tax)</field>
                        <field name="code">EG_TY_SALE_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_sale_tax_thirty_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Sales 30% (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_schedule_tax_schedule_tax_purchase_base" model="account.report.line">
                <field name="name">Schedule Tax on Purchases (Base)</field>
                <field name="aggregation_formula">EG_H_PUR_SB.balance + EG_O_PUR_SB.balance + EG_F_PUR_SB.balance + EG_E_PUR_SB.balance + EG_T_PUR_SB.balance + EG_FF_PUR_SB.balance + EG_TY_PUR_SB.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_half" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 0.5% (Base)</field>
                        <field name="code">EG_H_PUR_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_base_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 0.5% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_one" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 1% (Base)</field>
                        <field name="code">EG_O_PUR_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_base_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 1% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_five" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 5% (Base)</field>
                        <field name="code">EG_F_PUR_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_base_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 5% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_eight" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 8% (Base)</field>
                        <field name="code">EG_E_PUR_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_base_eight_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 8% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_ten" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 10% (Base)</field>
                        <field name="code">EG_T_PUR_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_base_ten_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 10% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_fifteen" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 15% (Base)</field>
                        <field name="code">EG_FF_PUR_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_base_fifteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 15% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_base_thirty" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 30% (Base)</field>
                        <field name="code">EG_TY_PUR_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_base_thirty_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 30% (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax" model="account.report.line">
                <field name="name">Schedule Tax on Purchases (Tax)</field>
                <field name="aggregation_formula">EG_H_PUR_ST.balance + EG_O_PUR_ST.balance + EG_F_PUR_ST.balance + EG_E_PUR_ST.balance + EG_T_PUR_ST.balance + EG_FF_PUR_ST.balance + EG_TY_PUR_ST.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_half" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 0.5% (Tax)</field>
                        <field name="code">EG_H_PUR_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_half_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 0.5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_one" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 1% (Tax)</field>
                        <field name="code">EG_O_PUR_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_one_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 1% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_five" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 5% (Tax)</field>
                        <field name="code">EG_F_PUR_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_five_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_eight" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 8% (Tax)</field>
                        <field name="code">EG_E_PUR_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_eight_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 8% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_ten" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 10% (Tax)</field>
                        <field name="code">EG_T_PUR_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_ten_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 10% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 15% (Tax)</field>
                        <field name="code">EG_FF_PUR_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_thirty" model="account.report.line">
                        <field name="name">Schedule Tax on Purchases 30% (Tax)</field>
                        <field name="code">EG_TY_PUR_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_schedule_tax_schedule_tax_purchase_tax_thirty_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SCHD Purchases 30% (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
    <record id="tax_report_other_taxes" model="account.report">
        <field name="name">4. Other Taxes</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.eg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_other_taxes_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_other_taxes_stamp_tax_base" model="account.report.line">
                <field name="name">Stamp Tax Sales (Base)</field>
                <field name="aggregation_formula">EG_STMP_TW_SB.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_other_taxes_stamp_tax_base_sales" model="account.report.line">
                        <field name="name">Stamp Tax Sales 20% (Base)</field>
                        <field name="code">EG_STMP_TW_SB</field>
                        <field name="expression_ids">
                            <record id="tax_report_other_taxes_stamp_tax_base_sales_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Stamp Tax Sales 20% (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_other_taxes_stamp_tax_tax" model="account.report.line">
                <field name="name">Stamp Tax Sales (Tax)</field>
                <field name="aggregation_formula">EG_STMP_TW_ST.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_other_taxes_stamp_tax_tax_sales" model="account.report.line">
                        <field name="name">Stamp Tax Sales 20% (Tax)</field>
                        <field name="code">EG_STMP_TW_ST</field>
                        <field name="expression_ids">
                            <record id="tax_report_other_taxes_stamp_tax_tax_sales_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Stamp Tax Sales 20% (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_other_taxes_stamp_purchase_tax_base" model="account.report.line">
                <field name="name">Stamp Tax Purchases (Base)</field>
                <field name="aggregation_formula">EG_STMP_TW_PB.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_other_taxes_stamp_purchase_tax_base_purchase" model="account.report.line">
                        <field name="name">Stamp Tax Purchases 20% (Base)</field>
                        <field name="code">EG_STMP_TW_PB</field>
                        <field name="expression_ids">
                            <record id="tax_report_other_taxes_stamp_purchase_tax_base_purchase_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Stamp Tax Purchases 20% (Base)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_other_taxes_stamp_purchase_tax_tax" model="account.report.line">
                <field name="name">Stamp Tax Purchases (Tax)</field>
                <field name="aggregation_formula">EG_STMP_TW_PT.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_other_taxes_stamp_purchase_tax_tax_purchase" model="account.report.line">
                        <field name="name">Stamp Tax Purchases 20% (Tax)</field>
                        <field name="code">EG_STMP_TW_PT</field>
                        <field name="expression_ids">
                            <record id="tax_report_other_taxes_stamp_purchase_tax_tax_purchase_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Stamp Tax Purchases 20% (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-eg.csv

```csv
"id","name","code","account_type","reconcile","name@ar_001"
"egy_account_100101","Right of use Asset (IFRS 16)","100101","asset_fixed","False","حق استخدام الأصول (IFRS 16)"
"egy_account_100102","Accumulated Depreciation right use asset (IFRS 16)","100102","asset_fixed","False","الاستهلاك المتراكم استخدام حق الأصول (IFRS 16)"
"egy_account_100103","VAT Receivable","100103","asset_non_current","False","ضريبة القيمة المضافة المدينة"
"egy_account_101004","Outstanding Receipts","101004","asset_current","False","الوصولات المدفوعة"
"egy_account_101005","Main Safe","101005","asset_current","False","خزينة رئيسية"
"egy_account_101006","Main Safe - Foreign Currency","101006","asset_current","False","خزينة رئيسية - عملات اخرى"
"egy_account_101007","Visa & Master Credit Cards","101007","asset_current","False","بطاقات الائتمان فيزا وماستر"
"egy_account_101008","Gateway Credit Cards","101008","asset_current","False","بطاقات الائتمان Gateway"
"egy_account_101009","Manual Visa & Master Cards","101009","asset_current","False","فيزا وماستر بطاقات"
"egy_account_101010","PayPal Account","101010","asset_current","False","Paypal رصيد"
"egy_account_102011","Accounts Receivable","102011","asset_receivable","True","الذمم المدينة"
"egy_account_102012","Accounts Receivable (PoS)","102012","asset_receivable","True","ذمم مدينة (PoS)"
"egy_account_102013","Post Dated Cheques Received","102013","asset_current","False","شيكات مؤجلة"
"egy_account_102014","Other Receivable","102014","asset_current","False","ذمم مدينة اخرى"
"egy_account_102015","Other Debtors","102015","asset_current","False","مدينون اخرون"
"egy_account_103016","Shipment Insurance","103016","asset_current","False","تأمين الشحن"
"egy_account_103017","Shipments Documentation Charges","103017","asset_current","False","رسوم"
"egy_account_103018","Shipment Other Charges","103018","asset_current","False","رسوم شحنات اخرى"
"egy_account_103019","Handling Difference in Inventory","103019","asset_current","False","فرق المخزون"
"egy_account_103020","Items Delivered to Customs on temprary Base","103020","asset_current","False","بنود في الجمارك"
"egy_account_104021","Prepaid Medical Insurance","104021","asset_current","False","تأمين طبي مدفوع مسبقا"
"egy_account_104022","Prepaid Life Insurance","104022","asset_current","False","تأمين على الحياة مدفوع مسبقا"
"egy_account_104023","Prepaid Office Rent","104023","asset_current","False","ايجار مكتب مدفوع مسبقا"
"egy_account_104024","Prepaid Other Insurance","104024","asset_current","False","تأمينات اخرى مدفوعة مسبقا"
"egy_account_104025","Prepaid License Fees","104025","asset_current","False","رسوم ترخيص مدفوعة مسبقا"
"egy_account_104026","Prepaid Maintenance","104026","asset_current","False","رسوم صيانة مدفوعة مسبقا"
"egy_account_104027","Prepaid Site Hosting Fees","104027","asset_current","False","رسوم استضافة موقع مدفوعة مسبقا"
"egy_account_104028","Prepaid Employees Housing","104028","asset_current","False","بدل سكن للموظفين مدفوع مسبقا"
"egy_account_104029","Prepaid Schooling Fees","104029","asset_current","False","بدل رسوم تعليم مدرسي مدفوع مسبقا"
"egy_account_104030","Prepaid Consultancy Fees","104030","asset_current","False","رسوم استشارات مدفوعة مسبقا"
"egy_account_104031","Prepaid Legal Fees","104031","asset_current","False","الرسوم القانونية مدفوعة مسبقا"
"egy_account_104033","PrePaid Advertisement Expenses","104033","asset_current","False","دعاية و الإعلان مدفوعة مسبقا"
"egy_account_104034","Prepaid Bank Guarantee","104034","asset_current","False","ضمان بنكي مدفوع مسبقا"
"egy_account_104035","Other Prepayments","104035","asset_current","False","دفعات مقدمة أخرى"
"egy_account_104036","Prepaid Finance charge for Loans","104036","asset_current","False","تكاليف تمويل قروض مدفوعة مسبقا"
"egy_account_104037","Deposit - Office Rent","104037","asset_current","False","رسوم تأمين - ايجار مكتبي"
"egy_account_104038","Deposits - Customs","104038","asset_current","False","رسوم تأمين - جمارك"
"egy_account_104040","Deposit Others","104040","asset_current","False","رسوم تأمين - اخرى"
"egy_account_104041","VAT Input","104041","asset_current","False","مدخلات ضريبة القيمة المضافة"
"egy_account_104042","WH tax Advance with Customers - On behalf of my company","104042","asset_current","False","ضريبة خصم المنبع مدفوعة عن طريق العملاء"
"egy_account_105003","Outstanding Payments","105003","asset_current","False","المدفوعات المستحقة"
"egy_account_106001","Leasehold Improvement","106001","asset_current","False","تحسين المستأجرات"
"egy_account_106002","Furniture and Equipment","106002","asset_fixed","False","أثاث و معدات"
"egy_account_106003","Computer Hardware & Software","106003","asset_fixed","False","الكمبيوترات و قطع الغيار و البرمجيات"
"egy_account_106004","Motor Vehicles","106004","asset_fixed","False","السيارات"
"egy_account_106006","Amortisation on Leasehold Improvement","106006","asset_current","False","اطفاء على تحسين المستأجرات"
"egy_account_106007","Acc.Deprn.of Furniture & Office Equipment","106007","asset_current","False","مجمع اهتلاك اثاث و معدات المكتب"
"egy_account_106008","Acc. Deprn.Computer Hardware & Software","106008","asset_current","False","مجمع اهتلاك الكمبيوترات و قطع الغيار و البرمجيات"
"egy_account_106009","Acc. Depreciation of Motor Vehicles","106009","asset_current","False","مجمع اهتلاك السيارات"
"egy_account_106010","Registration of Trademarks","106010","asset_current","False","تسجيل العلامات التجارية"
"egy_account_106011","Computer Card Renewal","106011","asset_current","False","بطاقة تجديد كمبيوتر"
"egy_account_201001","Bank Suspense Account","201001","liability_current","False","حساب البنك المعلق"
"egy_account_201002","Payables","201002","liability_payable","True","الذمم الدائنة"
"egy_account_201003","Credit Notes to Customers","201003","liability_current","False","اشعار دائن للعملاء"
"egy_account_201004","Accrued - Salaries","201004","liability_current","False","الرواتب المستحقة"
"egy_account_201005","Leave Tickets Provision","201005","liability_current","False","مخصص تذاكر"
"egy_account_201006","Leave Days Provision","201006","liability_current","False","مخصص ايام اجازة"
"egy_account_201007","Accrued - Commissions","201007","liability_current","False","عمولة مستحقة"
"egy_account_201008","Accrued Salaries Increment","201008","liability_current","False","راتب اضافي مستحق"
"egy_account_201009","Accrued-Staff Bonus","201009","liability_current","False","مكافأة مستحقة"
"egy_account_201010","Accrued Other Personnel Cost","201010","liability_current","False","تكاليف موظفين مستحقة"
"egy_account_201011","Accrued - Utilities","201011","liability_current","False","فواتير مستحقة"
"egy_account_201012","Accrued - Telephone","201012","liability_current","False","نتكاليف هاتف مستحقة"
"egy_account_201013","Accrued - Sponsorship","201013","liability_current","False","تكفل مستحق"
"egy_account_201014","Accrued - Audit Fees","201014","liability_current","False","اتعاب تدقيق مستحقة"
"egy_account_201015","Accrued - Office Rent","201015","liability_current","False","ايجار مكتب مستحق"
"egy_account_201016","Accrued Others","201016","liability_current","False","اخرى مستحقة"
"egy_account_201017","VAT Output","201017","liability_current","False","مخرجات ضريبة القيمة المضافة"
"egy_account_201018","Deferred income","201018","liability_current","False","الإيرادات مؤجلة"
"egy_account_201020","WHTax Payable - On behalf of suppliers","201020","liability_current","False","ضريبة خصم المنبع للدفع عن الموردين"
"egy_account_201021","Legal Reserve","201021","liability_current","False","احتياطى قانوني"
"egy_account_201022","Taxes Provision","201022","liability_current","False","مخصص ضرائب ورسوم متنازع عليها"
"egy_account_201023","Customer Provision","201023","liability_current","False","مخصص الديون المشكوك في تحصيله"
"egy_account_201024","Schedule Tax collected & payable","201024","liability_current","False","ضريبة جدول دائنة"
"egy_account_201025","Stamp Tax payable","201025","liability_current","False","ضريبة الدمغة دائنة"
"egy_account_201026","Social Contribution - Payable to authorities","201026","liability_current","False","تأمين اجتماعي دائن"
"egy_account_201027","Income Tax payable to Authority - Deducted from employee's salaries","201027","liability_current","False","تأمين اجتماعي دائن - مقتطع من الموظفين"
"egy_account_202001","End of Service Provision","202001","liability_non_current","False","مخصص نهاية الخدمة"
"egy_account_202002","Reservations","202002","liability_non_current","False","احتياطات و حجوزات"
"egy_account_202003","VAT Payable","202003","liability_non_current","False","ضريبة القيمة المضافة المستحقة"
"egy_account_400001","Cost of Goods Sold in Trading","400001","expense_direct_cost","False","تكلفة البضاعة المباعة في التجارة"
"egy_account_400002","Cost Of Goods Sold I/C Sales","400002","expense_direct_cost","False","تكلفة البضاعة المباعة مع المبيعات"
"egy_account_400003","Basic Salary","400003","expense","False","مصروف الراتب الاساسي"
"egy_account_400004","Housing Allowance","400004","expense","False","مصروف بدل سكن"
"egy_account_400005","Transportation Allowance","400005","expense","False","مصروف بدل نقل"
"egy_account_400006","Leave Ticket","400006","expense","False","مصروف تذاكر موظفين"
"egy_account_400007","Leave Salary","400007","expense","False","مصروف اجازة موظفين"
"egy_account_400008","End Of Service Indemnity","400008","expense","False","مصروف نهاية الخدمة"
"egy_account_400009","Medical Insurance","400009","expense","False","مصروف تأمين طبي"
"egy_account_400010","Life Insurance","400010","expense","False","مصروف تأمين على الحياة"
"egy_account_400011","Sales Commission","400011","expense","False","مصروف عمولة مبيعات"
"egy_account_400012","Staff Other Allowances","400012","expense","False","مصروف بدلات اخرى للموظفين"
"egy_account_400013","Uniform","400013","expense","False","مصروف زي موحد"
"egy_account_400014","Visa Expenses","400014","expense","False","مصروف تأشيرة"
"egy_account_400015","Personnel Cost Others","400015","expense","False","مصروف موظفين - اخرى"
"egy_account_400016","Office Rent","400016","expense","False","مصروف اجار مكتب"
"egy_account_400017","Warehouse Rent","400017","expense","False","مصروف ايجار مستودع"
"egy_account_400018","Water & Electricity","400018","expense","False","مصروف مياه و كهرباء"
"egy_account_400019","Other Utility Cahrges","400019","expense","False","مصروف خدمات اخرى"
"egy_account_400020","Telephone","400020","expense","False","مصروف هاتف"
"egy_account_400021","Courrier","400021","expense","False","مصروف شحن"
"egy_account_400022","Web Site Hosting Fees","400022","expense","False","رسوم استضافة موقع"
"egy_account_400023","Others - Communication","400023","expense","False","مصاريف اتصالات اخرى"
"egy_account_400024","Air tickets","400024","expense","False","مصاريف تذاكر طيران"
"egy_account_400025","Hotel","400025","expense","False","مصاريف فندق"
"egy_account_400026","Meals","400026","expense","False","مصاريف فندق"
"egy_account_400027","Per Diem","400027","expense","False","مصاريف يومية"
"egy_account_400028","Others","400028","expense","False","مصاريف اخرى"
"egy_account_400029","Audit Fees","400029","expense","False","مصروف اتعاب تدقيق"
"egy_account_400031","Legal fees","400031","expense","False","مصروف رسوم قانونية"
"egy_account_400032","Trade License Fees","400032","expense","False","مصروف رسوم الرخصة التجارية"
"egy_account_400033","Others - Professional Fees","400033","expense","False","مصروف أخرى الرسوم الفنية"
"egy_account_400034","Other - Advertising Expenses","400034","expense","False","مصروف أخرى مصاريف الإعلان"
"egy_account_400035","Write Off Receivables & Payables","400035","expense","False","مصروف شطب المدين و الذمم الدائنة"
"egy_account_400036","Write Off Inventory","400036","expense","False","مصروف فرق المخزون"
"egy_account_400037","Amortisation of Preoperating Expenses","400037","expense","False","مصروف إطفاء مصاريف"
"egy_account_400038","Cash Shortage","400038","expense","False","مصروف نقص نقدي"
"egy_account_400039","Others - Provision & Write off","400039","expense","False","مخصصات و فروقات اخرى"
"egy_account_400040","Insurance","400040","expense","False","مصروف تأمين"
"egy_account_400041","Training","400041","expense","False","مصروف تدريب"
"egy_account_400042","Maintenance","400042","expense","False","مصروف صيانة"
"egy_account_400043","Security & Guard","400043","expense","False","مصروف حراسة و امن"
"egy_account_400044","Cleaning","400044","expense","False","مصروف تنظيف"
"egy_account_400045","Subscriptions","400045","expense","False","مصروف الاشتراكات"
"egy_account_400046","Gifts & Donations","400046","expense","False","مصروف هدايا و هبات"
"egy_account_400047","Kitchen and Buffet Expenses","400047","expense","False","مصروف المطبخ وبوفيه"
"egy_account_400048","Vehicle Expenses","400048","expense","False","مصروف سيارة"
"egy_account_400049","Convoyance Expenses","400049","expense","False","مصروف نقل اصول"
"egy_account_400050","Others - Office Various Expenses","400050","expense","False","مصاريف مكتب اخرى"
"egy_account_400051","Other Bank Charges","400051","expense","False","مصروف الرسوم المصرفية الأخرى"
"egy_account_400052","Loss On Fixed Assets Disposal","400052","expense","False","مصروف خسارة بيع و تخلص من اصول"
"egy_account_400053","Loss on Difference on Exchange","400053","expense","False","مصروف خسارة على الفرق العملات"
"egy_account_400054","Disposal of Business Branch","400054","expense","False","مصروف وقف فرع من الاعمال"
"egy_account_400055","Income Tax","400055","expense","False","مصروف ضريبة الدخل"
"egy_account_400056","Previous Year Adjustments Account","400056","expense","False","مصروف حساب تسويات السنة السابقة"
"egy_account_400057","Other Non Operating Expenses","400057","expense","False","المصاريف غير التشغيلية"
"egy_account_400058","Credit Card Charges","400058","expense","False","مصروف رسوم بطاقات الائتمان"
"egy_account_400059","Bank Finance & Loan Charges","400059","expense","False","مصروف بنك التمويل والقروض"
"egy_account_400060","Air Miles Card Charges","400060","expense","False","مصروف رسوم بطاقة Air Miles"
"egy_account_400061","Credit Card Swipe Charges","400061","expense","False","مصروف رسوم بطاقات الائتمان"
"egy_account_400062","PayPal Charges","400062","expense","False","Paypal مصروف رسوم"
"egy_account_400063","Amortization on Leasehold Improvement","400063","expense","False","مصروف إطفاء تحسينات المستأجرة"
"egy_account_400064","Depreciation Of Furniture & Office Equipment","400064","expense","False","مصروف الاستهلاك  الأثاث"
"egy_account_400065","Depreciation Of Computer Hard & Soft","400065","expense","False","مصروف الاستهلاك اجهزة الكمبيوتر"
"egy_account_400066","Depreciation Of Motor Vehicles","400066","expense","False","مصروف استهلاك المركبات"
"egy_account_400067","Consultancy Fees","400067","expense","False","رسوم الاستشارات"
"egy_account_400068","Provision for Doubtful Debts","400068","expense","False","مخصص الديون المعدومة"
"egy_account_400069","Closing Account","400069","expense","False","حساب ختامي"
"egy_account_400070","Depreciation on right of use asset (IFRS 16)","400070","expense","False","الاستهلاك في حق الأصول استخدام (IFRS 16)"
"egy_account_400072","Interest Expense","400072","expense","False","مصروف فائدة"
"egy_account_400074","Bad Debts","400074","expense","False","مصروف ديون معدومة"
"egy_account_400075","Schedule Tax Expense","400075","expense","False","مصروف ضريبة الجدول"
"egy_account_400076","WH Tax Expense","400076","expense","False","مصروف ضريبة خصم المنبع"
"egy_account_400077","Stamp tax expense","400077","expense","False","مصروف ضريبة الدمغة"
"egy_account_400078","Social Contibution - Company portion expense","400078","expense","False","مصروف تأمينات اجتماعية - من الشركة"
"egy_account_400079","Cash Discount Loss","400079","expense","False",""
"egy_account_500001","Sales Account","500001","income","False","مبيعات"
"egy_account_500002","Sales of I/C","500002","income","False","مبيعات شركات تابعة"
"egy_account_500003","Management Consultancy Fees","500003","income","False","مكاسب استشارات ادارية"
"egy_account_500004","Sales from Other Region","500004","income","False","مبيعات مناطق اخرى"
"egy_account_500005","Advertising Income","500005","income","False","دخل الإعلانات"
"egy_account_500006","Branding Income","500006","income","False","دخل علامات تجارية"
"egy_account_500007","Space Rental Income","500007","income","False","دخل تأجير"
"egy_account_500008","Service Income","500008","income","False","دخل خدمات"
"egy_account_500009","Interest Revenue","500009","income","False","ايراد فائدة"
"egy_account_500010","Capital Gain","500010","income","False","مكاسب رأس المال"
"egy_account_500011","Gain On Difference Of Exchange","500011","income","False","ربح فرق عملات"
"egy_account_500013","Other Income","500013","income","False","دخول اخرى"
"egy_account_500014","Cash Discount Gain","500014","income_other","False",""
"egy_account_999001","Cash Difference Loss","999001","expense","False","خسارة الفرق النقدي"
"egy_account_999002","Cash Difference Gain","999002","income","False","مكاسب الفرق النقدي"
"egy_account_999999","Undistributed Profits/Losses","999999","equity_unaffected","False","ارباح / خسائر غير موزعة"

```

## File: data\template\account.fiscal.position-eg.csv

```csv
"id","name","sequence","auto_apply","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"account_fiscal_position_egypt","Egypt","19","1","base.eg","",""
"account_fiscal_position_non_egypt","Non-Egypt","20","1","","eg_standard_sale_14","eg_zero_sale_0"
"","","","","","eg_standard_purchase_14","eg_zero_purchase_0"

```

## File: data\template\account.tax-eg.csv

```csv
"id","name","type_tax_use","amount","amount_type","description","invoice_label","l10n_eg_eta_code","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/use_in_tax_closing","description@ar_001","price_include_override"
"eg_standard_sale_14","14%","sale","14.0","percent","","VAT 14%","t1_v009","eg_tax_vat","base","invoice","+1. VAT 14% (Base)","","","قيمة مضافة %14",""
"","","","","","","","","","tax","invoice","+1. VAT 14% (Tax)","egy_account_201017","","",""
"","","","","","","","","","base","refund","-1. VAT 14% (Base)","","","",""
"","","","","","","","","","tax","refund","-1. VAT 14% (Tax)","egy_account_201017","","",""
"eg_standard_purchase_14","14%","purchase","14.0","percent","","VAT 14%","t1_v009","eg_tax_vat","base","invoice","+5. VAT 14% Expenses (Base)","","","قيمة مضافة %14",""
"","","","","","","","","","tax","invoice","+5. VAT 14% Expenses (Tax)","egy_account_104041","","",""
"","","","","","","","","","base","refund","-5. VAT 14% Expenses (Base)","","","",""
"","","","","","","","","","tax","refund","-5. VAT 14% Expenses (Tax)","egy_account_104041","","",""
"eg_zero_sale_0","0%","sale","0.0","percent","","Zero Rated 0%","","eg_tax_group_other","base","invoice","+2. Zero Rated (Base)","","","صفرية %0",""
"","","","","","","","","","tax","invoice","+2. Zero Rated (Tax)","False","","",""
"","","","","","","","","","base","refund","-2. Zero Rated (Base)","","","",""
"","","","","","","","","","tax","refund","-2. Zero Rated (Tax)","False","","",""
"eg_zero_purchase_0","0%","purchase","0.0","percent","","Zero Rated 0%","","eg_tax_group_other","base","invoice","+6. Zero Rated (Base)","","","صفرية %0",""
"","","","","","","","","","tax","invoice","+6. Zero Rated (Tax)","False","","",""
"","","","","","","","","","base","refund","-6. Zero Rated (Base)","","","",""
"","","","","","","","","","tax","refund","-6. Zero Rated (Tax)","False","","",""
"eg_exempt_sale","0% EXEMPT","sale","0.0","percent","","Exempt","t1_v003","eg_tax_group_other","base","invoice","+3. Exempt Sales (Base)","","","معفاة",""
"","","","","","","","","","tax","invoice","+3. Exempt Sales (Tax)","False","","",""
"","","","","","","","","","base","refund","-3. Exempt Sales (Base)","","","",""
"","","","","","","","","","tax","refund","-3. Exempt Sales (Tax)","False","","",""
"eg_exempt_purchase","0% EXEMPT","purchase","0.0","percent","","Exempt","t1_v003","eg_tax_group_other","base","invoice","+7. Exempt Expenses (Base)","","","معفاة",""
"","","","","","","","","","tax","invoice","+7. Exempt Expenses (Tax)","False","","",""
"","","","","","","","","","base","refund","-7. Exempt Expenses (Base)","","","",""
"","","","","","","","","","tax","refund","-7. Exempt Expenses (Tax)","False","","",""
"eg_stamp_tax_20_sale","20%","sale","20.0","percent","","Stamp","t5_st01","eg_tax_group_stamp","base","invoice","+Stamp Tax Sales 20% (Base)","","","الدمغة",""
"","","","","","","","","","tax","invoice","+Stamp Tax Sales 20% (Tax)","egy_account_201025","False","",""
"","","","","","","","","","base","refund","-Stamp Tax Sales 20% (Base)","","","",""
"","","","","","","","","","tax","refund","-Stamp Tax Sales 20% (Tax)","egy_account_201025","False","",""
"eg_stamp_tax_20_purchase","20%","purchase","20.0","percent","","Stamp","t5_st01","eg_tax_group_stamp","base","invoice","+Stamp Tax Purchases 20% (Base)","","","الدمغة",""
"","","","","","","","","","tax","invoice","+Stamp Tax Purchases 20% (Tax)","egy_account_400077","False","",""
"","","","","","","","","","base","refund","-Stamp Tax Purchases 20% (Base)","","","",""
"","","","","","","","","","tax","refund","-Stamp Tax Purchases 20% (Tax)","egy_account_400077","False","",""
"eg_schedule_tax_8_purchase","8% S","purchase","8.0","percent","Schedule 8%","SCHD 8%","t2_tbl01","eg_tax_group_schedule_8","base","invoice","+SCHD Purchases 8% (Base)","","","الجدول %8",""
"","","","","","","","","","tax","invoice","+SCHD Purchases 8% (Tax)","egy_account_400075","False","",""
"","","","","","","","","","base","refund","-SCHD Purchases 8% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Purchases 8% (Tax)","egy_account_400075","False","",""
"eg_schedule_tax_8_sale","8% S","sale","8.0","percent","Schedule 8%","SCHD 8%","t2_tbl01","eg_tax_group_schedule_8","base","invoice","+SCHD Sales 8% (Base)","","","الجدول %8",""
"","","","","","","","","","tax","invoice","+SCHD Sales 8% (Tax)","egy_account_201024","False","",""
"","","","","","","","","","base","refund","-SCHD Sales 8% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Sales 8% (Tax)","egy_account_201024","False","",""
"eg_withholding_1_sale","1% WH","sale","-1.0","percent","Withholding -1%","WH -1%","","eg_tax_group_withholding_1","base","invoice","+WH on Sales -1% (Base)","","","الصناعة و التجارة %1","tax_excluded"
"","","","","","","","","","tax","invoice","-WH Sales -1% (Tax)","egy_account_104042","False","",""
"","","","","","","","","","base","refund","-WH on Sales -1% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Sales -1% (Tax)","egy_account_104042","False","",""
"eg_schedule_tax_10_purchase","10% S","purchase","10.0","percent","Schedule 10%","SCHD 10%","","eg_tax_group_schedule_10","base","invoice","+SCHD Purchases 10% (Base)","","","الجدول %10",""
"","","","","","","","","","tax","invoice","+SCHD Purchases 10% (Tax)","egy_account_400075","False","",""
"","","","","","","","","","base","refund","-SCHD Purchases 10% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Purchases 10% (Tax)","egy_account_400075","False","",""
"eg_withholding_05_sale","0.5% WH","sale","-0.5","percent","Withholding -0.5%","WH -0.5%","","eg_tax_group_withholding_half","base","invoice","+WH Sales -0.5% (Base)","","","الصناعة و التجارة %0.5","tax_excluded"
"","","","","","","","","","tax","invoice","-WH Sales -0.5% (Tax)","egy_account_104042","False","",""
"","","","","","","","","","base","refund","-WH Sales -0.5% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Sales -0.5% (Tax)","egy_account_104042","False","",""
"eg_withholding_05_purchase","0.5% WH","purchase","-0.5","percent","Withholding -0.5%","WH -0.5%","","eg_tax_group_withholding_half","base","invoice","+WH Purchases -0.5% (Base)","","","الصناعة و التجارة %0.5",""
"","","","","","","","","","tax","invoice","-WH Purchases -0.5% (Tax)","egy_account_201020","False","",""
"","","","","","","","","","base","refund","-WH Purchases -0.5% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Purchases -0.5% (Tax)","egy_account_201020","False","",""
"eg_withholding_1_purchase","1% WH","purchase","-1.0","percent","Withholding -1%","WH -1%","","eg_tax_group_withholding_1","base","invoice","+WH Purchases -1% (Base)","","","الصناعة و التجارة %1",""
"","","","","","","","","","tax","invoice","-WH Purchases -1% (Tax)","egy_account_201020","False","",""
"","","","","","","","","","base","refund","-WH Purchases -1% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Purchases -1% (Tax)","egy_account_201020","False","",""
"eg_schedule_tax_10_sale","10% S","sale","10.0","percent","Schedule 10%","SCHD 10%","","eg_tax_group_schedule_10","base","invoice","+SCHD Sales 10% (Base)","","","الجدول %10",""
"","","","","","","","","","tax","invoice","+SCHD Sales 10% (Tax)","egy_account_201024","False","",""
"","","","","","","","","","base","refund","-SCHD Sales 10% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Sales 10% (Tax)","egy_account_201024","False","",""
"eg_withholding_3_sale","3% WH","sale","-3.0","percent","Withholding -3%","WH -3%","t4_w004","eg_tax_group_withholding_3","base","invoice","+WH on Sales -3% (Base)","","","الصناعة و التجارة %3","tax_excluded"
"","","","","","","","","","tax","invoice","-WH Sales -3% (Tax)","egy_account_104042","False","",""
"","","","","","","","","","base","refund","-WH on Sales -3% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Sales -3% (Tax)","egy_account_104042","False","",""
"eg_schedule_tax_1_sale","1% S","sale","1.0","percent","Schedule 1%","SCHD 1%","","eg_tax_group_schedule_1","base","invoice","+SCHD Sales 1% (Base)","","","الجدول %1",""
"","","","","","","","","","tax","invoice","+SCHD Sales 1% (Tax)","egy_account_201024","False","",""
"","","","","","","","","","base","refund","-SCHD Sales 1% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Sales 1% (Tax)","egy_account_201024","False","",""
"eg_withholding_3_purchase","3% WH","purchase","-3.0","percent","Withholding -3%","WH -3%","t4_w004","eg_tax_group_withholding_3","base","invoice","+WH Purchases -3% (Base)","","","الصناعة و التجارة %3",""
"","","","","","","","","","tax","invoice","-WH Purchases -3% (Tax)","egy_account_201020","False","",""
"","","","","","","","","","base","refund","-WH Purchases -3% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Purchases -3% (Tax)","egy_account_201020","False","",""
"eg_schedule_tax_1_purchase","1% S","purchase","1.0","percent","Schedule 1%","SCHD 1%","","eg_tax_group_schedule_1","base","invoice","+SCHD Purchases 1% (Base)","","","الجدول %1",""
"","","","","","","","","","tax","invoice","+SCHD Purchases 1% (Tax)","egy_account_400075","False","",""
"","","","","","","","","","base","refund","-SCHD Purchases 1% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Purchases 1% (Tax)","egy_account_400075","False","",""
"eg_withholding_5_sale","5% WH","sale","-5.0","percent","Withholding -5%","WH -5%","","eg_tax_group_withholding_5","base","invoice","+WH on Sales -5% (Base)","","","الصناعة و التجارة %5","tax_excluded"
"","","","","","","","","","tax","invoice","-WH Sales -5% (Tax)","egy_account_104042","False","",""
"","","","","","","","","","base","refund","-WH on Sales -5% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Sales -5% (Tax)","egy_account_104042","False","",""
"eg_schedule_tax_15_purchase","15% S","purchase","15.0","percent","Schedule 15%","SCHD 15%","","eg_tax_group_schedule_15","base","invoice","+SCHD Purchases 15% (Base)","","","الجدول %15",""
"","","","","","","","","","tax","invoice","+SCHD Purchases 15% (Tax)","egy_account_400075","False","",""
"","","","","","","","","","base","refund","-SCHD Purchases 15% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Purchases 15% (Tax)","egy_account_400075","False","",""
"eg_withholding_5_purchase","5% WH","purchase","-5.0","percent","Withholding -5%","WH -5%","","eg_tax_group_withholding_5","base","invoice","+WH Purchases -5% (Base)","","","الصناعة و التجارة %5",""
"","","","","","","","","","tax","invoice","-WH Purchases -5% (Tax)","egy_account_201020","False","",""
"","","","","","","","","","base","refund","-WH Purchases -5% (Base)","","","",""
"","","","","","","","","","tax","refund","+WH Purchases -5% (Tax)","egy_account_201020","False","",""
"eg_schedule_tax_15_sale","15% S","sale","15.0","percent","Schedule 15%","SCHD 15%","","eg_tax_group_schedule_15","base","invoice","+SCHD Sales 15% (Base)","","","الجدول %15",""
"","","","","","","","","","tax","invoice","+SCHD Sales 15% (Tax)","egy_account_201024","False","",""
"","","","","","","","","","base","refund","-SCHD Sales 15% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Sales 15% (Tax)","egy_account_201024","False","",""
"eg_schedule_tax_30_sale","30% S","sale","30.0","percent","Schedule 30%","SCHD 30%","","eg_tax_group_schedule_30","base","invoice","+SCHD Sales 30% (Base)","","","الجدول %30",""
"","","","","","","","","","tax","invoice","+SCHD Sales 30% (Tax)","egy_account_201024","False","",""
"","","","","","","","","","base","refund","-SCHD Sales 30% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Sales 30% (Tax)","egy_account_201024","False","",""
"eg_schedule_tax_30_purchase","30% S","purchase","30.0","percent","Schedule 30%","SCHD 30%","","eg_tax_group_schedule_30","base","invoice","+SCHD Purchases 30% (Base)","","","الجدول %30",""
"","","","","","","","","","tax","invoice","+SCHD Purchases 30% (Tax)","egy_account_400075","False","",""
"","","","","","","","","","base","refund","-SCHD Purchases 30% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Purchases 30% (Tax)","egy_account_400075","False","",""
"eg_schedule_tax_05_purchase","0.5% S","purchase","0.5","percent","Schedule 0.5%","SCHD 0.5%","","eg_tax_group_schedule_half","base","invoice","+SCHD Purchases 0.5% (Base)","","","الجدول %0.5",""
"","","","","","","","","","tax","invoice","+SCHD Purchases 0.5% (Tax)","egy_account_400075","False","",""
"","","","","","","","","","base","refund","-SCHD Purchases 0.5% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Purchases 0.5% (Tax)","egy_account_400075","False","",""
"eg_schedule_tax_05_sale","0.5% S","sale","0.5","percent","Schedule 0.5%","SCHD 0.5%","","eg_tax_group_schedule_half","base","invoice","+SCHD Sales 0.5% (Base)","","","الجدول %0.5",""
"","","","","","","","","","tax","invoice","+SCHD Sales 0.5% (Tax)","egy_account_201024","False","",""
"","","","","","","","","","base","refund","-SCHD Sales 0.5% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Sales 0.5% (Tax)","egy_account_201024","False","",""
"eg_schedule_tax_5_purchase","5% S","purchase","5.0","percent","Schedule 5%","SCHD 5%","","eg_tax_group_schedule_5","base","invoice","+SCHD Purchases 5% (Base)","","","الجدول %5",""
"","","","","","","","","","tax","invoice","+SCHD Purchases 5% (Tax)","egy_account_400075","False","",""
"","","","","","","","","","base","refund","-SCHD Purchases 5% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Purchases 5% (Tax)","egy_account_400075","False","",""
"eg_schedule_tax_5_sale","5% S","sale","5.0","percent","Schedule 5%","SCHD 5%","","eg_tax_group_schedule_5","base","invoice","+SCHD Sales 5% (Base)","","","الجدول %5",""
"","","","","","","","","","tax","invoice","+SCHD Sales 5% (Tax)","egy_account_201024","False","",""
"","","","","","","","","","base","refund","-SCHD Sales 5% (Base)","","","",""
"","","","","","","","","","tax","refund","-SCHD Sales 5% (Tax)","egy_account_201024","False","",""

```

## File: data\template\account.tax.group-eg.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","preceding_subtotal","name@ar_001"
"eg_tax_vat","VAT 14%","base.eg","egy_account_100103","egy_account_202003","","قيمة مضافة %14"
"eg_tax_group_other","Other Taxes","base.eg","egy_account_100103","egy_account_202003","","ضرائب اخرى"
"eg_tax_group_stamp","Stamp Tax 20%","base.eg","egy_account_100103","egy_account_202003","","ضريبة الدمغة"
"eg_tax_group_withholding_half","Withholding Tax -0.5%","base.eg","egy_account_100103","egy_account_202003","Subtotal W/O WHTax","-ضرائب الصناعة و التجارة %0.5"
"eg_tax_group_withholding_1","Withholding Tax -1%","base.eg","egy_account_100103","egy_account_202003","Subtotal W/O WHTax","-ضرائب الصناعة و التجارة %1"
"eg_tax_group_withholding_3","Withholding Tax -3%","base.eg","egy_account_100103","egy_account_202003","Subtotal W/O WHTax","-ضرائب الصناعة و التجارة %3"
"eg_tax_group_withholding_5","Withholding Tax -5%","base.eg","egy_account_100103","egy_account_202003","Subtotal W/O WHTax","-ضرائب الصناعة و التجارة %5"
"eg_tax_group_schedule_half","Schedule Tax 0.5%","base.eg","egy_account_100103","egy_account_202003","","ضرائب الجدول %0.5"
"eg_tax_group_schedule_1","Schedule Tax 1%","base.eg","egy_account_100103","egy_account_202003","","ضرائب الجدول %1"
"eg_tax_group_schedule_5","Schedule Tax 5%","base.eg","egy_account_100103","egy_account_202003","","ضرائب الجدول %5"
"eg_tax_group_schedule_8","Schedule Tax 8%","base.eg","egy_account_100103","egy_account_202003","","ضرائب الجدول %8"
"eg_tax_group_schedule_10","Schedule Tax 10%","base.eg","egy_account_100103","egy_account_202003","","ضرائب الجدول %10"
"eg_tax_group_schedule_15","Schedule Tax 15%","base.eg","egy_account_100103","egy_account_202003","","ضرائب الجدول %15"
"eg_tax_group_schedule_30","Schedule Tax 30%","base.eg","egy_account_100103","egy_account_202003","","ضرائب الجدول %30"

```

## File: i18n_extra\l10n_eg.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_eg
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 15.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2022-02-18 10:29+0000\n"
"PO-Revision-Date: 2022-02-18 10:29+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_base_fourteen
msgid "1. Standard Rated 14% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_tax_fourteen
msgid "1. Standard Rated 14% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_base_fourteen
msgid "1. VAT 14% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_tax_fourteen
msgid "1. VAT 14% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_vat_return
msgid "1. VAT Return"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_withholding_tax
msgid "2. Withholding Tax"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_base_zero
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_base_zero
msgid "2. Zero Rated (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_tax_zero
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_tax_zero
msgid "2. Zero Rated (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_base_exempt
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_base_exempt
msgid "3. Exempt Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_tax_exempt
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_sale_tax_exempt
msgid "3. Exempt Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_schedule_tax
msgid "3. Schedule Tax"
msgstr ""

#. module: l10n_eg
#: model:account.tax.report,name:l10n_eg.tax_report_other_taxes
msgid "4. Other Taxes"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_base_fourteen
msgid "5. Standard Rated 14% Expenses (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_tax_fourteen
msgid "5. Standard Rated 14% Expenses (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_base_fourteen
msgid "5. VAT 14% Expenses (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_tax_fourteen
msgid "5. VAT 14% Expenses (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_base_zero
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_base_zero
msgid "6. Zero Rated (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_tax_zero
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_tax_zero
msgid "6. Zero Rated (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_base_exempt
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_base_exempt
msgid "7. Exempt Expenses (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_tax_exempt
#: model:account.report.line,tag_name:l10n_eg.tax_report_vat_return_expense_tax_exempt
msgid "7. Exempt Expenses (Tax)"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_tax_report
msgid "Account Tax Report"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields,field_description:l10n_eg.field_account_tax__l10n_eg_eta_code
#: model:ir.model.fields,field_description:l10n_eg.field_account_tax_template__l10n_eg_eta_code
#: model:ir.model.fields,field_description:l10n_eg.field_l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code
msgid "ETA Code (Egypt)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields,field_description:l10n_eg.field_account_tax_report__name
msgid "Name"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields,help:l10n_eg.field_account_tax_report__name
msgid "Name of this tax report"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_net
msgid "Net VAT Due"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_net_3
msgid "Net VAT due (or reclaimed) for the period"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_half
msgid "SCHD Purchases 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_half
msgid "SCHD Purchases 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_one
msgid "SCHD Purchases 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_one
msgid "SCHD Purchases 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_ten
msgid "SCHD Purchases 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_ten
msgid "SCHD Purchases 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_fifteen
msgid "SCHD Purchases 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen
msgid "SCHD Purchases 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_thirty
msgid "SCHD Purchases 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_thirty
msgid "SCHD Purchases 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_five
msgid "SCHD Purchases 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_five
msgid "SCHD Purchases 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_eight
msgid "SCHD Purchases 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_eight
msgid "SCHD Purchases 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_half
msgid "SCHD Sales 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_half
msgid "SCHD Sales 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_one
msgid "SCHD Sales 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_one
msgid "SCHD Sales 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_ten
msgid "SCHD Sales 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_ten
msgid "SCHD Sales 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_fifteen
msgid "SCHD Sales 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_fifteen
msgid "SCHD Sales 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_thirty
msgid "SCHD Sales 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_thirty
msgid "SCHD Sales 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_five
msgid "SCHD Sales 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_five
msgid "SCHD Sales 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_eight
msgid "SCHD Sales 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_eight
msgid "SCHD Sales 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base
msgid "Schedule Tax on Purchases (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax
msgid "Schedule Tax on Purchases (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_half
msgid "Schedule Tax on Purchases 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_half
msgid "Schedule Tax on Purchases 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_one
msgid "Schedule Tax on Purchases 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_one
msgid "Schedule Tax on Purchases 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_ten
msgid "Schedule Tax on Purchases 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_ten
msgid "Schedule Tax on Purchases 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_fifteen
msgid "Schedule Tax on Purchases 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_fifteen
msgid "Schedule Tax on Purchases 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_thirty
msgid "Schedule Tax on Purchases 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_thirty
msgid "Schedule Tax on Purchases 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_five
msgid "Schedule Tax on Purchases 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_five
msgid "Schedule Tax on Purchases 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_base_eight
msgid "Schedule Tax on Purchases 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_purchase_tax_eight
msgid "Schedule Tax on Purchases 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base
msgid "Schedule Tax on Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax
msgid "Schedule Tax on Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_half
msgid "Schedule Tax on Sales 0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_half
msgid "Schedule Tax on Sales 0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_one
msgid "Schedule Tax on Sales 1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_one
msgid "Schedule Tax on Sales 1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_ten
msgid "Schedule Tax on Sales 10% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_ten
msgid "Schedule Tax on Sales 10% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_fifteen
msgid "Schedule Tax on Sales 15% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_fifteen
msgid "Schedule Tax on Sales 15% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_thirty
msgid "Schedule Tax on Sales 30% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_thirty
msgid "Schedule Tax on Sales 30% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_five
msgid "Schedule Tax on Sales 5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_five
msgid "Schedule Tax on Sales 5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_base_eight
msgid "Schedule Tax on Sales 8% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_schedule_tax_schedule_tax_sale_tax_eight
msgid "Schedule Tax on Sales 8% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_base
msgid "Stamp Tax Purchases (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_tax
msgid "Stamp Tax Purchases (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_base_purchase
#: model:account.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_base_purchase
msgid "Stamp Tax Purchases 20% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_tax_purchase
#: model:account.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_purchase_tax_tax_purchase
msgid "Stamp Tax Purchases 20% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_base
msgid "Stamp Tax Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_tax
msgid "Stamp Tax Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_base_sales
#: model:account.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_tax_base_sales
msgid "Stamp Tax Sales 20% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_other_taxes_stamp_tax_tax_sales
#: model:account.report.line,tag_name:l10n_eg.tax_report_other_taxes_stamp_tax_tax_sales
msgid "Stamp Tax Sales 20% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v001
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v001
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v001
msgid "T1 - V001 - Export"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v002
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v002
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v002
msgid "T1 - V002 - Export to free areas and other areas"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v003
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v003
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v003
msgid "T1 - V003 - Exempted good or service"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v004
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v004
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v004
msgid "T1 - V004 - A non-taxable good or service"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v005
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v005
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v005
msgid "T1 - V005 - Exemptions for diplomats, consulates and embassies"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v006
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v006
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v006
msgid "T1 - V006 - Defence and National security Exemptions"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v007
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v007
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v007
msgid "T1 - V007 - Agreements exemptions"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v008
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v008
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v008
msgid "T1 - V008 - Special Exemptios and other reasons"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v009
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v009
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v009
msgid "T1 - V009 - General Item sales"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t1_v010
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t1_v010
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t1_v010
msgid "T1 - V010 - Other Rates"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t10_mn01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t10_mn01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t10_mn01
msgid "T10 - Mn01 - Municipality Fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t10_mn02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t10_mn02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t10_mn02
msgid "T10 - Mn02 - Municipality Fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t11_mi01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t11_mi01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t11_mi01
msgid "T11 - MI01 - Medical insurance fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t11_mi02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t11_mi02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t11_mi02
msgid "T11 - MI02 - Medical insurance fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t12_of01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t12_of01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t12_of01
msgid "T12 - OF01 - Other fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t12_of02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t12_of02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t12_of02
msgid "T12 - OF02 - Other fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t13_st03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t13_st03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t13_st03
msgid "T13 - ST03 - Stamping tax (percentage)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t14_st04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t14_st04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t14_st04
msgid "T14 - ST04 - Stamping Tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t15_ent03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t15_ent03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t15_ent03
msgid "T15 - Ent03 - Entertainment tax (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t15_ent04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t15_ent04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t15_ent04
msgid "T15 - Ent04 - Entertainment tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t16_rd03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t16_rd03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t16_rd03
msgid "T16 - RD03 - Resource development fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t16_rd04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t16_rd04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t16_rd04
msgid "T16 - RD04 - Resource development fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t17_sc03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t17_sc03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t17_sc03
msgid "T17 - SC03 - Service charges (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t17_sc04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t17_sc04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t17_sc04
msgid "T17 - SC04 - Service charges (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t18_mn03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t18_mn03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t18_mn03
msgid "T18 - Mn03 - Municipality Fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t18_mn04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t18_mn04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t18_mn04
msgid "T18 - Mn04 - Municipality Fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t19_mi03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t19_mi03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t19_mi03
msgid "T19 - MI03 - Medical insurance fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t19_mi04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t19_mi04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t19_mi04
msgid "T19 - MI04 - Medical insurance fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t2_tbl01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t2_tbl01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t2_tbl01
msgid "T2 - Tbl01 - Table tax (percentage)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t20_of03
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t20_of03
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t20_of03
msgid "T20 - OF03 - Other fees (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t20_of04
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t20_of04
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t20_of04
msgid "T20 - OF04 - Other fees (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t3_tbl02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t3_tbl02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t3_tbl02
msgid "T3 - Tbl02 - Table tax (Fixed Amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w001
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w001
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w001
msgid "T4 - W001 - Contracting"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w002
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w002
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w002
msgid "T4 - W002 - Supplies"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w003
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w003
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w003
msgid "T4 - W003 - Purachases"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w004
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w004
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w004
msgid "T4 - W004 - Services"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w005
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w005
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w005
msgid ""
"T4 - W005 - Sums paid by the cooperative societies for car transportation to"
" their members"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w006
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w006
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w006
msgid "T4 - W006 - Commissionagency & brokerage"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w007
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w007
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w007
msgid ""
"T4 - W007 - Discounts & grants & additional exceptional incentives(smoke, "
"cement companies)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w008
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w008
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w008
msgid ""
"T4 - W008 - All discounts & grants & commissions (petroleum, "
"telecommunications, other companies)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w009
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w009
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w009
msgid "T4 - W009 - Supporting export subsidies"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w010
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w010
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w010
msgid "T4 - W010 - Professional fees"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w011
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w011
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w011
msgid "T4 - W011 - Commission & brokerage _A_57"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w012
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w012
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w012
msgid "T4 - W012 - Hospitals collecting from doctors"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w013
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w013
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w013
msgid "T4 - W013 - Royalties"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w014
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w014
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w014
msgid "T4 - W014 - Customs clearance"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w015
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w015
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w015
msgid "T4 - W015 - Exemption"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t4_w016
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t4_w016
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t4_w016
msgid "T4 - W016 - advance payments"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t5_st01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t5_st01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t5_st01
msgid "T5 - ST01 - Stamping tax (percentage)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t6_st02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t6_st02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t6_st02
msgid "T6 - ST02 - Stamping Tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t7_ent01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t7_ent01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t7_ent01
msgid "T7 - Ent01 - Entertainment tax (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t7_ent02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t7_ent02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t7_ent02
msgid "T7 - Ent02 - Entertainment tax (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t8_rd01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t8_rd01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t8_rd01
msgid "T8 - RD01 - Resource development fee (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t8_rd02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t8_rd02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t8_rd02
msgid "T8 - RD02 - Resource development fee (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t9_sc01
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t9_sc01
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t9_sc01
msgid "T9 - SC01 - Service charges (rate)"
msgstr ""

#. module: l10n_eg
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax__l10n_eg_eta_code__t9_sc02
#: model:ir.model.fields.selection,name:l10n_eg.selection__account_tax_template__l10n_eg_eta_code__t9_sc02
#: model:ir.model.fields.selection,name:l10n_eg.selection__l10n_eg_eta_account_tax_mixin__l10n_eg_eta_code__t9_sc02
msgid "T9 - SC02 - Service charges (amount)"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_tax
msgid "Tax"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_account_tax_template
msgid "Templates for Taxes"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_net_1
msgid "Total value of due tax for the period"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_net_2
msgid "Total value of recoverable tax for the period"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_base
msgid "VAT on Expenses and all other Inputs (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_expense_tax
msgid "VAT on Expenses and all other Inputs (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_base
msgid "VAT on Sales and all other Outputs (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_vat_return_sale_tax
msgid "VAT on Sales and all other Outputs (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_half
msgid "WH Purchases -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_half
msgid "WH Purchases -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_one
msgid "WH Purchases -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_one
msgid "WH Purchases -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_three
msgid "WH Purchases -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_three
msgid "WH Purchases -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_base_five
msgid "WH Purchases -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_purchase_tax_five
msgid "WH Purchases -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_half
msgid "WH Sales -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_half
msgid "WH Sales -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_one
msgid "WH Sales -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_three
msgid "WH Sales -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_tax_five
msgid "WH Sales -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_one
msgid "WH on Sales -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_three
msgid "WH on Sales -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,tag_name:l10n_eg.tax_report_withholding_tax_sale_base_five
msgid "WH on Sales -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base
msgid "Withholding Tax on Purchases (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax
msgid "Withholding Tax on Purchases (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_half
msgid "Withholding Tax on Purchases -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_half
msgid "Withholding Tax on Purchases -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_one
msgid "Withholding Tax on Purchases -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_one
msgid "Withholding Tax on Purchases -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_three
msgid "Withholding Tax on Purchases -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_three
msgid "Withholding Tax on Purchases -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_base_five
msgid "Withholding Tax on Purchases -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_purchase_tax_five
msgid "Withholding Tax on Purchases -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base
msgid "Withholding Tax on Sales (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax
msgid "Withholding Tax on Sales (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_half
msgid "Withholding Tax on Sales -0.5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_half
msgid "Withholding Tax on Sales -0.5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_one
msgid "Withholding Tax on Sales -1% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_one
msgid "Withholding Tax on Sales -1% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_three
msgid "Withholding Tax on Sales -3% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_three
msgid "Withholding Tax on Sales -3% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_base_five
msgid "Withholding Tax on Sales -5% (Base)"
msgstr ""

#. module: l10n_eg
#: model:account.report.line,name:l10n_eg.tax_report_withholding_tax_sale_tax_five
msgid "Withholding Tax on Sales -5% (Tax)"
msgstr ""

#. module: l10n_eg
#: model:ir.model,name:l10n_eg.model_l10n_eg_eta_account_tax_mixin
msgid "l10n_eg.eta.account.tax.mixin"
msgstr ""

```

## File: models\account_tax.py

```python
from odoo import models, fields


class AccountTax(models.Model):
    _inherit = 'account.tax'
    _description = 'ETA tax codes mixin'

    l10n_eg_eta_code = fields.Selection(
        selection=[
            ('t1_v001', 'T1 - V001 - Export'),
            ('t1_v002', 'T1 - V002 - Export to free areas and other areas'),
            ('t1_v003', 'T1 - V003 - Exempted good or service'),
            ('t1_v004', 'T1 - V004 - A non-taxable good or service'),
            ('t1_v005', 'T1 - V005 - Exemptions for diplomats, consulates and embassies'),
            ('t1_v006', 'T1 - V006 - Defence and National security Exemptions'),
            ('t1_v007', 'T1 - V007 - Agreements exemptions'),
            ('t1_v008', 'T1 - V008 - Special Exemption and other reasons'),
            ('t1_v009', 'T1 - V009 - General Item sales'),
            ('t1_v010', 'T1 - V010 - Other Rates'),
            ('t2_tbl01', 'T2 - Tbl01 - Table tax (percentage)'),
            ('t3_tbl02', 'T3 - Tbl02 - Table tax (Fixed Amount)'),
            ('t4_w001', 'T4 - W001 - Contracting'),
            ('t4_w002', 'T4 - W002 - Supplies'),
            ('t4_w003', 'T4 - W003 - Purchases'),
            ('t4_w004', 'T4 - W004 - Services'),
            ('t4_w005', 'T4 - W005 - Sums paid by the cooperative societies for car transportation to their members'),
            ('t4_w006', 'T4 - W006 - Commission agency & brokerage'),
            ('t4_w007', 'T4 - W007 - Discounts & grants & additional exceptional incentives (smoke, cement companies)'),
            ('t4_w008', 'T4 - W008 - All discounts & grants & commissions (petroleum, telecommunications, and other)'),
            ('t4_w009', 'T4 - W009 - Supporting export subsidies'),
            ('t4_w010', 'T4 - W010 - Professional fees'),
            ('t4_w011', 'T4 - W011 - Commission & brokerage _A_57'),
            ('t4_w012', 'T4 - W012 - Hospitals collecting from doctors'),
            ('t4_w013', 'T4 - W013 - Royalties'),
            ('t4_w014', 'T4 - W014 - Customs clearance'),
            ('t4_w015', 'T4 - W015 - Exemption'),
            ('t4_w016', 'T4 - W016 - advance payments'),
            ('t5_st01', 'T5 - ST01 - Stamping tax (percentage)'),
            ('t6_st02', 'T6 - ST02 - Stamping Tax (amount)'),
            ('t7_ent01', 'T7 - Ent01 - Entertainment tax (rate)'),
            ('t7_ent02', 'T7 - Ent02 - Entertainment tax (amount)'),
            ('t8_rd01', 'T8 - RD01 - Resource development fee (rate)'),
            ('t8_rd02', 'T8 - RD02 - Resource development fee (amount)'),
            ('t9_sc01', 'T9 - SC01 - Service charges (rate)'),
            ('t9_sc02', 'T9 - SC02 - Service charges (amount)'),
            ('t10_mn01', 'T10 - Mn01 - Municipality Fees (rate)'),
            ('t10_mn02', 'T10 - Mn02 - Municipality Fees (amount)'),
            ('t11_mi01', 'T11 - MI01 - Medical insurance fee (rate)'),
            ('t11_mi02', 'T11 - MI02 - Medical insurance fee (amount)'),
            ('t12_of01', 'T12 - OF01 - Other fees (rate)'),
            ('t12_of02', 'T12 - OF02 - Other fees (amount)'),
            ('t13_st03', 'T13 - ST03 - Stamping tax (percentage)'),
            ('t14_st04', 'T14 - ST04 - Stamping Tax (amount)'),
            ('t15_ent03', 'T15 - Ent03 - Entertainment tax (rate)'),
            ('t15_ent04', 'T15 - Ent04 - Entertainment tax (amount)'),
            ('t16_rd03', 'T16 - RD03 - Resource development fee (rate)'),
            ('t16_rd04', 'T16 - RD04 - Resource development fee (amount)'),
            ('t17_sc03', 'T17 - SC03 - Service charges (rate)'),
            ('t17_sc04', 'T17 - SC04 - Service charges (amount)'),
            ('t18_mn03', 'T18 - Mn03 - Municipality Fees (rate)'),
            ('t18_mn04', 'T18 - Mn04 - Municipality Fees (amount)'),
            ('t19_mi03', 'T19 - MI03 - Medical insurance fee (rate)'),
            ('t19_mi04', 'T19 - MI04 - Medical insurance fee (amount)'),
            ('t20_of03', 'T20 - OF03 - Other fees (rate)'),
            ('t20_of04', 'T20 - OF04 - Other fees (amount)')
        ],
        string='ETA Code (Egypt)', default=False)

```

## File: models\template_eg.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('eg')
    def _get_eg_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'egy_account_102011',
            'property_account_payable_id': 'egy_account_201002',
            'property_account_expense_categ_id': 'egy_account_400028',
            'property_account_income_categ_id': 'egy_account_500001',
            }

    @template('eg', 'res.company')
    def _get_eg_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.eg',
                'bank_account_code_prefix': '101',
                'cash_account_code_prefix': '105',
                'transfer_account_code_prefix': '100',
                'account_default_pos_receivable_account_id': 'egy_account_102012',
                'income_currency_exchange_account_id': 'egy_account_500011',
                'expense_currency_exchange_account_id': 'egy_account_400053',
                'account_journal_suspense_account_id': 'egy_account_201001',
                'account_journal_early_pay_discount_loss_account_id': 'egy_account_400079',
                'account_journal_early_pay_discount_gain_account_id': 'egy_account_500014',
                'default_cash_difference_income_account_id': 'egy_account_999002',
                'default_cash_difference_expense_account_id': 'egy_account_999001',
                'account_sale_tax_id': 'eg_standard_sale_14',
                'account_purchase_tax_id': 'eg_standard_purchase_14',
            },
        }

    @template('eg', 'account.journal')
    def _get_eg_account_journal(self):
        """ If EGYPT chart, we add 2 new journals TA and IFRS"""
        return {
            "tax_adjustment": {
                "name": "Tax Adjustments",
                "code": "TA",
                "type": "general",
                "sequence": 1,
                "show_on_dashboard": True,
            },
            "ifrs": {
                "name": "IFRS 16",
                "code": "IFRS",
                "type": "general",
                "show_on_dashboard": True,
                "sequence": 10,
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_eg
from . import account_tax

```

## File: views\account_tax.xml

```xml
<data>
    <record id="view_account_tax_form" model="ir.ui.view">
        <field name="name">account.tax.form</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <field name="tax_scope" position="after">
                <field name="l10n_eg_eta_code" invisible="country_code != 'EG'"/>
            </field>
        </field>
    </record>

    <record id="view_tax_eta_code_tree" model="ir.ui.view">
        <field name="name">account.tax.eta.code.list</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_tree" />
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="l10n_eg_eta_code" optional="hide"/>
            </field>
        </field>
    </record>
</data>

```

