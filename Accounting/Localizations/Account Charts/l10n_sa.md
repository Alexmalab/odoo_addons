# Odoo Module: l10n_sa

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Saudi Arabia - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['sa'],
    'version': '2.0',
    'author': 'Odoo S.A., DVIT.ME (http://www.dvit.me)',
    'category': 'Accounting/Localizations/Account Charts',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/saudi_arabia.html',
    'description': """
Saudi Arabia Accounting Module
===========================================================
Saudi Arabia Accounting Basic Charts and Localization

Activates:

- Chart of Accounts
- Taxes
- Vat Filling Report
- Withholding Tax Report
- Fiscal Positions
""",
    'depends': [
        'l10n_gcc_invoice',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_data.xml',
        'data/account_tax_report_data.xml',
        'data/report_paperformat_data.xml',
        'views/report_invoice.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_data.xml

```xml
<odoo>
    <data noupdate="1">
        <!-- set VAT label to show on invoice report -->
        <record id="base.sa" model="res.country">
            <field name="vat_label">VAT Number</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report_vat_filing" model="account.report">
        <field name="name">VAT Filing Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.sa"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_vat_filing_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_vat_all_sales_base" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Base)</field>
                <field name="aggregation_formula">SA_STD_SALE_B.balance + SA_SPCL_SALE_B.balance + SA_ZERO_SALE_B.balance + SA_EXP_SALE_B.balance + SA_EXM_SALE_B.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_15_base" model="account.report.line">
                        <field name="name">1. Standard Rated 15% (Base)</field>
                        <field name="code">SA_STD_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_15_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1. Standard Rates 15% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_special_sales_to_locals_base" model="account.report.line">
                        <field name="name">2. Special Sales to Locals (Base)</field>
                        <field name="code">SA_SPCL_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_special_sales_to_locals_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Special Sales to Locals (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_local_sales_subject_to_0_base" model="account.report.line">
                        <field name="name">3. Local Sales Subject to 0% (Base)</field>
                        <field name="code">SA_ZERO_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_local_sales_subject_to_0_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Local Sales Subject to 0% (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_export_sales_base" model="account.report.line">
                        <field name="name">4. Export Sales (Base)</field>
                        <field name="code">SA_EXP_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_export_sales_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4. Export Sales (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_sales_base" model="account.report.line">
                        <field name="name">5. Exempt Sales (Base)</field>
                        <field name="code">SA_EXM_SALE_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_sales_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. Exempt Sales (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_net_sales_base" model="account.report.line">
                        <field name="name">6. Net Sales (Base)</field>
                        <field name="aggregation_formula">SA_STD_SALE_B.balance + SA_SPCL_SALE_B.balance + SA_ZERO_SALE_B.balance + SA_EXP_SALE_B.balance + SA_EXM_SALE_B.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_all_sales_tax" model="account.report.line">
                <field name="name">VAT on Sales and all other Outputs (Tax)</field>
                <field name="aggregation_formula">SA_STD_SALE_T.balance + SA_SPCL_SALE_T.balance + SA_ZERO_SALE_T.balance + SA_EXP_SALE_T.balance + SA_EXM_SALE_T.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_15_tax" model="account.report.line">
                        <field name="name">1. Standard Rated 15% (Tax)</field>
                        <field name="code">SA_STD_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_15_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1. Standard Rates 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_special_sales_to_locals_tax" model="account.report.line">
                        <field name="name">2. Special Sales to Locals (Tax)</field>
                        <field name="code">SA_SPCL_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_special_sales_to_locals_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Special Sales to Locals (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_local_sales_subject_to_0_tax" model="account.report.line">
                        <field name="name">3. Local Sales Subject to 0% (Tax)</field>
                        <field name="code">SA_ZERO_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_local_sales_subject_to_0_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Local Sales Subject to 0% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_export_sales_tax" model="account.report.line">
                        <field name="name">4. Export Sales (Tax)</field>
                        <field name="code">SA_EXP_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_export_sales_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4. Export Sales (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_sales_tax" model="account.report.line">
                        <field name="name">5. Exempt Sales (Tax)</field>
                        <field name="code">SA_EXM_SALE_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_sales_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. Exempt Sales (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_net_sales_tax" model="account.report.line">
                        <field name="name">6. Net Sales (Tax)</field>
                        <field name="aggregation_formula">SA_STD_SALE_T.balance + SA_SPCL_SALE_T.balance + SA_ZERO_SALE_T.balance + SA_EXP_SALE_T.balance + SA_EXM_SALE_T.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_all_expenses_base" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Base)</field>
                <field name="aggregation_formula">SA_STD_PUR_B.balance + SA_CUST_PUR_B.balance + SA_RCM_PUR_B.balance + SA_ZER_PUR_B.balance + SA_EXM_PUR_B.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_15_purchases_base" model="account.report.line">
                        <field name="name">7. Standard rated 15% Purchases (Base)</field>
                        <field name="code">SA_STD_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_15_purchases_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Standard rated 15% Purchases (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_taxable_imports_15_paid_to_customs_base" model="account.report.line">
                        <field name="name">8. Taxable Imports 15% Paid to Customs (Base)</field>
                        <field name="code">SA_CUST_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_taxable_imports_15_paid_to_customs_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8. Taxable Imports 15% Paid to Customs (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_imports_subject_tp_reverse_charge_mechanism_base" model="account.report.line">
                        <field name="name">9. Imports subject to reverse charge mechanism (Base)</field>
                        <field name="code">SA_RCM_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_imports_subject_tp_reverse_charge_mechanism_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9. Imports subject to reverse charge mechanism (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_zero_rated_purchases_base" model="account.report.line">
                        <field name="name">10. Zero Rated Purchases (Base)</field>
                        <field name="code">SA_ZER_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_zero_rated_purchases_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10. Zero Rated Purchases (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_purchases_base" model="account.report.line">
                        <field name="name">11. Exempt Purchases (Base)</field>
                        <field name="code">SA_EXM_PUR_B</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_purchases_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11. Exempt Purchases (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_net_purchases_base" model="account.report.line">
                        <field name="name">12. Net Purchases (Base)</field>
                        <field name="aggregation_formula">SA_STD_PUR_B.balance + SA_CUST_PUR_B.balance + SA_RCM_PUR_B.balance + SA_ZER_PUR_B.balance + SA_EXM_PUR_B.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_all_expenses_tax" model="account.report.line">
                <field name="name">VAT on Expenses and all other Inputs (Tax)</field>
                <field name="aggregation_formula">SA_STD_PUR_T.balance + SA_CUST_PUR_T.balance + SA_RCM_PUR_T.balance + SA_ZER_PUR_T.balance + SA_EXM_PUR_T.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_standard_rated_15_purchases_tax" model="account.report.line">
                        <field name="name">7. Standard rated 15% Purchases (Tax)</field>
                        <field name="code">SA_STD_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_standard_rated_15_purchases_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Standard rated 15% Purchases (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_taxable_imports_15_paid_to_customs_tax" model="account.report.line">
                        <field name="name">8. Taxable Imports 15% Paid to Customs (Tax)</field>
                        <field name="code">SA_CUST_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_taxable_imports_15_paid_to_customs_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8. Taxable Imports 15% Paid to Customs (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_imports_subject_tp_reverse_charge_mechanism_tax" model="account.report.line">
                        <field name="name">9. Imports subject to reverse charge mechanism (Tax)</field>
                        <field name="code">SA_RCM_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_imports_subject_tp_reverse_charge_mechanism_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9. Imports subject to reverse charge mechanism (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_zero_rated_purchases_tax" model="account.report.line">
                        <field name="name">10. Zero Rated Purchases (Tax)</field>
                        <field name="code">SA_ZER_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_zero_rated_purchases_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10. Zero Rated Purchases (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_exempt_purchases_tax" model="account.report.line">
                        <field name="name">11. Exempt Purchases (Tax)</field>
                        <field name="code">SA_EXM_PUR_T</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_exempt_purchases_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11. Exempt Purchases (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_net_purchases_tax" model="account.report.line">
                        <field name="name">12. Net Purchases (Tax)</field>
                        <field name="aggregation_formula">SA_STD_PUR_T.balance + SA_CUST_PUR_T.balance + SA_RCM_PUR_T.balance + SA_ZER_PUR_T.balance + SA_EXM_PUR_T.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_net_vat_due" model="account.report.line">
                <field name="name">Net VAT Due</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_line_net_vat_due_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_total_value_of_due_tax_for_the_period" model="account.report.line">
                        <field name="name">Total value of due tax for the period</field>
                        <field name="aggregation_formula">SA_STD_SALE_T.balance + SA_SPCL_SALE_T.balance + SA_ZERO_SALE_T.balance + SA_EXP_SALE_T.balance + SA_EXM_SALE_T.balance</field>
                    </record>
                    <record id="tax_report_line_total_value_of_recoverable_tax_for_the_period" model="account.report.line">
                        <field name="name">Total value of recoverable tax for the period</field>
                        <field name="aggregation_formula">SA_STD_PUR_T.balance + SA_CUST_PUR_T.balance + SA_RCM_PUR_T.balance + SA_ZER_PUR_T.balance + SA_EXM_PUR_T.balance</field>
                    </record>
                    <record id="tax_report_line_net_vat_due_or_reclaimed_for_the_period" model="account.report.line">
                        <field name="name">Net VAT due (or reclaimed) for the period</field>
                        <field name="aggregation_formula">SA_STD_SALE_T.balance + SA_SPCL_SALE_T.balance + SA_ZERO_SALE_T.balance + SA_EXP_SALE_T.balance + SA_EXM_SALE_T.balance - (SA_STD_PUR_T.balance + SA_CUST_PUR_T.balance + SA_RCM_PUR_T.balance + SA_ZER_PUR_T.balance + SA_EXM_PUR_T.balance)</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
    <record id="tax_report_withholding_tax" model="account.report">
        <field name="name">Withholding Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.sa"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_withholding_tax_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_withholding_tax_on_purchased_services_base" model="account.report.line">
                <field name="name">Withholding Tax on Purchased Services (Base)</field>
                <field name="aggregation_formula">SA_RENTB.balance + SA_AIRB.balance + SA_SEAB.balance + SA_TELEB.balance + SA_DIVB.balance + SA_CONB.balance + SA_ROLB.balance + SA_INSB.balance + SA_ROYB.balance + SA_MAIB.balance + SA_BRAB.balance + SA_OTHB.balance + SA_MAGB.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_withholding_tax_5_rental_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Rental) (Base)</field>
                        <field name="code">SA_RENTB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_rental_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Rental) (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_tickets_or_air_freight_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Tickets or Air Freight) (Base)</field>
                        <field name="code">SA_AIRB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_tickets_or_air_freight_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Tickets or Air Freight) (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_tickets_or_sea_freight_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Tickets or Sea Freight)(Base)</field>
                        <field name="code">SA_SEAB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_tickets_or_sea_freight_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Tickets or Sea Freight)(Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_international_telecommunication_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (International Telecommunication)(Base)</field>
                        <field name="code">SA_TELEB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_international_telecommunication_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (International Telecommunication)(Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_distributed_profits_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Distributed Profits) (Base)</field>
                        <field name="code">SA_DIVB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_distributed_profits_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Distributed Profits) (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_consulting_and_technical_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Consulting and Technical) (Base)</field>
                        <field name="code">SA_CONB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_consulting_and_technical_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Consulting and Technical) (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_return_from_loans_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Return from Loans) (Base)</field>
                        <field name="code">SA_ROLB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_return_from_loans_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Return from Loans) (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_insurance_and_reinsurance_base" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Insurance &amp; Reinsurance) (Base)</field>
                        <field name="code">SA_INSB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_insurance_and_reinsurance_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Insurance &amp; Reinsurance) (Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_royalties_base" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Royalties)(Base)</field>
                        <field name="code">SA_ROYB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_royalties_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Royalties)(Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_paid_services_from_main_branch_base" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Paid Services from Main Branch)(Base)</field>
                        <field name="code">SA_MAIB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_paid_services_from_main_branch_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Paid Services from Main Branch)(Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_paid_services_from_another_branch_base" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Paid Services from another branch)(Base)</field>
                        <field name="code">SA_BRAB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_paid_services_from_another_branch_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Paid Services from another branch)(Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_others_base" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Others)(Base)</field>
                        <field name="code">SA_OTHB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_others_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Others)(Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_20_managerial_base" model="account.report.line">
                        <field name="name">Withholding Tax 20% (Managerial)(Base)</field>
                        <field name="code">SA_MAGB</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_20_managerial_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 20% (Managerial)(Base)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_total_base" model="account.report.line">
                        <field name="name">Withholding Tax Total (Base)</field>
                        <field name="aggregation_formula">SA_RENTB.balance+SA_AIRB.balance+SA_SEAB.balance+SA_TELEB.balance+SA_DIVB.balance+SA_CONB.balance+SA_ROLB.balance+SA_INSB.balance+SA_ROYB.balance+SA_MAIB.balance+SA_BRAB.balance+SA_OTHB.balance+SA_MAGB.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_withholding_tax_on_purchased_services_tax" model="account.report.line">
                <field name="name">Withholding Tax on Purchased Services (Tax)</field>
                <field name="aggregation_formula">SA_RENTT.balance + SA_AIRT.balance + SA_SEAT.balance + SA_TELET.balance + SA_DIVT.balance + SA_CONT.balance + SA_ROLT.balance + SA_INST.balance + SA_ROYT.balance + SA_MAIT.balance + SA_BRAT.balance + SA_OTHT.balance + SA_MAGT.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_withholding_tax_5_rental_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Rental) (Tax)</field>
                        <field name="code">SA_RENTT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_rental_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Rental) (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_tickets_or_air_freight_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Tickets or Air Freight) (Tax)</field>
                        <field name="code">SA_AIRT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_tickets_or_air_freight_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Tickets or Air Freight) (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_tickets_or_sea_freight_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Tickets or Sea Freight)(Tax)</field>
                        <field name="code">SA_SEAT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_tickets_or_sea_freight_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Tickets or Sea Freight)(Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_international_telecommunication_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (International Telecommunication)(Tax)</field>
                        <field name="code">SA_TELET</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_international_telecommunication_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (International Telecommunication)(Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_distributed_profits_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Distributed Profits) (Tax)</field>
                        <field name="code">SA_DIVT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_distributed_profits_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Distributed Profits) (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_consulting_and_technical_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Consulting and Technical) (Tax)</field>
                        <field name="code">SA_CONT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_consulting_and_technical_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Consulting and Technical) (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_return_from_loans_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Return from Loans) (Tax)</field>
                        <field name="code">SA_ROLT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_return_from_loans_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Return from Loans) (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_5_insurance_and_reinsurance_tax" model="account.report.line">
                        <field name="name">Withholding Tax 5% (Insurance &amp; Reinsurance) (Tax)</field>
                        <field name="code">SA_INST</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_5_insurance_and_reinsurance_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 5% (Insurance &amp; Reinsurance) (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_royalties_tax" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Royalties)(Tax)</field>
                        <field name="code">SA_ROYT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_royalties_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Royalties)(Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_paid_services_from_main_branch_tax" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Paid Services from Main Branch)(Tax)</field>
                        <field name="code">SA_MAIT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_paid_services_from_main_branch_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Paid Services from Main Branch)(Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_paid_services_from_another_branch_tax" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Paid Services from another branch)(Tax)</field>
                        <field name="code">SA_BRAT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_paid_services_from_another_branch_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Paid Services from another branch)(Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_15_others_tax" model="account.report.line">
                        <field name="name">Withholding Tax 15% (Others)(Tax)</field>
                        <field name="code">SA_OTHT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_15_others_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 15% (Others)(Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_20_managerial_tax" model="account.report.line">
                        <field name="name">Withholding Tax 20% (Managerial)(Tax)</field>
                        <field name="code">SA_MAGT</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_withholding_tax_20_managerial_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Withholding Tax 20% (Managerial)(Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_withholding_tax_total_tax" model="account.report.line">
                        <field name="name">Withholding Tax Total (Tax)</field>
                        <field name="aggregation_formula">SA_RENTT.balance+SA_AIRT.balance+SA_SEAT.balance+SA_TELET.balance+SA_DIVT.balance+SA_CONT.balance+SA_ROLT.balance+SA_INST.balance+SA_ROYT.balance+SA_MAIT.balance+SA_BRAT.balance+SA_OTHT.balance+SA_MAGT.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\report_paperformat_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="paperformat_l10n_sa_a4" model="report.paperformat">
            <field name="name">Saudi Arabia A4</field>
            <field name="orientation">Portrait</field>
            <field name="margin_bottom">32</field>
            <field name="header_spacing">52</field>
            <field name="margin_top">52</field>
            <field name="margin_left">0</field>
            <field name="margin_right">0</field>
        </record>
    </data>
    <data>
        <function model="res.company" name="write">
            <value model="res.company" search="[
                ('partner_id.country_id', '=', ref('base.sa'))]"/>
            <value eval="{'paperformat_id': ref('l10n_sa.paperformat_l10n_sa_a4')}"/>
        </function>
    </data>
</odoo>

```

## File: data\template\account.account-sa.csv

```csv
"id","name","code","account_type","reconcile","name@ar"
"sa_account_100101","Right of use Asset (IFRS 16)","100101","asset_fixed","False","حق استخدام الأصول (IFRS 16)"
"sa_account_100102","Accumulated Depreciation right use asset (IFRS 16)","100102","asset_fixed","False","الاستهلاك المتراكم استخدام حق الأصول (IFRS 16)"
"sa_account_100103","VAT Receivable","100103","asset_non_current","False","ضريبة القيمة المضافة المدينة"
"sa_account_101005","Main Safe","101005","asset_current","False","خزينة رئيسية"
"sa_account_101006","Main Safe - Foreign Currency","101006","asset_current","False","خزينة رئيسية - عملات اخرى"
"sa_account_101007","Visa & Master Credit Cards","101007","asset_current","False","بطاقات الائتمان فيزا وماستر"
"sa_account_101008","Gateway Credit Cards","101008","asset_current","False","بطاقات الائتمان Gateway"
"sa_account_101009","Manual Visa & Master Cards","101009","asset_current","False","فيزا وماستر بطاقات"
"sa_account_101010","PayPal Account","101010","asset_current","False","Paypal رصيد"
"sa_account_101060","VAT Paid to Customs","101060","asset_current","False","ضريبة القيمة المضافة المدفوعة للجمارك"
"sa_account_102011","Accounts Receivable","102011","asset_receivable","True","الذمم المدينة"
"sa_account_102012","Accounts Receivable (PoS)","102012","asset_receivable","True","ذمم مدينة (PoS)"
"sa_account_102013","Post Dated Cheques Received","102013","asset_current","False","شيكات مؤجلة"
"sa_account_102014","Other Receivable","102014","asset_current","False","ذمم مدينة اخرى"
"sa_account_102015","Other Debtors","102015","asset_current","False","مدينون اخرون"
"sa_account_103016","Shipment Insurance","103016","asset_current","False","تأمين الشحن"
"sa_account_103017","Shipments Documentation Charges","103017","asset_current","False","رسوم"
"sa_account_103018","Shipment Other Charges","103018","asset_current","False","رسوم شحنات اخرى"
"sa_account_103019","Handling Difference in Inventory","103019","asset_current","False","فرق المخزون"
"sa_account_103020","Items Delivered to Customs on temprary Base","103020","asset_current","False","بنود في الجمرك"
"sa_account_104020","Prepaid Expense","104020","asset_current","False","المصروفات المدفوعة مقدماً"
"sa_account_104021","Prepaid Medical Insurance","104021","asset_current","False","تأمين طبي مدفوع مسبقا"
"sa_account_104022","Prepaid Life Insurance","104022","asset_current","False","تأمين على الحياة مدفوع مسبقا"
"sa_account_104023","Prepaid Office Rent","104023","asset_current","False","ايجار مكتب مدفوع مسبقا"
"sa_account_104024","Prepaid Other Insurance","104024","asset_current","False","تأمينات اخرى مدفوعة مسبقا"
"sa_account_104025","Prepaid License Fees","104025","asset_current","False","رسوم ترخيص مدفوعة مسبقا"
"sa_account_104026","Prepaid Maintenance","104026","asset_current","False","رسوم صيانة مدفوعة مسبقا"
"sa_account_104027","Prepaid Site Hosting Fees","104027","asset_current","False","رسوم استضافة موقع مدفوعة مسبقا"
"sa_account_104028","Prepaid Employees Housing","104028","asset_current","False","بدل سكن للموظفين مدفوع مسبقا"
"sa_account_104029","Prepaid Schooling Fees","104029","asset_current","False","بدل رسوم تعليم مدرسي مدفوع مسبقا"
"sa_account_104030","Prepaid Consultancy Fees","104030","asset_current","False","رسوم استشارات مدفوعة مسبقا"
"sa_account_104031","Prepaid Legal Fees","104031","asset_current","False","الرسوم القانونية مدفوعة مسبقا"
"sa_account_104032","Prepaid Sponsorship Fees","104032","asset_current","False","رسوم كفالة مدفوعة مسبقا"
"sa_account_104033","PrePaid Advertisement Expenses","104033","asset_current","False","دعاية و الإعلان مدفوعة مسبقا"
"sa_account_104034","Prepaid Bank Guarantee","104034","asset_current","False","ضمان بنكي مدفوع مسبقا"
"sa_account_104035","Other Prepayments","104035","asset_current","False","دفعات مقدمة أخرى"
"sa_account_104036","Prepaid Finance charge for Loans","104036","asset_current","False","تكاليف تمويل قروض مدفوعة مسبقا"
"sa_account_104037","Deposit - Office Rent","104037","asset_current","False","رسوم تأمين - ايجار مكتبي"
"sa_account_104038","Deposits - Customs","104038","asset_current","False","رسوم تأمين - جمارك"
"sa_account_104039","Deposit to Immigration (Visa)","104039","asset_current","False","رسوم تأمين - شؤون الهجرة"
"sa_account_104040","Deposit Others","104040","asset_current","False","رسوم تأمين - اخرى"
"sa_account_104041","VAT Input","104041","asset_current","False","مدخلات ضريبة القيمة المضافة"
"sa_account_106001","Leasehold Improvement","106001","asset_current","False","تحسين المستأجرات"
"sa_account_106002","Furniture and Equipment","106002","asset_current","False","أثاث و معدات"
"sa_account_106003","Computer Hardware & Software","106003","asset_current","False","الكمبيوترات و قطع الغيار و البرمجيات"
"sa_account_106004","Motor Vehicles","106004","asset_current","False","السيارات"
"sa_account_106005","Work In Progress","106005","asset_current","False","عمل جاري"
"sa_account_106006","Amortisation on Leasehold Improvement","106006","asset_current","False","اطفاء على تحسين المستأجرات"
"sa_account_106007","Acc.Deprn.of Furniture & Office Equipment","106007","asset_current","False","مجمع اهتلاك اثاث و معدات المكتب"
"sa_account_106008","Acc. Deprn.Computer Hardware & Software","106008","asset_current","False","مجمع اهتلاك الكمبيوترات و قطع الغيار و البرمجيات"
"sa_account_106009","Acc. Depreciation of Motor Vehicles","106009","asset_current","False","مجمع اهتلاك السيارات"
"sa_account_106010","Registration of Trademarks","106010","asset_current","False","تسجيل العلامات التجارية"
"sa_account_106011","Computer Card Renewal","106011","asset_current","False","بطاقة تجديد كمبيوتر"
"sa_account_201002","Payables","201002","liability_payable","True","الذمم الدائنة"
"sa_account_201003","Credit Notes to Customers","201003","liability_current","False","اشعار دائن للعملاء"
"sa_account_201004","Accrued - Salaries","201004","liability_current","False","الرواتب المستحقة"
"sa_account_201005","Leave Tickets Provision","201005","liability_current","False","مخصص تذاكر"
"sa_account_201006","Leave Days Provision","201006","liability_current","False","مخصص ايام اجازة"
"sa_account_201007","Accrued - Commissions","201007","liability_current","False","عمولة مستحقة"
"sa_account_201008","Accrued Salaries Increment","201008","liability_current","False","راتب اضافي مستحق"
"sa_account_201009","Accrued-Staff Bonus","201009","liability_current","False","مكافأة مستحقة"
"sa_account_201010","Accrued Other Personnel Cost","201010","liability_current","False","تكاليف موظفين مستحقة"
"sa_account_201011","Accrued - Utilities","201011","liability_current","False","فواتير مستحقة"
"sa_account_201012","Accrued - Telephone","201012","liability_current","False","نتكاليف هاتف مستحقة"
"sa_account_201013","Accrued - Sponsorship","201013","liability_current","False","تكفل مستحق"
"sa_account_201014","Accrued - Audit Fees","201014","liability_current","False","اتعاب تدقيق مستحقة"
"sa_account_201015","Accrued - Office Rent","201015","liability_current","False","ايجار مكتب مستحق"
"sa_account_201016","Accrued Others","201016","liability_current","False","اخرى مستحقة"
"sa_account_201017","VAT Output","201017","liability_current","False","مخرجات ضريبة القيمة المضافة"
"sa_account_201018","Deferred income","201018","liability_current","False","الإيرادات مؤجلة"
"sa_account_201019","Zakat Provision","201019","liability_current","False","مخصص الزكاة"
"sa_account_201020","Withholding Tax Payable","201020","liability_current","False","ضريبة مستقطعة مستحقة"
"sa_account_202001","End of Service Provision","202001","liability_non_current","False","مخصص نهاية الخدمة"
"sa_account_202002","Reservations","202002","liability_non_current","False","احتياطات و حجوزات"
"sa_account_202003","VAT Payable","202003","liability_non_current","False","ضريبة القيمة المضافة المستحقة"
"sa_account_400001","Cost of Goods Sold in Trading","400001","expense_direct_cost","False","تكلفة البضاعة المباعة في التجارة"
"sa_account_400002","Cost Of Goods Sold I/C Sales","400002","expense_direct_cost","False","تكلفة البضاعة المباعة مع المبيعات"
"sa_account_400003","Basic Salary","400003","expense","False","مصروف الراتب الاساسي"
"sa_account_400004","Housing Allowance","400004","expense","False","مصروف بدل سكن"
"sa_account_400005","Transportation Allowance","400005","expense","False","مصروف بدل نقل"
"sa_account_400006","Leave Ticket","400006","expense","False","مصروف تذاكر موظفين"
"sa_account_400007","Leave Salary","400007","expense","False","مصروف اجازة موظفين"
"sa_account_400008","End Of Service Indemnity","400008","expense","False","مصروف نهاية الخدمة"
"sa_account_400009","Medical Insurance","400009","expense","False","مصروف تأمين طبي"
"sa_account_400010","Life Insurance","400010","expense","False","مصروف تأمين على الحياة"
"sa_account_400011","Sales Commission","400011","expense","False","مصروف عمولة مبيعات"
"sa_account_400012","Staff Other Allowances","400012","expense","False","مصروف بدلات اخرى للموظفين"
"sa_account_400013","Uniform","400013","expense","False","مصروف زي موحد"
"sa_account_400014","Visa Expenses","400014","expense","False","مصروف تأشيرة"
"sa_account_400015","Personnel Cost Others","400015","expense","False","مصروف موظفين - اخرى"
"sa_account_400016","Office Rent","400016","expense","False","مصروف اجار مكتب"
"sa_account_400017","Warehouse Rent","400017","expense","False","مصروف ايجار مستودع"
"sa_account_400018","Water & Electricity","400018","expense","False","مصروف مياه و كهرباء"
"sa_account_400019","Other Utility Cahrges","400019","expense","False","مصروف خدمات اخرى"
"sa_account_400020","Telephone","400020","expense","False","مصروف هاتف"
"sa_account_400021","Courrier","400021","expense","False","مصروف شحن"
"sa_account_400022","Web Site Hosting Fees","400022","expense","False","رسوم استضافة موقع"
"sa_account_400023","Others - Communication","400023","expense","False","مصاريف اتصالات اخرى"
"sa_account_400024","Air tickets","400024","expense","False","مصاريف تذاكر طيران"
"sa_account_400025","Hotel","400025","expense","False","مصاريف فندق"
"sa_account_400026","Meals","400026","expense","False","مصاريف وجبات"
"sa_account_400027","Per Diem","400027","expense","False","مصاريف يومية"
"sa_account_400028","Others","400028","expense","False","مصاريف اخرى"
"sa_account_400029","Audit Fees","400029","expense","False","مصروف اتعاب تدقيق"
"sa_account_400030","Sponsorship Fees","400030","expense","False","مصروف رسوم كفالة"
"sa_account_400031","Legal fees","400031","expense","False","مصروف رسوم قانونية"
"sa_account_400032","Trade License Fees","400032","expense","False","مصروف رسوم الرخصة التجارية"
"sa_account_400033","Others - Professional Fees","400033","expense","False","مصروف أخرى الرسوم الفنية"
"sa_account_400034","Other - Advertising Expenses","400034","expense","False","مصروف أخرى مصاريف الإعلان"
"sa_account_400035","Write Off Receivables & Payables","400035","expense","False","مصروف شطب الذمم الدائنة"
"sa_account_400036","Write Off Inventory","400036","expense","False","مصروف فرق المخزون"
"sa_account_400037","Amortisation of Preoperating Expenses","400037","expense","False","مصروف إطفاء مصاريف"
"sa_account_400038","Cash Shortage","400038","expense","False","مصروف نقص نقدي"
"sa_account_400039","Others - Provision & Write off","400039","expense","False","مخصصات و فروقات اخرى"
"sa_account_400040","Insurance","400040","expense","False","مصروف تأمين"
"sa_account_400041","Training","400041","expense","False","مصروف تدريب"
"sa_account_400042","Maintenance","400042","expense","False","مصروف صيانة"
"sa_account_400043","Security & Guard","400043","expense","False","مصروف حراسة و امن"
"sa_account_400044","Cleaning","400044","expense","False","مصروف تنظيف"
"sa_account_400045","Subscriptions","400045","expense","False","مصروف الاشتراكات"
"sa_account_400046","Gifts & Donations","400046","expense","False","مصروف هدايا و هبات"
"sa_account_400047","Kitchen and Buffet Expenses","400047","expense","False","مصروف المطبخ وبوفيه"
"sa_account_400048","Vehicle Expenses","400048","expense","False","مصروف سيارة"
"sa_account_400049","Convoyance Expenses","400049","expense","False","مصروف نقل اصول"
"sa_account_400050","Others - Office Various Expenses","400050","expense","False","مصاريف مكتب اخرى"
"sa_account_400051","Other Bank Charges","400051","expense","False","مصروف الرسوم المصرفية الأخرى"
"sa_account_400052","Loss On Fixed Assets Disposal","400052","expense","False","مصروف خسارة بيع و تخلص من اصول"
"sa_account_400053","Loss on Difference on Exchange","400053","expense","False","مصروف خسارة على الفرق العملات"
"sa_account_400054","Disposal of Business Branch","400054","expense","False","مصروف وقف فرع من الاعمال"
"sa_account_400055","Income Tax","400055","expense","False","مصروف ضريبة الدخل"
"sa_account_400056","Previous Year Adjustments Account","400056","expense","False","مصروف حساب تسويات السنة السابقة"
"sa_account_400057","Other Non Operating Expenses","400057","expense","False","المصاريف غير التشغيلية"
"sa_account_400058","Credit Card Charges","400058","expense","False","مصروف رسوم بطاقات الائتمان"
"sa_account_400059","Bank Finance & Loan Charges","400059","expense","False","مصروف بنك التمويل والقروض"
"sa_account_400060","Air Miles Card Charges","400060","expense","False","مصروف رسوم بطاقة Air Miles"
"sa_account_400062","PayPal Charges","400062","expense","False","Paypal مصروف رسوم"
"sa_account_400063","Amortization on Leasehold Improvement","400063","expense","False","مصروف إطفاء تحسينات المستأجرة"
"sa_account_400064","Depreciation Of Furniture & Office Equipment","400064","expense","False","مصروف الاستهلاك  الأثاث"
"sa_account_400065","Depreciation Of Computer Hard & Soft","400065","expense","False","مصروف الاستهلاك اجهزة الكمبيوتر"
"sa_account_400066","Depreciation Of Motor Vehicles","400066","expense","False","مصروف استهلاك المركبات"
"sa_account_400067","Consultancy Fees","400067","expense","False","رسوم الاستشارات"
"sa_account_400068","Provision for Doubtful Debts","400068","expense","False","مخصص الديون المعدومة"
"sa_account_400069","Closing Account","400069","expense","False","حساب ختامي"
"sa_account_400070","Depreciation on right of use asset (IFRS 16)","400070","expense","False","الاستهلاك في حق الأصول استخدام (IFRS 16)"
"sa_account_400072","Zakat Expense","400072","expense","False","مصاريف الزكاة"
"sa_account_400073","Withholding Tax Expense","400073","expense","False","مصروف ضريبة مستقطعة"
"sa_account_500001","Sales Account","500001","income","False","مبيعات"
"sa_account_500002","Sales of I/C","500002","income","False","مبيعات شركات تابعة"
"sa_account_500003","Management Consultancy Fees","500003","income","False","مكاسب استشارات ادارية"
"sa_account_500004","Sales from Other Region","500004","income","False","مبيعات مناطق اخرى"
"sa_account_500005","Advertising Income","500005","income","False","دخل الإعلانات"
"sa_account_500006","Branding Income","500006","income","False","دخل علامات تجارية"
"sa_account_500007","Space Rental Income","500007","income","False","دخل تأجير"
"sa_account_500008","Service Income","500008","income","False","دخل خدمات"
"sa_account_500009","Interest Revenue","500009","income","False","ايراد فائدة"
"sa_account_500010","Capital Gain","500010","income","False","مكاسب رأس المال"
"sa_account_500011","Gain On Difference Of Exchange","500011","income","False","ربح فرق عملات"
"sa_account_500013","Other Income","500013","income","False","دخول اخرى"
"sa_account_999999","Undistributed Profits/Losses","999999","equity_unaffected","False","ارباح / خسائر غير موزعة"

```

## File: data\template\account.fiscal.position-sa.csv

```csv
"id","name","auto_apply","sequence","country_id","country_group_id"
"l10n_sa_account_fiscal_position_ksa","KSA","1","16","base.sa",""
"l10n_sa_account_fiscal_position_gcc","GCC","1","16","","base.gulf_cooperation_council"
"l10n_sa_account_fiscal_position_non_gcc","Non-GCC","","16","",""

```

## File: data\template\account.tax-sa.csv

```csv
"id","name","type_tax_use","amount","amount_type","description","invoice_label","tax_group_id","tax_scope","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@ar"
"sa_sales_tax_15","15%","sale","15.0","percent","","Sales Tax 15%","sa_tax_group_taxes_15","","base","invoice","+1. Standard Rates 15% (Base)","","","المبيعات الخاضعة للنسبة الأساسية %15"
"","","","","","","","","","tax","invoice","+1. Standard Rates 15% (Tax)","sa_account_201017","",""
"","","","","","","","","","base","refund","-1. Standard Rates 15% (Base)","","",""
"","","","","","","","","","tax","refund","-1. Standard Rates 15% (Tax)","sa_account_201017","",""
"sa_local_sales_tax_0","0%","sale","0.0","percent","","Local Sales 0%","sa_tax_group_taxes_other","","base","invoice","+3. Local Sales Subject to 0% (Base)","","","المبيعات المحلية الخاضعة للنسبة الصفرية %0"
"","","","","","","","","","tax","invoice","+3. Local Sales Subject to 0% (Tax)","","",""
"","","","","","","","","","base","refund","-3. Local Sales Subject to 0% (Base)","","",""
"","","","","","","","","","tax","refund","-3. Local Sales Subject to 0% (Tax)","","",""
"sa_export_sales_tax_0","0% EX","sale","0.0","percent","","Export Sales 0%","sa_tax_group_taxes_other","","base","invoice","+4. Export Sales (Base)","","","الصادرات %0"
"","","","","","","","","","tax","invoice","+4. Export Sales (Tax)","","",""
"","","","","","","","","","base","refund","-4. Export Sales (Base)","","",""
"","","","","","","","","","tax","refund","-4. Export Sales (Tax)","","",""
"sa_exempt_sales_tax_0","0% EXEMPT","sale","0.0","percent","","Exempt Sales Tax 0%","sa_tax_group_taxes_other","","base","invoice","+5. Exempt Sales (Base)","","","المبيعات المعفاة من الضريبة %0"
"","","","","","","","","","tax","invoice","+5. Exempt Sales (Tax)","","",""
"","","","","","","","","","base","refund","-5. Exempt Sales (Base)","","",""
"","","","","","","","","","tax","refund","-5. Exempt Sales (Tax)","","",""
"sa_purchase_tax_15","15%","purchase","15.0","percent","","Purchase Tax 15%","sa_tax_group_taxes_15","","base","invoice","+7. Standard rated 15% Purchases (Base)","","","المشتريات الخاضعة للنسبة الأساسية %15"
"","","","","","","","","","tax","invoice","+7. Standard rated 15% Purchases (Tax)","sa_account_104041","",""
"","","","","","","","","","base","refund","-7. Standard rated 15% Purchases (Base)","","",""
"","","","","","","","","","tax","refund","-7. Standard rated 15% Purchases (Tax)","sa_account_104041","",""
"sa_rcp_tax_15","15% R C","purchase","15.0","percent","","Reverse charge provision Tax 15%","sa_tax_group_taxes_15","service","base","invoice","+9. Imports subject to reverse charge mechanism (Base)","","","الاستيرادات الخاضعة لضريبة القيمة المضافة التي تُطبق عليها آلية الاحتساب العكسي %15"
"","","","","","","","","","tax","invoice","+9. Imports subject to reverse charge mechanism (Tax)","sa_account_104041","",""
"","","","","","","","","","base","refund","-9. Imports subject to reverse charge mechanism (Base)","","",""
"","","","","","","","","","tax","refund","-9. Imports subject to reverse charge mechanism (Tax)","sa_account_104041","",""
"sa_import_tax_paid_15_paid_to_customs","15% EX","purchase","15.0","percent","","Import tax 15% Paid to customs","sa_tax_group_taxes_other","","base","invoice","+8. Taxable Imports 15% Paid to Customs (Base)","","","الاستيرادات الخاضعة لضريبة القيمة المضافة بالنسبة الأساسية و التي تدفع في الجمارك %15"
"","","","","","","","","","tax","invoice","+8. Taxable Imports 15% Paid to Customs (Tax)","sa_account_101060","",""
"","","","","","","","","","base","refund","-8. Taxable Imports 15% Paid to Customs (Base)","","",""
"","","","","","","","","","tax","refund","-8. Taxable Imports 15% Paid to Customs (Tax)","sa_account_101060","",""
"sa_purchases_tax_0","0%","purchase","0.0","percent","","Purchases 0%","sa_tax_group_taxes_other","","base","invoice","+10. Zero Rated Purchases (Base)","","","المشتريات الخاضعة لنسبة %0"
"","","","","","","","","","tax","invoice","+10. Zero Rated Purchases (Tax)","","",""
"","","","","","","","","","base","refund","-10. Zero Rated Purchases (Base)","","",""
"","","","","","","","","","tax","refund","-10. Zero Rated Purchases (Tax)","","",""
"sa_exempt_purchases_tax","0% EXEMPT","purchase","0.0","percent","","Exempt Purchases","sa_tax_group_taxes_other","","base","invoice","+11. Exempt Purchases (Base)","","","المشتريات المعفاة من الضريبة"
"","","","","","","","","","tax","invoice","+11. Exempt Purchases (Tax)","","",""
"","","","","","","","","","base","refund","-11. Exempt Purchases (Base)","","",""
"","","","","","","","","","tax","refund","-11. Exempt Purchases (Tax)","","",""
"sa_withholding_tax_5_rental","5% WH R","purchase","5.0","percent","Withholding Tax 5% (Rental)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (Rental) (Base)","","","استقطاع الضريبة 5 % (إيجار)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (Rental) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (Rental) (Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (Rental) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_5_tickets_or_air_freight","5% WH T A","purchase","5.0","percent","Withholding Tax 5% (Tickets or Air Freight)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (Tickets or Air Freight) (Base)","","","استقطاع الضريبة 5 % (تذاكر طيران أو شحن جوي)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (Tickets or Air Freight) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (Tickets or Air Freight) (Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (Tickets or Air Freight) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_5_tickets_or_sea_freight","5% WH T S","purchase","5.0","percent","Withholding Tax 5% (Tickets or Sea Freight)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (Tickets or Sea Freight)(Base)","","","استقطاع الضريبة 5 % (تذاكر أو شحن بحري)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (Tickets or Sea Freight)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (Tickets or Sea Freight)(Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (Tickets or Sea Freight)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_5_international_telecommunication","5% WH I T","purchase","5.0","percent","Withholding Tax 5% (International Telecommunication)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (International Telecommunication)(Base)","","","استقطاع الضريبة 5 % (خدمات اتصاالت هاتفية دولية)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (International Telecommunication)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (International Telecommunication)(Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (International Telecommunication)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_5_distributed_profits","5% WH D P","purchase","5.0","percent","Withholding Tax 5% (Distributed Profits)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (Distributed Profits) (Base)","","","استقطاع الضريبة 5 % (أرباح موزعة)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (Distributed Profits) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (Distributed Profits) (Tax)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (Distributed Profits) (Base)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_5_consulting_and_technical","5% WH C","purchase","5.0","percent","Withholding Tax 5% (Consulting and Technical)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (Consulting and Technical) (Base)","","","استقطاع الضريبة 5 % (خدمات فنية أو استشارية)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (Consulting and Technical) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (Consulting and Technical) (Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (Consulting and Technical) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_5_return_from_loans","5% WH L","purchase","5.0","percent","Withholding Tax 5% (Return from Loans)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (Return from Loans) (Base)","","","استقطاع الضريبة 5 % (عوائد قروض)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (Return from Loans) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (Return from Loans) (Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (Return from Loans) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_5_insurance_amd_reinsurance","5% WH I","purchase","5.0","percent","Withholding Tax 5% (Insurance & Reinsurance)","Withholding Tax 5%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 5% (Insurance & Reinsurance) (Base)","","","استقطاع الضريبة 5 % (قسط تأمين أو إعادة تأمين)"
"","","","","","","","","","tax","invoice","+Withholding Tax 5% (Insurance & Reinsurance) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 5% (Insurance & Reinsurance) (Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 5% (Insurance & Reinsurance) (Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_15_royalties","15% WH R","purchase","15.0","percent","Withholding Tax 15% (Royalties)","Withholding Tax 15%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 15% (Royalties)(Base)","","","استقطاع الضريبة 15 % (أتاوة أو ريع)"
"","","","","","","","","","tax","invoice","+Withholding Tax 15% (Royalties)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 15% (Royalties)(Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 15% (Royalties)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_15_paid_services_from_main_branch","15% WH S M","purchase","15.0","percent","Withholding Tax 15% (Paid Services from Main Branch)","Withholding Tax 15%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 15% (Paid Services from Main Branch)(Base)","","","استقطاع الضريبة 15 % (خدمات مدفوعة للمركز الرئيسي)"
"","","","","","","","","","tax","invoice","+Withholding Tax 15% (Paid Services from Main Branch)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 15% (Paid Services from Main Branch)(Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 15% (Paid Services from Main Branch)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_15_paid_services_from_another_branch","15% WH S O","purchase","15.0","percent","Withholding Tax 15% (Paid Services from another branch)","Withholding Tax 15%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 15% (Paid Services from another branch)(Base)","","","استقطاع الضريبة 15 % (خدمات مدفوعة لشركة مرتبطة)"
"","","","","","","","","","tax","invoice","+Withholding Tax 15% (Paid Services from another branch)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 15% (Paid Services from another branch)(Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 15% (Paid Services from another branch)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_15_others","15% WH O","purchase","15.0","percent","Withholding Tax 15% (Others)","Withholding Tax 15%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 15% (Others)(Base)","","","استقطاع الضريبة 15 % (لأي دفعات أخرى)"
"","","","","","","","","","tax","invoice","+Withholding Tax 15% (Others)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 15% (Others)(Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 15% (Others)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""
"sa_withholding_tax_20_managerial","20% WH M","purchase","20.0","percent","Withholding Tax 20% (Managerial)","Withholding Tax 20%","sa_tax_group_taxes_withholding","service","base","invoice","+Withholding Tax 20% (Managerial)(Base)","","","استقطاع الضريبة 20 % (أتعاب إدارة)"
"","","","","","","","","","tax","invoice","+Withholding Tax 20% (Managerial)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","invoice","","sa_account_201020","-100",""
"","","","","","","","","","base","refund","-Withholding Tax 20% (Managerial)(Base)","","",""
"","","","","","","","","","tax","refund","-Withholding Tax 20% (Managerial)(Tax)","sa_account_400073","",""
"","","","","","","","","","tax","refund","","sa_account_201020","-100",""

```

## File: data\template\account.tax.group-sa.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"sa_tax_group_taxes_15","VAT Taxes","base.sa","sa_account_100103","sa_account_202003"
"sa_tax_group_taxes_other","Other Taxes","base.sa","sa_account_100103","sa_account_202003"
"sa_tax_group_taxes_withholding","Withholding Tax","base.sa","sa_account_100103","sa_account_202003"

```

## File: i18n_extra\l10n_sa.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_sa
#
msgid ""
msgstr ""

#. module: l10n_sa
#: model:account.tax.report,name:l10n_sa.tax_report_vat_filing
msgid "VAT Filing Report"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_vat_all_sales_base
msgid "VAT on Sales and all other Outputs (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_standard_rated_15_base
msgid "1. Standard Rated 15% (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_special_sales_to_locals_base
msgid "2. Special Sales to Locals (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_local_sales_subject_to_0_base
msgid "3. Local Sales Subject to 0% (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_export_sales_base
msgid "4. Export Sales (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_exempt_sales_base
msgid "5. Exempt Sales (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_net_sales_base
msgid "6. Net Sales (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_vat_all_expenses_base
msgid "VAT on Expenses and all other Inputs (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_standard_rated_15_purchases_base
msgid "7. Standard rated 15% Purchases (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_taxable_imports_15_paid_to_customs_base
msgid "8. Taxable Imports 15% Paid to Customs (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_imports_subject_tp_reverse_charge_mechanism_base
msgid "9. Imports subject to reverse charge mechanism (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_zero_rated_purchases_base
msgid "10. Zero Rated Purchases (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_exempt_purchases_base
msgid "11. Exempt Purchases (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_net_purchases_base
msgid "12. Net Purchases (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_vat_all_sales_tax
msgid "VAT on Sales and all other Outputs (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_standard_rated_15_tax
msgid "1. Standard Rated 15% (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_special_sales_to_locals_tax
msgid "2. Special Sales to Locals (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_local_sales_subject_to_0_tax
msgid "3. Local Sales Subject to 0% (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_export_sales_tax
msgid "4. Export Sales (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_exempt_sales_tax
msgid "5. Exempt Sales (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_net_sales_tax
msgid "6. Net Sales (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_vat_all_expenses_tax
msgid "VAT on Expenses and all other Inputs (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_standard_rated_15_purchases_tax
msgid "7. Standard rated 15% Purchases (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_taxable_imports_15_paid_to_customs_tax
msgid "8. Taxable Imports 15% Paid to Customs (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_imports_subject_tp_reverse_charge_mechanism_tax
msgid "9. Imports subject to reverse charge mechanism (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_zero_rated_purchases_tax
msgid "10. Zero Rated Purchases (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_exempt_purchases_tax
msgid "11. Exempt Purchases (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_net_purchases_tax
msgid "12. Net Purchases (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_net_vat_due
msgid "Net VAT Due"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_total_value_of_due_tax_for_the_period
msgid "Total value of due tax for the period"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_net_vat_due_or_reclaimed_for_the_period
msgid "Net VAT due (or reclaimed) for the period"
msgstr ""

#. module: l10n_sa
#: model:account.tax.report,name:l10n_sa.tax_report_withholding_tax
msgid "Withholding Tax Report"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_on_purchased_services_base
msgid "Withholding Tax on Purchased Services (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_rental_base
msgid "Withholding Tax 5% (Rental) (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_tickets_or_air_freight_base
msgid "Withholding Tax 5% (Tickets or Air Freight) (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_tickets_or_sea_freight_base
msgid "Withholding Tax 5% (Tickets or Sea Freight)(Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_international_telecommunication_base
msgid "Withholding Tax 5% (International Telecommunication)(Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_distributed_profits_base
msgid "Withholding Tax 5% (Distributed Profits) (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_consulting_and_technical_base
msgid "Withholding Tax 5% (Consulting and Technical) (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_return_from_loans_base
msgid "Withholding Tax 5% (Return from Loans) (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_insurance_and_reinsurance_base
msgid "Withholding Tax 5% (Insurance & Reinsurance) (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_royalties_base
msgid "Withholding Tax 15% (Royalties)(Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_paid_services_from_main_branch_base
msgid "Withholding Tax 15% (Paid Services from Main Branch)(Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_paid_services_from_another_branch_base
msgid "Withholding Tax 15% (Paid Services from another branch)(Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_others_base
msgid "Withholding Tax 15% (Others)(Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_20_managerial_base
msgid "Withholding Tax 20% (Managerial)(Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_total_base
msgid "Withholding Tax Total (Base)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_on_purchased_services_tax
msgid "Withholding Tax on Purchased Services (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_rental_tax
msgid "Withholding Tax 5% (Rental) (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_tickets_or_air_freight_tax
msgid "Withholding Tax 5% (Tickets or Air Freight) (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_tickets_or_sea_freight_tax
msgid "Withholding Tax 5% (Tickets or Sea Freight)(Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_international_telecommunication_tax
msgid "Withholding Tax 5% (International Telecommunication)(Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_distributed_profits_tax
msgid "Withholding Tax 5% (Distributed Profits) (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_consulting_and_technical_tax
msgid "Withholding Tax 5% (Consulting and Technical) (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_return_from_loans_tax
msgid "Withholding Tax 5% (Return from Loans) (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_5_insurance_and_reinsurance_tax
msgid "Withholding Tax 5% (Insurance & Reinsurance) (Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_royalties_tax
msgid "Withholding Tax 15% (Royalties)(Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_paid_services_from_main_branch_tax
msgid "Withholding Tax 15% (Paid Services from Main Branch)(Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_paid_services_from_another_branch_tax
msgid "Withholding Tax 15% (Paid Services from another branch)(Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_15_others_tax
msgid "Withholding Tax 15% (Others)(Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_20_managerial_tax
msgid "Withholding Tax 20% (Managerial)(Tax)"
msgstr ""

#. module: l10n_sa
#: model:account.report.line,name:l10n_sa.tax_report_line_withholding_tax_total_tax
msgid "Withholding Tax Total (Tax)"
msgstr ""

#. module: l10n_sa
#: model:res.country,vat_label:base.sa
msgid "VAT Number"
msgstr ""

#. module: l10n_sa
#: model:res.currency,currency_unit_label:base.SAR
msgid "Riyal"
msgstr ""

#. module: l10n_sa
#: model:res.currency,currency_subunit_label:base.SAR
msgid "Halala"
msgstr ""

#. module: l10n_sa
#: model:account.tax.group,name:l10n_sa.sa_tax_group_taxes_15
msgid "VAT Taxes"
msgstr ""

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_repr


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_sa_qr_code_str = fields.Char(string='Zatka QR Code', compute='_compute_qr_code_str')
    l10n_sa_confirmation_datetime = fields.Datetime(string='Confirmation Date', readonly=True, copy=False)

    @api.depends('country_code', 'move_type')
    def _compute_show_delivery_date(self):
        # EXTENDS 'account'
        super()._compute_show_delivery_date()
        for move in self:
            if move.country_code == 'SA':
                move.show_delivery_date = move.is_sale_document()

    @api.depends('amount_total_signed', 'amount_tax_signed', 'l10n_sa_confirmation_datetime', 'company_id', 'company_id.vat')
    def _compute_qr_code_str(self):
        """ Generate the qr code for Saudi e-invoicing. Specs are available at the following link at page 23
        https://zatca.gov.sa/ar/E-Invoicing/SystemsDevelopers/Documents/20210528_ZATCA_Electronic_Invoice_Security_Features_Implementation_Standards_vShared.pdf
        """
        def get_qr_encoding(tag, field):
            company_name_byte_array = field.encode()
            company_name_tag_encoding = tag.to_bytes(length=1, byteorder='big')
            company_name_length_encoding = len(company_name_byte_array).to_bytes(length=1, byteorder='big')
            return company_name_tag_encoding + company_name_length_encoding + company_name_byte_array

        for record in self:
            qr_code_str = ''
            if record.l10n_sa_confirmation_datetime and record.company_id.vat:
                seller_name_enc = get_qr_encoding(1, record.company_id.display_name)
                company_vat_enc = get_qr_encoding(2, record.company_id.vat)
                time_sa = fields.Datetime.context_timestamp(self.with_context(tz='Asia/Riyadh'), record.l10n_sa_confirmation_datetime)
                timestamp_enc = get_qr_encoding(3, time_sa.isoformat())
                totals = record._get_l10n_sa_totals()
                invoice_total_enc = get_qr_encoding(4, float_repr(abs(totals['total_amount']), 2))
                total_vat_enc = get_qr_encoding(5, float_repr(abs(totals['total_tax']), 2))

                str_to_encode = seller_name_enc + company_vat_enc + timestamp_enc + invoice_total_enc + total_vat_enc
                qr_code_str = base64.b64encode(str_to_encode).decode()
            record.l10n_sa_qr_code_str = qr_code_str

    def _post(self, soft=True):
        res = super()._post(soft)
        for move in self:
            if move.country_code == 'SA' and move.is_sale_document():
                vals = {}
                if not move.l10n_sa_confirmation_datetime:
                    vals['l10n_sa_confirmation_datetime'] = fields.Datetime.now()
                if not move.delivery_date:
                    vals['delivery_date'] = move.invoice_date
                move.write(vals)
        return res

    def _l10n_sa_reset_confirmation_datetime(self):
        for move in self.filtered(lambda m: m.country_code == 'SA'):
            move.l10n_sa_confirmation_datetime = False

    def button_draft(self):
        self._l10n_sa_reset_confirmation_datetime()
        super().button_draft()

    def _get_l10n_sa_totals(self):
        self.ensure_one()
        return {
            'total_amount': self.amount_total_signed,
            'total_tax': self.amount_tax_signed,
        }

```

## File: models\template_sa.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, Command
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('sa')
    def _get_sa_template_data(self):
        return {
            'property_account_receivable_id': 'sa_account_102011',
            'property_account_payable_id': 'sa_account_201002',
            'property_account_expense_categ_id': 'sa_account_400001',
            'property_account_income_categ_id': 'sa_account_500001',
            'code_digits': '6',
        }

    @template('sa', 'res.company')
    def _get_sa_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.sa',
                'bank_account_code_prefix': '101',
                'cash_account_code_prefix': '105',
                'transfer_account_code_prefix': '100',
                'account_default_pos_receivable_account_id': 'sa_account_102012',
                'income_currency_exchange_account_id': 'sa_account_500011',
                'expense_currency_exchange_account_id': 'sa_account_400053',
                'account_sale_tax_id': 'sa_sales_tax_15',
                'account_purchase_tax_id': 'sa_purchase_tax_15',
                'deferred_expense_account_id': 'sa_account_104020',
                'deferred_revenue_account_id': 'sa_account_201018'
            },
        }

    @template('sa', 'account.journal')
    def _get_sa_account_journal(self):
        """ If Saudi Arabia chart, we add 3 new journals Tax Adjustments, IFRS 16 and Zakat"""
        return {
            "tax_adjustment": {
                'name': 'Tax Adjustments',
                'code': 'TA',
                'type': 'general',
                'show_on_dashboard': True,
                'sequence': 1,
            },
            "ifrs16": {
                'name': 'IFRS 16 Right of Use Asset',
                'code': 'IFRS',
                'type': 'general',
                'show_on_dashboard': True,
                'sequence': 10,
            },
            "zakat": {
                'name': 'Zakat',
                'code': 'ZAKAT',
                'type': 'general',
                'show_on_dashboard': True,
                'sequence': 10,
            }
        }

    @template('sa', 'account.account')
    def _get_sa_account_account(self):
        return {
            "sa_account_100101": {'allowed_journal_ids': [Command.link('ifrs16')]},
            "sa_account_100102": {'allowed_journal_ids': [Command.link('ifrs16')]},
            "sa_account_400070": {'allowed_journal_ids': [Command.link('ifrs16')]},
            "sa_account_201019": {'allowed_journal_ids': [Command.link('zakat')]},
            "sa_account_400072": {'allowed_journal_ids': [Command.link('zakat')]},
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_sa
from . import account_move

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="arabic_english_invoice" inherit_id="l10n_gcc_invoice.arabic_english_invoice">
        <xpath expr="//div[@name='due_date']" position="after">
            <div class="row" t-if="o.delivery_date" name="delivery_date">
                <div class="col-6"></div>
                <div class="col-2">
                    <strong style="white-space:nowrap">Delivery Date:
                    </strong>
                </div>
                <div class="col-2">
                    <span t-field="o.delivery_date"/>
                </div>
                <div class="col-2 text-end">
                    <strong style="white-space:nowrap">:
                        تاريخ التوصيل
                    </strong>
                </div>
            </div>
        </xpath>
        <xpath expr="//t[@t-set='address']" position="after">
            <t t-set="information_block">
                <div class="row">
                    <p class="col-6 me-3">
                        <img t-if="o.l10n_sa_qr_code_str"
                            style="display:block;"
                            t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s'%('QR', quote_plus(o.l10n_sa_qr_code_str), 200, 200)"/>
                    </p>
                    <div class="col-6" t-if="o.partner_shipping_id and (o.partner_shipping_id != o.partner_id)" groups="account.group_delivery_invoice_address" name="shipping_address_block">
                        <strong>Shipping Address:</strong>
                        <div t-field="o.partner_shipping_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                    </div>
                </div>
            </t>
        </xpath>
        <xpath expr="//th[@name='th_total']//span[2]" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//th[@name='th_total']//span[2]" position="after">
            <span>
                Subtotal<br/>(inclusive of VAT)
            </span>
        </xpath>
        <xpath expr="//th[@name='th_total']//span" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//th[@name='th_total']//span" position="after">
            <span>
                المجموع شامل ضريبة القيمة المضافة
            </span>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span[2]" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span[2]" position="after">
            <span>
                Subtotal<br/>(exclusive of VAT)
            </span>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span" position="after">
            <span>
                المجموع الفرعي بدون الضريبة
            </span>
        </xpath>
        <xpath expr="//th[@name='th_taxes']//span" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//th[@name='th_taxes']//span" position="after">
            <span>
                نسبة الضريبة
            </span>
        </xpath>
        <xpath expr="//tr" position="attributes">
            <attribute name="style">font-size: 14px;</attribute>
        </xpath>
        <xpath expr="//span[@t-field='line.l10n_gcc_invoice_tax_amount']" position="attributes">
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </xpath>
        <xpath expr="//span[@t-field='line.price_unit']" position="attributes">
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]//strong" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]//strong" position="after">
            <strong>
                Invoice Taxable Amount
                /<br/>
                المبلغ الخاضع للضريبة غير شامل ضريبة القيمة المضافة
            </strong>
        </xpath>
        <xpath expr="//tr[hasclass('o_total')]//strong" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//tr[hasclass('o_total')]//strong" position="after">
            <strong>
                Invoice Total (inclusive of VAT)
                /
                إجمالي قيمة الفاتورة شامل ضريبة القيمة المضافة
            </strong>
        </xpath>
        <xpath expr="//div[@name='invoice_date']//span" position="before">
            <span t-if="o.l10n_sa_confirmation_datetime" t-field="o.l10n_sa_confirmation_datetime"/>
        </xpath>
        <xpath expr="//div[@name='invoice_date']//span[@t-field='o.invoice_date']" position="attributes">
            <attribute name="t-if">not o.l10n_sa_confirmation_datetime</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]" position="attributes">
            <attribute name="class">clearfix pt-2 pb-2</attribute>
        </xpath>
    </template>
</odoo>

```

