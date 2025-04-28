# Odoo Module: l10n_ae

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
    'name': 'United Arab Emirates - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/united_arab_emirates.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ae'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
United Arab Emirates Accounting Module
=======================================================
United Arab Emirates accounting basic charts and localization.

Activates:

- Chart of Accounts
- Taxes
- Tax Report
- Fiscal Positions
    """,
    'depends': [
        'base',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/l10n_ae_data.xml',
        'data/account_tax_report_data.xml',
        'data/res.bank.csv',
        'views/report_invoice_templates.xml',
        'views/account_move.xml',
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
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ae"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_base_all_sales" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Base)</field>
                <field name="aggregation_formula">1_STANDARD_RATED_SUPPLIES_BASE.balance + TAX_REF_TOUR_SCHEME_BASE.balance + REVERSE_CHARGE_PRO_BASE.balance + ZERO_RATE_SUPP_BASE.balance + EXAMPT_SUPP_BASE.balance + OUT_OF_SCOPE_BASE_0.balance + GOODS_IMPORT_IN_UAE_BASE.balance + ADJUST_GOODS_IMPORT_IN_UAE_BASE.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_supplies_base" model="account.report.line">
                        <field name="name">1. Standard Rated supplies (Base)</field>
                        <field name="code">1_STANDARD_RATED_SUPPLIES_BASE</field>
                        <field name="aggregation_formula">STD_RATE_SUPP_BASE_AB.balance + STD_RATE_SUPP_BASE_DB.balance + STD_RATE_SUPP_BASE_SJ.balance + STD_RATE_SUPP_BASE_AJ.balance + STD_RATE_SUPP_BASE_UM.balance + STD_RATE_SUPP_BASE_RA.balance + STD_RATE_SUPP_BASE_FU.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_standard_rated_supplies_base_abu_dhabi" model="account.report.line">
                                <field name="name">a. Abu Dhabi</field>
                                <field name="code">STD_RATE_SUPP_BASE_AB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_abu_dhabi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">a. Abu Dhabi (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_dubai" model="account.report.line">
                                <field name="name">b. Dubai</field>
                                <field name="code">STD_RATE_SUPP_BASE_DB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_dubai_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">b. Dubai (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_sharjah" model="account.report.line">
                                <field name="name">c. Sharjah</field>
                                <field name="code">STD_RATE_SUPP_BASE_SJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_sharjah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">c. Sharjah (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_ajman" model="account.report.line">
                                <field name="name">d. Ajman</field>
                                <field name="code">STD_RATE_SUPP_BASE_AJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_ajman_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">d. Ajman (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_umm_al_quwain" model="account.report.line">
                                <field name="name">e. Umm Al Quwain</field>
                                <field name="code">STD_RATE_SUPP_BASE_UM</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_umm_al_quwain_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">e. Umm Al Quwain (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_ras_al_khaima" model="account.report.line">
                                <field name="name">f. Ras Al-Khaima</field>
                                <field name="code">STD_RATE_SUPP_BASE_RA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_ras_al_khaima_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">f. Ras Al-Khaima (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_fujairah" model="account.report.line">
                                <field name="name">g. Fujairah</field>
                                <field name="code">STD_RATE_SUPP_BASE_FU</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_base_fujairah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">g. Fujairah (Base)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_base_subtotal" model="account.report.line">
                                <field name="name">Sub Total</field>
                                <field name="aggregation_formula">STD_RATE_SUPP_BASE_AB.balance + STD_RATE_SUPP_BASE_DB.balance + STD_RATE_SUPP_BASE_SJ.balance + STD_RATE_SUPP_BASE_AJ.balance + STD_RATE_SUPP_BASE_UM.balance + STD_RATE_SUPP_BASE_RA.balance + STD_RATE_SUPP_BASE_FU.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_tax_refund_tourist_base" model="account.report.line">
                        <field name="name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme</field>
                        <field name="code">TAX_REF_TOUR_SCHEME_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_tax_refund_tourist_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_reverse_charge_base" model="account.report.line">
                        <field name="name">3. Supplies subject to reverse charge provisions</field>
                        <field name="code">REVERSE_CHARGE_PRO_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_reverse_charge_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Supplies subject to reverse charge provisions (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_zero_rated_supplies_base" model="account.report.line">
                        <field name="name">4. Zero rated supplies</field>
                        <field name="code">ZERO_RATE_SUPP_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_zero_rated_supplies_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4. Zero rated supplies (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_supplies_base" model="account.report.line">
                        <field name="name">5. Exempt supplies</field>
                        <field name="code">EXAMPT_SUPP_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_supplies_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. Exempt supplies (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_out_of_scope_base" model="account.report.line">
                        <field name="name">6. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_BASE_0</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_out_of_scope_base_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_import_uae_base" model="account.report.line">
                        <field name="name">7. Goods imported into the UAE</field>
                        <field name="code">GOODS_IMPORT_IN_UAE_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_import_uae_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Goods imported into the UAE (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_adjustment_import_uae_base" model="account.report.line">
                        <field name="name">8. Adjustments to goods imported into the UAE</field>
                        <field name="code">ADJUST_GOODS_IMPORT_IN_UAE_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_adjustment_import_uae_base_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_base_all_sales_total" model="account.report.line">
                        <field name="name">9. Total</field>
                        <field name="aggregation_formula">ADJUST_GOODS_IMPORT_IN_UAE_BASE.balance + GOODS_IMPORT_IN_UAE_BASE.balance + OUT_OF_SCOPE_BASE_0.balance + EXAMPT_SUPP_BASE.balance + ZERO_RATE_SUPP_BASE.balance + REVERSE_CHARGE_PRO_BASE.balance + TAX_REF_TOUR_SCHEME_BASE.balance + (STD_RATE_SUPP_BASE_AB.balance + STD_RATE_SUPP_BASE_DB.balance + STD_RATE_SUPP_BASE_SJ.balance + STD_RATE_SUPP_BASE_AJ.balance + STD_RATE_SUPP_BASE_UM.balance + STD_RATE_SUPP_BASE_RA.balance + STD_RATE_SUPP_BASE_FU.balance)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_base_all_expense" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Base)</field>
                <field name="aggregation_formula">STD_RATE_EXPENSES_BASE.balance + SUPP_REV_CHARGE_PRO_BASE.balance + OUT_OF_SCOPE_1_BASE.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_expense_base" model="account.report.line">
                        <field name="name">10. Standard rated expenses</field>
                        <field name="code">STD_RATE_EXPENSES_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_expense_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10. Standard rated expenses (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_supplies_reverse_base" model="account.report.line">
                        <field name="name">11. Supplies subject to the reverse charge provisions</field>
                        <field name="code">SUPP_REV_CHARGE_PRO_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_supplies_reverse_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11. Supplies subject to the reverse charge provisions (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_out_of_scope" model="account.report.line">
                        <field name="name">12. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_1_BASE</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_out_of_scope_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_base_all_expense_total" model="account.report.line">
                        <field name="name">13. Totals</field>
                        <field name="aggregation_formula">OUT_OF_SCOPE_1_BASE.balance + SUPP_REV_CHARGE_PRO_BASE.balance + STD_RATE_EXPENSES_BASE.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_all_sales" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Tax)</field>
                <field name="aggregation_formula">1_STANDARD_RATED_SUPPLIES_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + ZERO_RATE_SUPP_TAX.balance + EXAMPT_SUPP_TAX.balance + OUT_OF_SCOPE_TAX_0.balance + GOODS_IMPORT_IN_UAE_TAX.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_supplies_vat" model="account.report.line">
                        <field name="name">1. Standard Rated supplies (Tax)</field>
                        <field name="code">1_STANDARD_RATED_SUPPLIES_TAX</field>
                        <field name="aggregation_formula">STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_standard_rated_supplies_vat_abu_dhabi" model="account.report.line">
                                <field name="name">a. Abu Dhabi</field>
                                <field name="code">STD_RATE_SUPP_TAX_AB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_abu_dhabi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">a. Abu Dhabi (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_dubai" model="account.report.line">
                                <field name="name">b. Dubai</field>
                                <field name="code">STD_RATE_SUPP_TAX_DB</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_dubai_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">b. Dubai (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_sharjah" model="account.report.line">
                                <field name="name">c. Sharjah</field>
                                <field name="code">STD_RATE_SUPP_TAX_SJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_sharjah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">c. Sharjah (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_ajman" model="account.report.line">
                                <field name="name">d. Ajman</field>
                                <field name="code">STD_RATE_SUPP_TAX_AJ</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_ajman_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">d. Ajman (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_umm_al_quwain" model="account.report.line">
                                <field name="name">e. Umm Al Quwain</field>
                                <field name="code">STD_RATE_SUPP_TAX_UM</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_umm_al_quwain_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">e. Umm Al Quwain (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_ras_al_khaima" model="account.report.line">
                                <field name="name">f. Ras Al-Khaima</field>
                                <field name="code">STD_RATE_SUPP_TAX_RA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_ras_al_khaima_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">f. Ras Al-Khaima (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_fujairah" model="account.report.line">
                                <field name="name">g. Fujairah</field>
                                <field name="code">STD_RATE_SUPP_TAX_FU</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_standard_rated_supplies_vat_fujairah_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">g. Fujairah (Tax)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_standard_rated_supplies_vat_subtotal" model="account.report.line">
                                <field name="name">Sub Total</field>
                                <field name="aggregation_formula">STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_tax_refund_tourist_vat" model="account.report.line">
                        <field name="name">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme</field>
                        <field name="code">TAX_REF_TOUR_SCHEME_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_tax_refund_tourist_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_reverse_charge_vat" model="account.report.line">
                        <field name="name">3. Supplies subject to reverse charge provisions</field>
                        <field name="code">REVERSE_CHARGE_PRO_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_reverse_charge_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Supplies subject to reverse charge provisions (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_zero_rated_supplies_vat" model="account.report.line">
                        <field name="name">4. Zero rated supplies</field>
                        <field name="code">ZERO_RATE_SUPP_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_zero_rated_supplies_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4. Zero rated supplies (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_supplies_vat" model="account.report.line">
                        <field name="name">5. Exempt supplies</field>
                        <field name="code">EXAMPT_SUPP_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_supplies_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. Exempt supplies (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_supplies_out_of_scope_vat" model="account.report.line">
                        <field name="name">6. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_TAX_0</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_supplies_out_of_scope_vat_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_import_uae_vat" model="account.report.line">
                        <field name="name">7. Goods imported into the UAE</field>
                        <field name="code">GOODS_IMPORT_IN_UAE_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_import_uae_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Goods imported into the UAE (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_adjustment_import_uae_vat" model="account.report.line">
                        <field name="name">8. Adjustments to goods imported into the UAE</field>
                        <field name="code">ADJUST_GOODS_IMPORT_IN_UAE_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_adjustment_import_uae_vat_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vat_all_sales_total" model="account.report.line">
                        <field name="name">9. Total</field>
                        <field name="aggregation_formula">(STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_all_expense" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Tax)</field>
                <field name="aggregation_formula">STD_RATE_EXPENSES_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + OUT_OF_SCOPE_1_TAX.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_expense_vat" model="account.report.line">
                        <field name="name">10. Standard rated expenses</field>
                        <field name="code">STD_RATE_EXPENSES_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_expense_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10. Standard rated expenses (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_supplies_reverse_vat" model="account.report.line">
                        <field name="name">11. Supplies subject to the reverse charge provisions</field>
                        <field name="code">SUPP_REV_CHARGE_PRO_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_supplies_reverse_vat_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11. Supplies subject to the reverse charge provisions (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_expense_out_of_scope_vat" model="account.report.line">
                        <field name="name">12. Out of scope</field>
                        <field name="code">OUT_OF_SCOPE_1_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_expense_out_of_scope_vat_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vat_all_expense_total" model="account.report.line">
                        <field name="name">13. Totals</field>
                        <field name="aggregation_formula">OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_net_vat_due" model="account.report.line">
                <field name="name">Net VAT Due</field>
                <field name="aggregation_formula">((STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance) - (OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_total_value_due_tax_period" model="account.report.line">
                        <field name="name">14. Total value of due tax for the period</field>
                        <field name="aggregation_formula">(STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance</field>
                    </record>
                    <record id="tax_report_line_total_value_recoverable_tax_period" model="account.report.line">
                        <field name="name">15. Total value of recoverable tax for the period</field>
                        <field name="aggregation_formula">OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance</field>
                    </record>
                    <record id="tax_report_line_net_vat_due_period" model="account.report.line">
                        <field name="name">16. Net VAT due (or reclaimed) for the period</field>
                        <field name="aggregation_formula">((STD_RATE_SUPP_TAX_AB.balance + STD_RATE_SUPP_TAX_DB.balance + STD_RATE_SUPP_TAX_SJ.balance + STD_RATE_SUPP_TAX_AJ.balance + STD_RATE_SUPP_TAX_UM.balance + STD_RATE_SUPP_TAX_RA.balance + STD_RATE_SUPP_TAX_FU.balance) + OUT_OF_SCOPE_TAX_0.balance + ADJUST_GOODS_IMPORT_IN_UAE_TAX.balance + GOODS_IMPORT_IN_UAE_TAX.balance + EXAMPT_SUPP_TAX.balance + ZERO_RATE_SUPP_TAX.balance + REVERSE_CHARGE_PRO_TAX.balance + TAX_REF_TOUR_SCHEME_TAX.balance) - (OUT_OF_SCOPE_1_TAX.balance + SUPP_REV_CHARGE_PRO_TAX.balance + STD_RATE_EXPENSES_TAX.balance)</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_ae_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- set VAT label to show on invoice report -->
    <record id="base.ae" model="res.country">
        <field name="vat_label">VAT</field>
    </record>
    <record id="base.AED" model="res.currency">
        <field name="symbol">AED</field>
    </record>
    <record id="gcc_countries_group" model="res.country.group">
        <field name="name">GCC VAT implementing States</field>
        <field name="country_ids" eval="[(6,0,[ref('base.ae'),ref('base.sa'),ref('base.bh')])]"/>
    </record>
</odoo>

```

## File: data\res.bank.csv

```csv
id,name,country
l10n_ae_bank_1,"ROYAL BANK OF SCOTLAND",United Arab Emirates
l10n_ae_bank_2,"NAT. BANK OF SHJ",United Arab Emirates
l10n_ae_bank_3,"ABUDHABI COMML.BANK",United Arab Emirates
l10n_ae_bank_4,"NAT. BANK OF UAQ",United Arab Emirates
l10n_ae_bank_5,"AL AHLI BANK OF KUWAIT",United Arab Emirates
l10n_ae_bank_6,"STANDARD CHARTERED",United Arab Emirates
l10n_ae_bank_7,"Al RAFIDIAN",United Arab Emirates
l10n_ae_bank_8,"UNION NATIONAL BANK",United Arab Emirates
l10n_ae_bank_9,"ARAB AFRICAN INT’L",United Arab Emirates
l10n_ae_bank_10,"UNITED ARAB BANK",United Arab Emirates
l10n_ae_bank_11,"ARBIFT",United Arab Emirates
l10n_ae_bank_12,"UNITED BANK LTD.",United Arab Emirates
l10n_ae_bank_13,"ARAB BANK",United Arab Emirates
l10n_ae_bank_14,"ARAB EMIRAES INVESTMENT BK",United Arab Emirates
l10n_ae_bank_15,"BANK MELLI IRAN",United Arab Emirates
l10n_ae_bank_16,"DEUTSCHE BK",United Arab Emirates
l10n_ae_bank_17,"BANK OF BARODA",United Arab Emirates
l10n_ae_bank_18,"ABU DHABI ISLAMIC BK",United Arab Emirates
l10n_ae_bank_19,"BANK OF SHARJAH",United Arab Emirates
l10n_ae_bank_20,"DUBAI BANK PJSC",United Arab Emirates
l10n_ae_bank_21,"BANK OF SADERAT IRAN",United Arab Emirates
l10n_ae_bank_22,"NOOR ISLAMIC BK",United Arab Emirates
l10n_ae_bank_23,"BANQUE BANORABE",United Arab Emirates
l10n_ae_bank_24,"Al HILAL BK",United Arab Emirates
l10n_ae_bank_25,"BANK MISR",United Arab Emirates
l10n_ae_bank_26,"DOHA BANK",United Arab Emirates
l10n_ae_bank_27,"CREDIT AGRICOLE",United Arab Emirates
l10n_ae_bank_28,"SAMBA",United Arab Emirates
l10n_ae_bank_29,"BANQUE LIBAINAISE",United Arab Emirates
l10n_ae_bank_30,"NAT. BK OF KUWAIT",United Arab Emirates
l10n_ae_bank_31,"BANQUE PARIBAS",United Arab Emirates
l10n_ae_bank_32,"AJMAN BK",United Arab Emirates
l10n_ae_bank_33,"BARCLAYS BANK",United Arab Emirates
l10n_ae_bank_34,"HABIB BANK ZURICH",United Arab Emirates
l10n_ae_bank_35,"HSBC",United Arab Emirates
l10n_ae_bank_36,"Abdul Latif Exchange - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_37,"CITI BANK",United Arab Emirates
l10n_ae_bank_38,"Ahmed Al Amery Exchange Est-Abu Dhabi",United Arab Emirates
l10n_ae_bank_39,"COMML. BANK INTL.",United Arab Emirates
l10n_ae_bank_40,"Ahmed Al Hussain Exchange Est. - Dubai",United Arab Emirates
l10n_ae_bank_41,"COMM. BANK OF DUBAI",United Arab Emirates
l10n_ae_bank_42,"Ain Al Faydah Exchange-Al Ain",United Arab Emirates
l10n_ae_bank_43,"DUBAI ISLAMIC BANK",United Arab Emirates
l10n_ae_bank_44,"Al Ahalia Money Exchange BureauAbu Dhabi",United Arab Emirates
l10n_ae_bank_45,"EL NILIEN BANK",United Arab Emirates
l10n_ae_bank_46,"Al Ansari Exchange-Abu Dhabi",United Arab Emirates
l10n_ae_bank_47,"NAT. BANK OF DUBAI",United Arab Emirates
l10n_ae_bank_48,"Al Ansari Exchange Services -Al Ain",United Arab Emirates
l10n_ae_bank_49,"FIRST GULF BANK",United Arab Emirates
l10n_ae_bank_50,"Al Azhar Exchange- Dubai",United Arab Emirates
l10n_ae_bank_51,"HABIB BANK LTD.",United Arab Emirates
l10n_ae_bank_52,"Al Bader Exchange-Abu Dhabi",United Arab Emirates
l10n_ae_bank_53,"INVEST BANK",United Arab Emirates
l10n_ae_bank_54,"Al Balooch Money Exchange- Al Ain",United Arab Emirates
l10n_ae_bank_55,"JANATA BANK",United Arab Emirates
l10n_ae_bank_56,"Al Dahab Exchange, Dubai",United Arab Emirates
l10n_ae_bank_57,"LLOYDS BANK",United Arab Emirates
l10n_ae_bank_58,"Al Darmaki Exchange Est. - Dubai",United Arab Emirates
l10n_ae_bank_59,"MASHREQ BANK",United Arab Emirates
l10n_ae_bank_60,"Al Dhahery Money Exchange- Al Ain",United Arab Emirates
l10n_ae_bank_61,"EMIRATES ISLAMIC BANK",United Arab Emirates
l10n_ae_bank_62,"Al Falah Exchange Company-Abu Dhabi",United Arab Emirates
l10n_ae_bank_63,"NAT. BANK OF ABU DHABI",United Arab Emirates
l10n_ae_bank_64,"Al Fardan Exchange-Abu Dhabi",United Arab Emirates
l10n_ae_bank_65,"NAT. BANK OF BAHRAIN",United Arab Emirates
l10n_ae_bank_66,"Al Fuad Exchange - Dubai",United Arab Emirates
l10n_ae_bank_67,"Al Gergawi Exchange - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_68,"NAT.BANK OF FUJAIRAH",United Arab Emirates
l10n_ae_bank_69,"Al Ghurair Exchange- Dubai",United Arab Emirates
l10n_ae_bank_70,"NAT. BANK OF OMAN",United Arab Emirates
l10n_ae_bank_71,"Al Ghurair International ExchangeDubai",United Arab Emirates
l10n_ae_bank_72,"RAK BANK",United Arab Emirates
l10n_ae_bank_73,"Al Hadha Exchange LLC. - Dubai",United Arab Emirates
l10n_ae_bank_74,"Al Hamed Exchange - Sharjah",United Arab Emirates
l10n_ae_bank_75,"Habib Exchange Co. LLC. - Sharjah",United Arab Emirates
l10n_ae_bank_76,"Al Hamriyah Exchange- Dubai",United Arab Emirates
l10n_ae_bank_77,"Hadi Express Exchange- Dubai",United Arab Emirates
l10n_ae_bank_78,"Al Jarwan Money Exchange-Sharjah",United Arab Emirates
l10n_ae_bank_79,"Harib Sultan Exchange-Abu Dhabi",United Arab Emirates
l10n_ae_bank_80,"Al Masood Exchange-Abu Dhabi",United Arab Emirates
l10n_ae_bank_81,"Horizon Exchange - Dubai",United Arab Emirates
l10n_ae_bank_82,"Al Mazroui Exchange Est-Abu Dhabi",United Arab Emirates
l10n_ae_bank_83,"International Developmentn Exchange-Dubai",United Arab Emirates
l10n_ae_bank_84,"Al Modawallah Exchange- Dubai",United Arab Emirates
l10n_ae_bank_85,"Jumana Exchange Est. - Dubai",United Arab Emirates
l10n_ae_bank_86,"Al Mona Exchange Co. - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_87,"Kanoo Exchange- Dubai",United Arab Emirates
l10n_ae_bank_88,"Al Mussabah Exchange- Dubai",United Arab Emirates
l10n_ae_bank_89,"Khalil Al Fardan Exchange- Dubai",United Arab Emirates
l10n_ae_bank_90,"Al Nafees Exchange LLC. - Dubai",United Arab Emirates
l10n_ae_bank_91,"Khalili Exchange Co. LLC. - Dubai",United Arab Emirates
l10n_ae_bank_92,"Al Ne'emah Exchange Co. LLC- Dubai",United Arab Emirates
l10n_ae_bank_93,"Lari Exchange Est-Abu Dhabi",United Arab Emirates
l10n_ae_bank_94,"Al Rajihi Exchange Company - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_95,"Lee La Megh Exchange LLC- Dubai",United Arab Emirates
l10n_ae_bank_96,"Al Razouki Int'l Exchange Co. LLC. - Dub",United Arab Emirates
l10n_ae_bank_97,"Malik Exchange-Abu Dhabi",United Arab Emirates
l10n_ae_bank_98,"Al Zari & Al Fardan Exchange LLC. - Sharjah",United Arab Emirates
l10n_ae_bank_99,"Multinet Trust Exchange - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_100,"Al Zarooni Exchange- Dubai",United Arab Emirates
l10n_ae_bank_101,"Nanikdas Nathoomal Exchange Co. LLC- Dub",United Arab Emirates
l10n_ae_bank_102,"Alukkass Exchange, Dubai",United Arab Emirates
l10n_ae_bank_103,"Naser Khoory Exchange Est. - Abu Dhabi",United Arab Emirates
l10n_ae_bank_104,"Arabian Exchange Co-Abu Dhabi",United Arab Emirates
l10n_ae_bank_105,"National Exchange Co.-Abu Dhabi",United Arab Emirates
l10n_ae_bank_106,"ARY International Exchange- Dubai",United Arab Emirates
l10n_ae_bank_107,"Oasis Exchange, Al Ain",United Arab Emirates
l10n_ae_bank_108,"Asia Exchange Centre- Dubai",United Arab Emirates
l10n_ae_bank_109,"Orient Exchange Co.- LLC- Dubai",United Arab Emirates
l10n_ae_bank_110,"Aziz Exchange Co. LLC.- Dubai",United Arab Emirates
l10n_ae_bank_111,"Pacific Exchange- Dubai",United Arab Emirates
l10n_ae_bank_112,"Bhagwandas Jethanand and SonsSharjah",United Arab Emirates
l10n_ae_bank_113,"Redha Al Ansari Exchange Est. - Dubai",United Arab Emirates
l10n_ae_bank_114,"Bin Bakheet Exchange Est.- Ajman",United Arab Emirates
l10n_ae_bank_115,"Reems Exchange- Dubai",United Arab Emirates
l10n_ae_bank_116,"Bin Belaila Exchange Co.-LLC. - Dubai",United Arab Emirates
l10n_ae_bank_117,"Sa'ad Exchange-Fujairah",United Arab Emirates
l10n_ae_bank_118,"Cash Express Exchange Est. - Dubai",United Arab Emirates
l10n_ae_bank_119,"Sabah Exchange-Sharjah",United Arab Emirates
l10n_ae_bank_120,"Central Exchange LLC Dubai",United Arab Emirates
l10n_ae_bank_121,"Sajwani Exchange- Dubai",United Arab Emirates
l10n_ae_bank_122,"City Exchange - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_123,"Salim Exchange-Sharjah",United Arab Emirates
l10n_ae_bank_124,"Daniba International Exchange- Dubai",United Arab Emirates
l10n_ae_bank_125,"Sana'a Exchange- Dubai",United Arab Emirates
l10n_ae_bank_126,"Day Exchange LLC.- Dubai",United Arab Emirates
l10n_ae_bank_127,"Sawan Exchange Co. - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_128,"Dinar Exchange - Dubai",United Arab Emirates
l10n_ae_bank_129,"Shaheen Money Exchange LLC. - Dubai",United Arab Emirates
l10n_ae_bank_130,"Dubai Exchange Centre - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_131,"Sharjah International ExchangeSharjah",United Arab Emirates
l10n_ae_bank_132,"Dubai Express Exchange, Dubai",United Arab Emirates
l10n_ae_bank_133,"Tabra & Al Nebal Exchange - Dubai",United Arab Emirates
l10n_ae_bank_134,"Emirates & East India Exchange - Sharjah",United Arab Emirates
l10n_ae_bank_135,"Tahir Exchange Est. - Dubai",United Arab Emirates
l10n_ae_bank_136,"Emirates India International Exchange-Sharjah",United Arab Emirates
l10n_ae_bank_137,"Taymour & Abou Harb Exchange Co. LLC. -Sharjah",United Arab Emirates
l10n_ae_bank_138,"Federal Exchange- Dubai",United Arab Emirates
l10n_ae_bank_139,"Al Rostamani Exchange- Dubai",United Arab Emirates
l10n_ae_bank_140,"First Gulf Exchange Centre- Dubai",United Arab Emirates
l10n_ae_bank_141,"U.A.E. Exchange Centre - LLC.- Dubai",United Arab Emirates
l10n_ae_bank_142,"Gomti Exchange - LLC. - Dubai",United Arab Emirates
l10n_ae_bank_143,"Union Exchange-Abu Dhabi",United Arab Emirates
l10n_ae_bank_144,"Gulf Express Exchange- Dubai",United Arab Emirates
l10n_ae_bank_145,"Universal Exchange Centre - Dubai",United Arab Emirates
l10n_ae_bank_146,"Gulf Int'l Exchange Co.-LLC. - Dubai",United Arab Emirates
l10n_ae_bank_147,"Wall Street Exchange Centre - LLCDubai",United Arab Emirates
l10n_ae_bank_148,"Zahra Al Yousuf Exchange- Dubai",United Arab Emirates
l10n_ae_bank_149,"AL DINAR EXCHANGE COMPANY",United Arab Emirates
l10n_ae_bank_150,"Global Exchange",United Arab Emirates
l10n_ae_bank_151,"HSBC Financial Services",United Arab Emirates
l10n_ae_bank_152,"Economic Exchange",United Arab Emirates
l10n_ae_bank_153,"ALFA EXCHANGE",United Arab Emirates
l10n_ae_bank_154,"Royal Exchange Co. LLC",United Arab Emirates
l10n_ae_bank_155,"DELMA EXCHANGE",United Arab Emirates
l10n_ae_bank_156,"Bin Belaila Exchange Co. L.L.C",United Arab Emirates
l10n_ae_bank_157,"SHARAF EXCHANGE",United Arab Emirates
l10n_ae_bank_158,"Alfalah Exchange Company",United Arab Emirates
l10n_ae_bank_159,"LULU EXCHANGE",United Arab Emirates
l10n_ae_bank_160,"GCC EXCHANGE",United Arab Emirates
l10n_ae_bank_161,"C3",United Arab Emirates
l10n_ae_bank_162,"Economic Exchange Centre",United Arab Emirates
l10n_ae_bank_163,"Western Union",United Arab Emirates
l10n_ae_bank_164,"Global Exchange",United Arab Emirates
l10n_ae_bank_165,"Waseela Equity",United Arab Emirates
l10n_ae_bank_166,"Zareen Exchange",United Arab Emirates
l10n_ae_bank_167,"Workers Equity Holdings",United Arab Emirates
l10n_ae_bank_168,"Future Exchange",United Arab Emirates
l10n_ae_bank_169,"SEIDCO",United Arab Emirates
l10n_ae_bank_170,"Nasim Al Barari Exchange",United Arab Emirates
l10n_ae_bank_171,"Finance House",United Arab Emirates
l10n_ae_bank_172,"Al Dhahery Exchange",United Arab Emirates
l10n_ae_bank_173,"Dunia",United Arab Emirates
l10n_ae_bank_174,"Al Nibal International Exchange",United Arab Emirates

```

## File: data\template\account.account-ae.csv

```csv
"id","name","code","account_type","reconcile"
"uae_account_100101","Right of use Asset (IFRS 16)","100101","asset_fixed","False"
"uae_account_100102","Accumulated Depreciation right use asset (IFRS 16)","100102","asset_fixed","False"
"uae_account_100103","VAT Receivable","100103","asset_non_current","False"
"uae_account_101005","Main Safe","101005","asset_current","False"
"uae_account_101006","Main Safe - Foreign Currency","101006","asset_current","False"
"uae_account_101007","Visa & Master Credit Cards","101007","asset_current","False"
"uae_account_101008","Gateway Credit Cards","101008","asset_current","False"
"uae_account_101009","Manual Visa & Master Cards","101009","asset_current","False"
"uae_account_101010","PayPal Account","101010","asset_current","False"
"uae_account_102011","Accounts Receivable","102011","asset_receivable","True"
"uae_account_102012","Accounts Receivable (PoS)","102012","asset_receivable","True"
"uae_account_102013","Post Dated Cheques Received","102013","asset_current","False"
"uae_account_102014","Other Receivable","102014","asset_current","False"
"uae_account_102015","Other Debtors","102015","asset_current","False"
"uae_account_103016","Shipment Insurance","103016","asset_current","False"
"uae_account_103017","Shipments Documentation Charges","103017","asset_current","False"
"uae_account_103018","Shipment Other Charges","103018","asset_current","False"
"uae_account_103019","Handling Difference in Inventory","103019","asset_current","False"
"uae_account_103020","Items Delivered to Customs on temprary Base","103020","asset_current","False"
"uae_account_104021","Prepaid Medical Insurance","104021","asset_current","False"
"uae_account_104022","Prepaid Life Insurance","104022","asset_current","False"
"uae_account_104023","Prepaid Office Rent","104023","asset_current","False"
"uae_account_104024","Prepaid Other Insurance","104024","asset_current","False"
"uae_account_104025","Prepaid License Fees","104025","asset_current","False"
"uae_account_104026","Prepaid Maintenance","104026","asset_current","False"
"uae_account_104027","Prepaid Site Hosting Fees","104027","asset_current","False"
"uae_account_104028","Prepaid Employees Housing","104028","asset_current","False"
"uae_account_104029","Prepaid Schooling Fees","104029","asset_current","False"
"uae_account_104030","Prepaid Consultancy Fees","104030","asset_current","False"
"uae_account_104031","Prepaid Legal Fees","104031","asset_current","False"
"uae_account_104032","Prepaid Sponsorship Fees","104032","asset_current","False"
"uae_account_104033","PrePaid Advertisement Expenses","104033","asset_current","False"
"uae_account_104034","Prepaid Bank Guarantee","104034","asset_current","False"
"uae_account_104035","Other Prepayments","104035","asset_current","False"
"uae_account_104036","Prepaid Finance charge for Loans","104036","asset_current","False"
"uae_account_104037","Deposit - Office Rent","104037","asset_current","False"
"uae_account_104038","Deposits - Customs","104038","asset_current","False"
"uae_account_104039","Deposit to Immigration (Visa)","104039","asset_current","False"
"uae_account_104040","Deposit Others","104040","asset_current","False"
"uae_account_104041","VAT Input","104041","asset_current","False"
"uae_account_106001","Leasehold Improvement","106001","asset_current","False"
"uae_account_106002","Furniture and Equipment","106002","asset_current","False"
"uae_account_106003","Computer Hardware & Software","106003","asset_current","False"
"uae_account_106004","Motor Vehicles","106004","asset_current","False"
"uae_account_106005","Work In Progrees","106005","asset_current","False"
"uae_account_106006","Amortisation on Leasehold Improvement","106006","asset_current","False"
"uae_account_106007","Acc.Deprn.of Furniture & Office Equipment","106007","asset_current","False"
"uae_account_106008","Acc. Deprn.Computer Hardware & Software","106008","asset_current","False"
"uae_account_106009","Acc. Depreciation of Motor Vehicles","106009","asset_current","False"
"uae_account_106010","Registration of Trademarks","106010","asset_current","False"
"uae_account_106011","Computer Card Renewal","106011","asset_current","False"
"uae_account_128000","Prepaid Expenses","128000","asset_current","False"
"uae_account_201002","Payables","201002","liability_payable","True"
"uae_account_201003","Credit Notes to Customers","201003","liability_current","False"
"uae_account_201004","Accrued - Salaries","201004","liability_current","False"
"uae_account_201005","Leave Tickets Provision","201005","liability_current","False"
"uae_account_201006","Leave Days Provision","201006","liability_current","False"
"uae_account_201007","Accrued - Commissions","201007","liability_current","False"
"uae_account_201008","Accrued Salaries Increment","201008","liability_current","False"
"uae_account_201009","Accrued-Staff Bonus","201009","liability_current","False"
"uae_account_201010","Accrued Other Personnel Cost","201010","liability_current","False"
"uae_account_201011","Accrued - Utilities","201011","liability_current","False"
"uae_account_201012","Accrued - Telephone","201012","liability_current","False"
"uae_account_201013","Accrued - Sponsorship","201013","liability_current","False"
"uae_account_201014","Accrued - Audit Fees","201014","liability_current","False"
"uae_account_201015","Accrued - Office Rent","201015","liability_current","False"
"uae_account_201016","Accrued Others","201016","liability_current","False"
"uae_account_201017","VAT Output","201017","liability_current","False"
"uae_account_201018","Deferred income","201018","liability_current","False"
"uae_account_201019","Accrued Dubai Customs","201019","liability_current","False"
"uae_account_202001","End of Service Provision","202001","liability_non_current","False"
"uae_account_202002","Reservations","202002","liability_non_current","False"
"uae_account_202003","VAT Payable","202003","liability_non_current","False"
"uae_account_212000","Deferred Revenues - Other","212000","liability_current","False"
"uae_account_400001","Cost of Goods Sold in Trading","400001","expense_direct_cost","False"
"uae_account_400002","Cost Of Goods Sold I/C Sales","400002","expense_direct_cost","False"
"uae_account_400003","Basic Salary","400003","expense","False"
"uae_account_400004","Housing Allowance","400004","expense","False"
"uae_account_400005","Transportation Allowance","400005","expense","False"
"uae_account_400006","Leave Ticket","400006","expense","False"
"uae_account_400007","Leave Salary","400007","expense","False"
"uae_account_400008","End Of Service Indemnity","400008","expense","False"
"uae_account_400009","Medical Insurance","400009","expense","False"
"uae_account_400010","Life Insurance","400010","expense","False"
"uae_account_400011","Sales Commission","400011","expense","False"
"uae_account_400012","Staff Other Allowances","400012","expense","False"
"uae_account_400013","Uniform","400013","expense","False"
"uae_account_400014","Visa Expenses","400014","expense","False"
"uae_account_400015","Personnel Cost Others","400015","expense","False"
"uae_account_400016","Office Rent","400016","expense","False"
"uae_account_400017","Warehouse Rent","400017","expense","False"
"uae_account_400018","Water & Electricity","400018","expense","False"
"uae_account_400019","Other Utility Cahrges","400019","expense","False"
"uae_account_400020","Telephone","400020","expense","False"
"uae_account_400021","Courrier","400021","expense","False"
"uae_account_400022","Web Site Hosting Fees","400022","expense","False"
"uae_account_400023","Others - Communication","400023","expense","False"
"uae_account_400024","Air tickets","400024","expense","False"
"uae_account_400025","Hotel","400025","expense","False"
"uae_account_400026","Meals","400026","expense","False"
"uae_account_400027","Per Diem","400027","expense","False"
"uae_account_400028","Others","400028","expense","False"
"uae_account_400029","Audit Fees","400029","expense","False"
"uae_account_400030","Sponsorship Fees","400030","expense","False"
"uae_account_400031","Legal fees","400031","expense","False"
"uae_account_400032","Trade License Fees","400032","expense","False"
"uae_account_400033","Others - Professional Fees","400033","expense","False"
"uae_account_400034","Other - Advertising Expenses","400034","expense","False"
"uae_account_400035","Write Off Receivables & Payables","400035","expense","False"
"uae_account_400036","Write Off Inventory","400036","expense","False"
"uae_account_400037","Amortisation of Preoperating Expenses","400037","expense","False"
"uae_account_400038","Cash Shortage","400038","expense","False"
"uae_account_400039","Others - Provision & Write off","400039","expense","False"
"uae_account_400040","Insurance","400040","expense","False"
"uae_account_400041","Training","400041","expense","False"
"uae_account_400042","Maintenance","400042","expense","False"
"uae_account_400043","Security & Guard","400043","expense","False"
"uae_account_400044","Cleaning","400044","expense","False"
"uae_account_400045","Subscriptions","400045","expense","False"
"uae_account_400046","Gifts & Donations","400046","expense","False"
"uae_account_400047","Kitchen and Buffet Expenses","400047","expense","False"
"uae_account_400048","Vehicle Expenses","400048","expense","False"
"uae_account_400049","Convoyance Expenses","400049","expense","False"
"uae_account_400050","Others - Office Various Expenses","400050","expense","False"
"uae_account_400051","Other Bank Charges","400051","expense","False"
"uae_account_400052","Loss On Fixed Assets Disposal","400052","expense","False"
"uae_account_400053","Loss on Difference on Exchange","400053","expense","False"
"uae_account_400054","Disposal of Business Branch","400054","expense","False"
"uae_account_400055","Income Tax","400055","expense","False"
"uae_account_400056","Previous Year Adjustments Account","400056","expense","False"
"uae_account_400057","Other Non Operating Expenses","400057","expense","False"
"uae_account_400058","Credit Card Charges","400058","expense","False"
"uae_account_400059","Bank Finance & Loan Charges","400059","expense","False"
"uae_account_400060","Air Miles Card Charges","400060","expense","False"
"uae_account_400061","Credit Card Swipe Charges","400061","expense","False"
"uae_account_400062","PayPal Charges","400062","expense","False"
"uae_account_400063","Amortization on Leasehold Improvement","400063","expense","False"
"uae_account_400064","Depreciation Of Furniture & Office Equipment","400064","expense","False"
"uae_account_400065","Depreciation Of Computer Hard & Soft","400065","expense","False"
"uae_account_400066","Depreciation Of Motor Vehicles","400066","expense","False"
"uae_account_400067","Consultancy Fees","400067","expense","False"
"uae_account_400068","Provision for Doubtful Debts","400068","expense","False"
"uae_account_400069","Closing Account","400069","expense","False"
"uae_account_400070","Depreciation on right of use asset (IFRS 16)","400070","expense","False"
"uae_account_400071","Cash Discount Loss","400071","expense","False"
"uae_account_500001","Sales Account","500001","income","False"
"uae_account_500002","Sales of I/C","500002","income","False"
"uae_account_500003","Management Consultancy Fees","500003","income","False"
"uae_account_500004","Sales from Other Region","500004","income","False"
"uae_account_500005","Advertising Income","500005","income","False"
"uae_account_500006","Branding Income","500006","income","False"
"uae_account_500007","Space Rental Income","500007","income","False"
"uae_account_500008","Service Income","500008","income","False"
"uae_account_500009","Interest Revenue","500009","income","False"
"uae_account_500010","Capital Gain","500010","income","False"
"uae_account_500011","Gain On Difference Of Exchange","500011","income","False"
"uae_account_500012","Excess In Till","500012","income","False"
"uae_account_500013","Other Income","500013","income","False"
"uae_account_500014","Cash Discount Gain","500014","income_other","False"
"uae_account_999999","Undistributed Profits/Losses","999999","equity_unaffected","False"

```

## File: data\template\account.fiscal.position-ae.csv

```csv
"id","name","auto_apply","sequence","country_id","state_ids","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"account_fiscal_position_dubai","Dubai","1","16","base.ae","base.state_ae_du","",""
"account_fiscal_position_abu_dhabi","Abu Dhabi","1","16","base.ae","base.state_ae_az","uae_sale_tax_5_dubai","uae_sale_tax_5_abu_dhabi"
"account_fiscal_position_sharjah","Sharjah","1","16","base.ae","base.state_ae_sh","uae_sale_tax_5_dubai","uae_sale_tax_5_sharjah"
"account_fiscal_position_ajman","Ajman","1","16","base.ae","base.state_ae_aj","uae_sale_tax_5_dubai","uae_sale_tax_5_ajman"
"account_fiscal_position_umm_al_quwain","Umm Al Quwain","1","16","base.ae","base.state_ae_uq","uae_sale_tax_5_dubai","uae_sale_tax_5_umm_al_quwain"
"account_fiscal_position_ras_al_khaima","Ras Al-Khaima","1","16","base.ae","base.state_ae_rk","uae_sale_tax_5_dubai","uae_sale_tax_5_ras_al_khaima"
"account_fiscal_position_fujairah","Fujairah","1","16","base.ae","base.state_ae_fu","uae_sale_tax_5_dubai","uae_sale_tax_5_fujairah"
"account_fiscal_position_non_uae_countries","Non-UAE","1","20","","","uae_sale_tax_5_dubai","uae_sale_tax_0"
"","","","","","","uae_purchase_tax_5","uae_purchase_tax_reverse_charge"

```

## File: data\template\account.tax-ae.csv

```csv
"id","name","type_tax_use","amount","amount_type","description","invoice_label","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent"
"uae_sale_tax_5_dubai","5% DB","sale","5.0","percent","Vat 5% Dubai","VAT 5%","ae_tax_group_5","base","invoice","+b. Dubai (Base)","",""
"","","","","","","","","tax","invoice","+b. Dubai (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-b. Dubai (Base)","",""
"","","","","","","","","tax","refund","-b. Dubai (Tax)","uae_account_201017",""
"uae_sale_tax_5_abu_dhabi","5% AD","sale","5.0","percent","VAT 5% Abu Dhabi","VAT 5%","ae_tax_group_5","base","invoice","+a. Abu Dhabi (Base)","",""
"","","","","","","","","tax","invoice","+a. Abu Dhabi (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-a. Abu Dhabi (Base)","",""
"","","","","","","","","tax","refund","-a. Abu Dhabi (Tax)","uae_account_201017",""
"uae_sale_tax_5_sharjah","5% S","sale","5.0","percent","VAT 5% Sharjah","VAT 5%","ae_tax_group_5","base","invoice","+c. Sharjah (Base)","",""
"","","","","","","","","tax","invoice","+c. Sharjah (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-c. Sharjah (Base)","",""
"","","","","","","","","tax","refund","-c. Sharjah (Tax)","uae_account_201017",""
"uae_sale_tax_5_ajman","5% A","sale","5.0","percent","VAT 5% Ajman","VAT 5%","ae_tax_group_5","base","invoice","+d. Ajman (Base)","",""
"","","","","","","","","tax","invoice","+d. Ajman (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-d. Ajman (Base)","",""
"","","","","","","","","tax","refund","-d. Ajman (Tax)","uae_account_201017",""
"uae_sale_tax_5_umm_al_quwain","5% UAQ","sale","5.0","percent","VAT 5% Umm Al Quwain","VAT 5%","ae_tax_group_5","base","invoice","+e. Umm Al Quwain (Base)","",""
"","","","","","","","","tax","invoice","+e. Umm Al Quwain (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-e. Umm Al Quwain (Base)","",""
"","","","","","","","","tax","refund","-e. Umm Al Quwain (Tax)","uae_account_201017",""
"uae_sale_tax_5_ras_al_khaima","5% RAK","sale","5.0","percent","VAT 5% Ras Al-Khaima","VAT 5%","ae_tax_group_5","base","invoice","+f. Ras Al-Khaima (Base)","",""
"","","","","","","","","tax","invoice","+f. Ras Al-Khaima (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-f. Ras Al-Khaima (Base)","",""
"","","","","","","","","tax","refund","-f. Ras Al-Khaima (Tax)","uae_account_201017",""
"uae_sale_tax_5_fujairah","5% F","sale","5.0","percent","VAT 5% Fujairah","VAT 5%","ae_tax_group_5","base","invoice","+g. Fujairah (Base)","",""
"","","","","","","","","tax","invoice","+g. Fujairah (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-g. Fujairah (Base)","",""
"","","","","","","","","tax","refund","-g. Fujairah (Tax)","uae_account_201017",""
"uae_sale_tax_exempted","0% Exempt","sale","0.0","percent","Exempted","Exempted","ae_tax_group_exempted","base","invoice","+5. Exempt supplies (Base)","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-5. Exempt supplies (Base)","",""
"","","","","","","","","tax","refund","","",""
"uae_sale_tax_0","0%","sale","0.0","percent","","VAT 0%","ae_tax_group_0","base","invoice","+4. Zero rated supplies (Base)","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-4. Zero rated supplies (Base)","",""
"","","","","","","","","tax","refund","","",""
"uae_export_tax","0% EX","sale","0.0","percent","Export Tax","Export Tax","","base","invoice","","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","","",""
"","","","","","","","","tax","refund","","",""
"uae_sale_tax_reverse_charge_dubai","5% R C D","sale","5.0","percent","Supplies subject to reverse charge provisions (Dubai)","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"uae_sale_tax_reverse_charge_abu_dhabi","5% R C AD","sale","5.0","percent","Supplies subject to reverse charge provisions (Abu Dhabi)","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"uae_sale_tax_reverse_charge_sharjah","5% R C S","sale","5.0","percent","Supplies subject to reverse charge provisions (Sharjah)","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"uae_sale_tax_reverse_charge_ajman","5% R C A","sale","5.0","percent","Supplies subject to reverse charge provisions (Ajman)","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"uae_sale_tax_reverse_charge_umm_al_quwain","5% R C UAQ","sale","5.0","percent","Supplies subject to reverse charge provisions (Umm Al Quwain)","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"uae_sale_tax_reverse_charge_ras_al_khaima","5% R C RAK","sale","5.0","percent","Supplies subject to reverse charge provisions (Ras Al-Khaima)","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"uae_sale_tax_reverse_charge_fujairah","5% R C F","sale","5.0","percent","Supplies subject to reverse charge provisions (Fujairah)","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"uae_sale_tax_tourist_refund","5% T R","sale","5.0","percent","Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme","Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme","","base","invoice","+2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Base)","",""
"","","","","","","","","tax","invoice","+2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Tax)","uae_account_201017",""
"","","","","","","","","base","refund","-2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Base)","",""
"","","","","","","","","tax","refund","-2. Tax Refunds provided to Tourists under the Tax Refunds for Tourists Scheme (Tax)","uae_account_201017",""
"uae_purchase_tax_5","5%","purchase","5.0","percent","","VAT 5%","ae_tax_group_5","base","invoice","+10. Standard rated expenses (Base)","",""
"","","","","","","","","tax","invoice","+10. Standard rated expenses (Tax)","uae_account_104041",""
"","","","","","","","","base","refund","-10. Standard rated expenses (Base)","",""
"","","","","","","","","tax","refund","-10. Standard rated expenses (Tax)","uae_account_104041",""
"uae_purchase_tax_exempted","0% Exempt","purchase","0.0","percent","Exempted Tax","Exempted Tax","ae_tax_group_exempted","base","invoice","","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","","",""
"","","","","","","","","tax","refund","","",""
"uae_purchase_tax_0","0%","purchase","0.0","percent","","VAT 0%","ae_tax_group_0","base","invoice","","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","","",""
"","","","","","","","","tax","refund","","",""
"uae_import_tax","5% EX","purchase","5.0","percent","Import Tax","Import Tax","","base","invoice","+7. Goods imported into the UAE (Base)","",""
"","","","","","","","","tax","invoice","+7. Goods imported into the UAE (Tax)","uae_account_104041",""
"","","","","","","","","base","refund","-7. Goods imported into the UAE (Base)","",""
"","","","","","","","","tax","refund","-7. Goods imported into the UAE (Tax)","uae_account_104041",""
"uae_purchase_tax_reverse_charge","5% R C","purchase","5.0","percent","Supplies subject to reverse charge provisions","Supplies subject to reverse charge provisions","","base","invoice","+11. Supplies subject to the reverse charge provisions (Base)||+3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","invoice","+11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","invoice","-3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"
"","","","","","","","","base","refund","-11. Supplies subject to the reverse charge provisions (Base)||-3. Supplies subject to reverse charge provisions (Base)","",""
"","","","","","","","","tax","refund","-11. Supplies subject to the reverse charge provisions (Tax)","uae_account_104041",""
"","","","","","","","","tax","refund","+3. Supplies subject to reverse charge provisions (Tax)","uae_account_201017","-100"

```

## File: data\template\account.tax.group-ae.csv

```csv
"id","country_id","name","tax_receivable_account_id","tax_payable_account_id"
"ae_tax_group_5","base.ae","VAT 5%","uae_account_100103","uae_account_202003"
"ae_tax_group_0","base.ae","VAT 0%","uae_account_100103","uae_account_202003"
"ae_tax_group_exempted","base.ae","VAT Exempted","uae_account_100103","uae_account_202003"

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    l10n_ae_vat_amount = fields.Monetary(compute='_compute_vat_amount', string='VAT Amount')

    @api.depends('price_subtotal', 'price_total')
    def _compute_vat_amount(self):
        for record in self:
            record.l10n_ae_vat_amount = record.price_total - record.price_subtotal

```

## File: models\template_ae.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, Command
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ae')
    def _get_ae_template_data(self):
        return {
            'property_account_receivable_id': 'uae_account_102011',
            'property_account_payable_id': 'uae_account_201002',
            'property_account_expense_categ_id': 'uae_account_400001',
            'property_account_income_categ_id': 'uae_account_500001',
            'code_digits': '6',
        }

    @template('ae', 'res.company')
    def _get_ae_res_company(self):
        sales_tax_xmlid = {
            'AZ': 'uae_sale_tax_5_abu_dhabi',
            'AJ': 'uae_sale_tax_5_ajman',
            'DU': 'uae_sale_tax_5_dubai',
            'FU': 'uae_sale_tax_5_fujairah',
            'RK': 'uae_sale_tax_5_ras_al_khaima',
            'SH': 'uae_sale_tax_5_sharjah',
            'UQ': 'uae_sale_tax_5_umm_al_quwain',
        }.get(self.env.company.state_id.code, 'uae_sale_tax_5_abu_dhabi')
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ae',
                'bank_account_code_prefix': '101',
                'cash_account_code_prefix': '105',
                'transfer_account_code_prefix': '100',
                'account_default_pos_receivable_account_id': 'uae_account_102012',
                'income_currency_exchange_account_id': 'uae_account_500011',
                'expense_currency_exchange_account_id': 'uae_account_400053',
                'account_journal_early_pay_discount_loss_account_id': 'uae_account_400071',
                'account_journal_early_pay_discount_gain_account_id': 'uae_account_500014',
                'account_sale_tax_id': sales_tax_xmlid,
                'account_purchase_tax_id': 'uae_purchase_tax_5',
            },
        }

    @template('ae', 'account.journal')
    def _get_ae_account_journal(self):
        """ If UAE chart, we add 2 new journals TA and IFRS"""
        return {
            "tax_adjustment":{
                "name": "Tax Adjustments",
                "code": "TA",
                "type": "general",
                "show_on_dashboard": True,
                "sequence": 1,
            },
            "ifrs16": {
                "name": "IFRS 16",
                "code": "IFRS",
                "type": "general",
                "show_on_dashboard": True,
                "sequence": 10,
            }
        }

    @template('ae', 'account.account')
    def _get_ae_account_account(self):
        return {
            "uae_account_100101": {
                'allowed_journal_ids': [Command.link('ifrs16')],
            },
            "uae_account_100102": {
                'allowed_journal_ids': [Command.link('ifrs16')],
            },
            "uae_account_400070": {
                'allowed_journal_ids': [Command.link('ifrs16')],
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ae
from . import account_move_line

```

## File: views\account_move.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record id="view_move_form" model="ir.ui.view">
            <field name="name">l10n_ae.account.move.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//list//field[@name='tax_ids']" position="after">
                    <field name="l10n_ae_vat_amount" optional="show" column_invisible="parent.country_code != 'AE'"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\report_invoice_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//span[@name='payment_term']" position="after">
            <p t-if="o.company_id.country_id.code == 'AE' and o.partner_id.country_id.code != 'AE' and o.env.ref('l10n_ae.gcc_countries_group') in o.partner_id.country_id.country_group_ids">
                Supply between <b>United Arab Emirates</b> and
                <b>
                    <span t-field="o.partner_id.country_id.name"/>
                </b>
            </p>
        </xpath>
        <xpath expr="//t[@t-set='layout_document_title']/span" position="before">
            <span t-if="o.company_id.country_id.code == 'AE' and o.move_type in ['out_invoice', 'out_refund']">TAX
            </span>
        </xpath>

        <xpath expr="//thead//th[@name='th_taxes']" position="replace">
            <th name="th_taxes"
                t-attf-class="text-start {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                <span t-if="o.company_id.country_id.code == 'AE'">VAT</span>
                <span t-else="">Taxes</span>
            </th>
            <th t-if="o.company_id.country_id.code == 'AE'" name="tax_amount"
                t-attf-class="text-start {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                <span>VAT Amount</span>
            </th>
        </xpath>

        <xpath expr="//span[@id='line_tax_ids']/.." position="after">
            <td t-if="o.company_id.country_id.code == 'AE'">
                <span t-field="line.l10n_ae_vat_amount" id="line_tax_amount"/>
            </td>
        </xpath>
    </template>

    <template id="document_tax_totals_company_currency_template" inherit_id="account.document_tax_totals_company_currency_template">
        <xpath expr="//p[hasclass('tax_computation_company_currency')]" position="after">
            <tr t-if="o.company_id.country_id.code == 'AE'">
                <t t-set="exchange_rate"
                   t-value="abs(o.amount_total_signed) / o.amount_total"/>
                <td>Exchange Rate</td>
                <td class="text-end" t-out="exchange_rate" t-options='{"widget": "float", "precision": 5}'/>
            </tr>
        </xpath>
    </template>
</odoo>

```

