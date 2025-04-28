# Odoo Module: l10n_cd

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Democratic Republic of the Congo - Accounting',
    'countries': ['cd'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for the Democratic Republic of the Congo.
===========================================================================

The Chart of Accounts is from SYSCOHADA.

    """,
    'depends': [
        'l10n_syscohada',
    ],
    'data': [
        'data/account_tax_report_data.xml'
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
<odoo>
    <record id="account_tax_report_cd" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.cd"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_cd_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_cd_sales" model="account.report.line">
                <field name="name">II. Operations carried out</field>
                <field name="code">CD_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cd_sales_taxable" model="account.report.line">
                        <field name="name">a) Taxable Turnover</field>
                        <field name="code">CD_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_sales_taxable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_GOODS_BASE.balance + CD_SERVICES_BASE.balance + CD_GOODS_SELF_BASE.balance + CD_SERVICES_SELF_BASE.balance + CD_PM.balance + CD_EXPORT.balance + CD_EXEMPT.balance + CD_NON_IMPOSABLE.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_cd_sales_taxable_goods" model="account.report.line">
                                <field name="name">1. Goods delivery</field>
                                <field name="code">CD_GOODS_BASE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_goods_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_1a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_taxable_services" model="account.report.line">
                                <field name="name">2. Services</field>
                                <field name="code">CD_SERVICES_BASE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_services_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_2a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_taxable_goods_self" model="account.report.line">
                                <field name="name">3. Goods self-delivery</field>
                                <field name="code">CD_GOODS_SELF_BASE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_goods_self_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_3a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_taxable_services_self" model="account.report.line">
                                <field name="name">4. Services self-delivery</field>
                                <field name="code">CD_SERVICES_SELF_BASE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_services_self_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_4a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_taxable_public_market" model="account.report.line">
                                <field name="name">5. Public Markets with external financing</field>
                                <field name="code">CD_PM</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_public_market_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_5a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_taxable_export" model="account.report.line">
                                <field name="name">6. Export</field>
                                <field name="code">CD_EXPORT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_export_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_6a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_taxable_exempt" model="account.report.line">
                                <field name="name">7. Exempted operations</field>
                                <field name="code">CD_EXEMPT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_exempt_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_7a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_taxable_non_imposable" model="account.report.line">
                                <field name="name">8. Non imposable operations</field>
                                <field name="code">CD_NON_IMPOSABLE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_taxable_non_imposable_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_8a</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_sales_tax" model="account.report.line">
                        <field name="name">b) Tax collected</field>
                        <field name="code">CD_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_sales_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_GOODS_TAX.balance + CD_SERVICES_TAX.balance + CD_GOODS_SELF_TAX.balance + CD_SERVICES_SELF_TAX.balance + CD_PM_TAX.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_cd_sales_tax_goods" model="account.report.line">
                                <field name="name">1. Goods delivery</field>
                                <field name="code">CD_GOODS_TAX</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_tax_goods_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_1b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_tax_services" model="account.report.line">
                                <field name="name">2. Services</field>
                                <field name="code">CD_SERVICES_TAX</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_tax_services_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_2b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_tax_goods_self" model="account.report.line">
                                <field name="name">3. Goods self-delivery</field>
                                <field name="code">CD_GOODS_SELF_TAX</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_tax_goods_self_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_3b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_tax_services_self" model="account.report.line">
                                <field name="name">4. Services self-delivery</field>
                                <field name="code">CD_SERVICES_SELF_TAX</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_tax_services_self_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_4b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_sales_tax_public_market" model="account.report.line">
                                <field name="name">5. Public Markets with external financing</field>
                                <field name="code">CD_PM_TAX</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_sales_tax_public_market_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_5b</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cd_importation_services" model="account.report.line">
                <field name="name">III. Services received from providers not established in DRC</field>
                <field name="code">CD_IMP_SERVICE</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cd_importation_services_base" model="account.report.line">
                        <field name="name">c) Invoice amounts</field>
                        <field name="code">CD_IMP_SERVICE_BASE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_importation_services_base_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CD_10c</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_importation_services_tax" model="account.report.line">
                        <field name="name">d) Tax collected</field>
                        <field name="code">CD_IMP_SERVICE_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_importation_services_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CD_10d</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cd_deductible" model="account.report.line">
                <field name="name">IV. Deductible tax on</field>
                <field name="code">CD_DEDUCTIBLE</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cd_deductible_assets" model="account.report.line">
                        <field name="name">11. Assets</field>
                        <field name="code">CD_ASSETS</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_deductible_assets_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_IMPORTATIONS_ASSETS.balance + CD_LOCAL_ASSETS.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_cd_deductible_importations_assets" model="account.report.line">
                                <field name="name">e) Importations</field>
                                <field name="code">CD_IMPORTATIONS_ASSETS</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_importations_assets_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_11e</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_deductible_local_assets" model="account.report.line">
                                <field name="name">f) local</field>
                                <field name="code">CD_LOCAL_ASSETS</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_local_assets_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_11f</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_deductible_goods" model="account.report.line">
                        <field name="name">12. Goods</field>
                        <field name="code">CD_GOODS</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_deductible_goods_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_IMPORTATIONS_GOODS.balance + CD_LOCAL_GOODS.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_cd_deductible_importations_goods" model="account.report.line">
                                <field name="name">e) Importations</field>
                                <field name="code">CD_IMPORTATIONS_GOODS</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_importations_goods_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_12e</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_deductible_local_goods" model="account.report.line">
                                <field name="name">f) local</field>
                                <field name="code">CD_LOCAL_GOODS</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_local_goods_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_12f</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_deductible_raw_materials" model="account.report.line">
                        <field name="name">13. Raw materials</field>
                        <field name="code">CD_RAW</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_deductible_raw_materials_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_IMPORTATIONS_RAW.balance + CD_LOCAL_RAW.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_cd_deductible_importations_raw_materials" model="account.report.line">
                                <field name="name">e) Importations</field>
                                <field name="code">CD_IMPORTATIONS_RAW</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_importations_raw_materials_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_13e</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_deductible_local_raw_materials" model="account.report.line">
                                <field name="name">f) local</field>
                                <field name="code">CD_LOCAL_RAW</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_local_raw_materials_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_13f</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_deductible_other" model="account.report.line">
                        <field name="name">14. Other goods and services</field>
                        <field name="code">CD_OTHER</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_deductible_other_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_IMPORTATIONS_OTHER.balance + CD_LOCAL_OTHER.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_cd_deductible_importations_other" model="account.report.line">
                                <field name="name">e) Importations</field>
                                <field name="code">CD_IMPORTATIONS_OTHER</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_importations_other_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_14e</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cd_deductible_local_other" model="account.report.line">
                                <field name="name">f) local</field>
                                <field name="code">CD_LOCAL_OTHER</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cd_deductible_local_other_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">CD_14f</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_deductible_total_deductible" model="account.report.line">
                        <field name="name">15. Total deductible VAT</field>
                        <field name="code">CD_TOTAL_DEDUCTIBLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_deductible_total_deductible_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_ASSETS.balance + CD_GOODS.balance + CD_RAW.balance + CD_OTHER.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_deductible_report" model="account.report.line">
                        <field name="name">16. Report of last month's credit</field>
                        <field name="code">CD_REPORT_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_deductible_report_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_REPORT_CREDIT._applied_carryover_balance</field>
                            </record>
                            <record id="account_tax_report_line_cd_deductible_report_balance_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_deductible_deductible_amount" model="account.report.line">
                        <field name="name">17. Deductible VAT amount</field>
                        <field name="code">CD_DEDUCTIBLE_AMOUNT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_deductible_deductible_amount_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_TOTAL_DEDUCTIBLE.balance + CD_REPORT_CREDIT.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cd_regularisations" model="account.report.line">
                <field name="name">V. Regularisations</field>
                <field name="code">CD_REGU</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_cd_regularisations_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CD_REPAYMENTS.balance + CD_ADD_DEDUCTION.balance + CD_RECOVERY.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cd_regularisations_repayments" model="account.report.line">
                        <field name="name">18. VAT Repayments</field>
                        <field name="code">CD_REPAYMENTS</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_regularisations_repayments_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_regularisations_add_deduction" model="account.report.line">
                        <field name="name">19. Additional Deductions</field>
                        <field name="code">CD_ADD_DEDUCTION</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_regularisations_add_deduction_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_mining" model="account.report.line">
                        <field name="name">20. VAT deducted at source by mining companies</field>
                        <field name="code">CD_MINING</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_mining_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_regularisations_recovery_pm" model="account.report.line">
                        <field name="name">21. Recovery of deductible vat credit on externally financed public contracts</field>
                        <field name="code">CD_RECOVERY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_regularisations_recovery_pm_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cd_tax_calculation" model="account.report.line">
                <field name="name">VI. Tax calculation</field>
                <field name="code">CD_TAX_CALC</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cd_tax_calculation_to_pay" model="account.report.line">
                        <field name="name">22. Net VAT to pay (b9+d10+18+20-17-19-20-b5)</field>
                        <field name="code">CD_NET_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_tax_calculation_to_pay_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="subformula">if_above(CDF(0))</field>
                                <field name="formula">CD_TAX.balance + CD_IMP_SERVICE_TAX.balance + CD_REPAYMENTS.balance + CD_RECOVERY.balance - CD_DEDUCTIBLE_AMOUNT.balance - CD_ADD_DEDUCTION.balance - CD_MINING.balance - CD_PM_TAX.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_calculation_credit" model="account.report.line">
                        <field name="name">23. VAT Credit (17+19+20+b5-b9-d10-18-20)</field>
                        <field name="code">CD_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_calculation_credit_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="subformula">if_above(CDF(0))</field>
                                <field name="formula">CD_DEDUCTIBLE_AMOUNT.balance + CD_ADD_DEDUCTION.balance + CD_MINING.balance + CD_PM_TAX.balance - CD_TAX.balance - CD_IMP_SERVICE_TAX.balance - CD_REPAYMENTS.balance - CD_RECOVERY.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_calculation_repayment_asked" model="account.report.line">
                        <field name="name">24. Repayment of VAT credit asked</field>
                        <field name="code">CD_REPAYMENT_ASKED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_calculation_repayment_asked_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_calculation_credit_reportable" model="account.report.line">
                        <field name="name">25. VAT Credit reportable (23 - 24)</field>
                        <field name="code">CD_CREDIT_REPORTABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_calculation_credit_reportable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_CREDIT.balance - CD_REPAYMENT_ASKED.balance</field>
                                <field name="subformula">if_above(CDF(0))</field>
                                <field name="carryover_target" eval="False"/>
                            </record>
                            <record id="account_tax_report_line_cd_calculation_credit_reportable_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_CREDIT_REPORTABLE.balance</field>
                                <field name="carryover_target">CD_REPORT_CREDIT._applied_carryover_balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_calculation_pm" model="account.report.line">
                        <field name="name">26. VAT on Public Markets with external financing (b5)</field>
                        <field name="code">CD_CALCULATION_PM</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_calculation_pm_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_PM_TAX.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_calculation_vat_third_party" model="account.report.line">
                        <field name="name">27. VAT on behalf of third parties</field>
                        <field name="code">CD_THIRD_PARTY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_calculation_vat_third_party_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cd_calculation_to_pay" model="account.report.line">
                        <field name="name">28. Amount to pay (22 + 26 + 27)</field>
                        <field name="code">CD_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cd_calculation_to_pay_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CD_NET_TO_PAY.balance + CD_CALCULATION_PM.balance + CD_THIRD_PARTY.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-cd.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.cd","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_16","tva_export_0"
"","","","","","","","tva_sale_16_services","tva_export_0"
"","","","","","","","tva_purchase_good_16","tva_import_goods_16"
"","","","","","","","tva_purchase_assets_16","tva_import_assets_16"

```

## File: data\template\account.fiscal.position-cd_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.cd","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_16","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_16_services","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_good_16","syscebnl_tva_import_goods_16"
"","","","","","","","syscebnl_tva_purchase_assets_16","syscebnl_tva_import_assets_16"

```

## File: data\template\account.tax-cd.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"tva_sale_16","16% G","16% merchandises","True","","16.0","percent","sale","tax_group_16","base","invoice","+CD_1a","","","16% Marchandises",""
"","","","","","","","","","tax","invoice","+CD_1b","pcg_4431","","",""
"","","","","","","","","","base","refund","-CD_1a","","","",""
"","","","","","","","","","tax","refund","-CD_1b","pcg_4431","","",""
"tva_sale_16_services","16% S","16% Services","True","","16.0","percent","sale","tax_group_16","base","invoice","+CD_2a","","","",""
"","","","","","","","","","tax","invoice","+CD_2b","pcg_4431","","",""
"","","","","","","","","","base","refund","-CD_2a","","","",""
"","","","","","","","","","tax","refund","-CD_2b","pcg_4431","","",""
"tva_sale_16_pm","16% PM","16% Public Markets","False","","16.0","percent","sale","tax_group_16","base","invoice","+CD_5a","","","16% Marchés publics",""
"","","","","","","","","","tax","invoice","+CD_5b","pcg_4431","","",""
"","","","","","","","","","base","refund","-CD_5a","","","",""
"","","","","","","","","","tax","refund","-CD_5b","pcg_4431","","",""
"tva_sale_16_goods_sd","16% G SD","16% Goods Self Delivery","False","","16.0","percent","sale","tax_group_16","base","invoice","+CD_3a","","","16% Biens Livraison à soi-meme","16 G LASM"
"","","","","","","","","","tax","invoice","+CD_3b","pcg_4431","","",""
"","","","","","","","","","base","refund","-CD_3a","","","",""
"","","","","","","","","","tax","refund","-CD_3b","pcg_4431","","",""
"tva_sale_16_services_sd","16% S SD","16% Services Self Delivery","False","","16.0","percent","sale","tax_group_16","base","invoice","+CD_4a","","","16% Services Livraison à soi-meme","16 S LASM"
"","","","","","","","","","tax","invoice","+CD_4b","pcg_4431","","",""
"","","","","","","","","","base","refund","-CD_4a","","","",""
"","","","","","","","","","tax","refund","-CD_4b","pcg_4431","","",""
"tva_purchase_good_16","16% M","16% merchandises","True","","16.0","percent","purchase","tax_group_16","base","invoice","","","","16% Marchandises",""
"","","","","","","","","","tax","invoice","+CD_12f","pcg_4452","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CD_12f","pcg_4452","","",""
"tva_purchase_assets_16","16% Asset","16% assets","True","","16.0","percent","purchase","tax_group_16","base","invoice","","","","16% immobilisations","16% Immo"
"","","","","","","","","","tax","invoice","+CD_11f","pcg_4451","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CD_11f","pcg_4451","","",""
"tva_export_0","0% EX","0% (export)","True","","0.0","","sale","tax_group_0","base","invoice","+CD_6a","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CD_6a","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_import_goods_16","16% G EX","16% (import)","True","","16.0","","purchase","tax_group_16","base","invoice","+CD_1a","","","16% (importation)",""
"","","","","","","","","","tax","invoice","-CD_1b","pcg_4431","-100","",""
"","","","","","","","","","tax","invoice","+CD_12e","pcg_4451","","",""
"","","","","","","","","","base","refund","-CD_1a","","","",""
"","","","","","","","","","tax","refund","+CD_1b","pcg_4431","-100","",""
"","","","","","","","","","tax","refund","-CD_12e","pcg_4451","","",""
"tva_import_assets_16","16% I EX","16% (import of assets)","False","","16.0","","purchase","tax_group_16","base","invoice","+CD_1a","","","16% (importation d'immobilisations)",""
"","","","","","","","","","tax","invoice","-CD_1b","pcg_4431","-100","",""
"","","","","","","","","","tax","invoice","+CD_11e","pcg_4451","","",""
"","","","","","","","","","base","refund","-CD_1a","","","",""
"","","","","","","","","","tax","refund","+CD_1b","pcg_4431","-100","",""
"","","","","","","","","","tax","refund","-CD_11e","pcg_4451","","",""
"tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","tax_group_0","base","invoice","+CD_7a","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CD_7a","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_non_imposable_0","0% NI","0% (non imposable)","False","","0.0","","sale","tax_group_0","base","invoice","+CD_8a","","","",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CD_8a","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","False","","0.0","","purchase","tax_group_0","base","invoice","","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-cd_syscebnl.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"syscebnl_tva_sale_16","16% G","16% merchandises","True","","16.0","percent","sale","syscebnl_tax_group_16","base","invoice","+CD_1a","","","16% Marchandises",""
"","","","","","","","","","tax","invoice","+CD_1b","syscebnl_443","","",""
"","","","","","","","","","base","refund","-CD_1a","","","",""
"","","","","","","","","","tax","refund","-CD_1b","syscebnl_443","","",""
"syscebnl_tva_sale_16_services","16% S","16% Services","True","","16.0","percent","sale","syscebnl_tax_group_16","base","invoice","+CD_2a","","","",""
"","","","","","","","","","tax","invoice","+CD_2b","syscebnl_443","","",""
"","","","","","","","","","base","refund","-CD_2a","","","",""
"","","","","","","","","","tax","refund","-CD_2b","syscebnl_443","","",""
"syscebnl_tva_sale_16_pm","16% PM","16% Public Markets","False","","16.0","percent","sale","syscebnl_tax_group_16","base","invoice","+CD_5a","","","16% Marchés publics",""
"","","","","","","","","","tax","invoice","+CD_5b","syscebnl_443","","",""
"","","","","","","","","","base","refund","-CD_5a","","","",""
"","","","","","","","","","tax","refund","-CD_5b","syscebnl_443","","",""
"syscebnl_tva_sale_16_goods_sd","16% G SD","16% Goods Self Delivery","False","","16.0","percent","sale","syscebnl_tax_group_16","base","invoice","+CD_3a","","","16% Biens Livraison à soi-meme","16 G LASM"
"","","","","","","","","","tax","invoice","+CD_3b","syscebnl_443","","",""
"","","","","","","","","","base","refund","-CD_3a","","","",""
"","","","","","","","","","tax","refund","-CD_3b","syscebnl_443","","",""
"syscebnl_tva_sale_16_services_sd","16% S SD","16% Services Self Delivery","False","","16.0","percent","sale","syscebnl_tax_group_16","base","invoice","+CD_4a","","","16% Services Livraison à soi-meme","16 S LASM"
"","","","","","","","","","tax","invoice","+CD_4b","syscebnl_443","","",""
"","","","","","","","","","base","refund","-CD_4a","","","",""
"","","","","","","","","","tax","refund","-CD_4b","syscebnl_443","","",""
"syscebnl_tva_purchase_good_16","16% M","16% merchandises","True","","16.0","percent","purchase","syscebnl_tax_group_16","base","invoice","","","","16% Marchandises",""
"","","","","","","","","","tax","invoice","+CD_12f","syscebnl_445","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CD_12f","syscebnl_445","","",""
"syscebnl_tva_purchase_assets_16","16% Asset","16% assets","True","","16.0","percent","purchase","syscebnl_tax_group_16","base","invoice","","","","16% immobilisations","16% Immo"
"","","","","","","","","","tax","invoice","+CD_11f","syscebnl_445","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CD_11f","syscebnl_445","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","True","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+CD_6a","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CD_6a","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_goods_16","16% G EX","16% (import)","True","","16.0","","purchase","syscebnl_tax_group_16","base","invoice","+CD_1a","","","16% (importation)",""
"","","","","","","","","","tax","invoice","-CD_1b","syscebnl_443","-100","",""
"","","","","","","","","","tax","invoice","+CD_12e","syscebnl_445","","",""
"","","","","","","","","","base","refund","-CD_1a","","","",""
"","","","","","","","","","tax","refund","+CD_1b","syscebnl_443","-100","",""
"","","","","","","","","","tax","refund","-CD_12e","syscebnl_445","","",""
"syscebnl_tva_import_assets_16","16% I EX","16% (import of assets)","False","","16.0","","purchase","syscebnl_tax_group_16","base","invoice","+CD_1a","","","16% (importation d'immobilisations)",""
"","","","","","","","","","tax","invoice","-CD_1b","syscebnl_443","-100","",""
"","","","","","","","","","tax","invoice","+CD_11e","syscebnl_445","","",""
"","","","","","","","","","base","refund","-CD_1a","","","",""
"","","","","","","","","","tax","refund","+CD_1b","syscebnl_443","-100","",""
"","","","","","","","","","tax","refund","-CD_11e","syscebnl_445","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+CD_7a","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CD_7a","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_non_imposable_0","0% NI","0% (non imposable)","False","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+CD_8a","","","",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CD_8a","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","False","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-cd.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_16","VAT 16%","T.V.A. 16%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-cd_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_16","VAT 16%","T.V.A. 16%","syscebnl_443","syscebnl_445"

```

## File: models\template_cd.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cd')
    def _get_cd_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('cd', 'res.company')
    def _get_cd_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cd',
                'account_sale_tax_id': 'tva_sale_16',
                'account_purchase_tax_id': 'tva_purchase_good_16',
            }
        )
        return company_values

    @template('cd', 'account.account')
    def _get_cd_account_account(self):
        return self._parse_csv('cd', 'account.account', module='l10n_syscohada')

```

## File: models\template_cd_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cd_syscebnl')
    def _get_cd_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('cd_syscebnl', 'res.company')
    def _get_cd_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cd',
                'account_sale_tax_id': 'syscebnl_tva_sale_16',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_good_16',
            }
        )
        return company_values

    @template('cd_syscebnl', 'account.account')
    def _get_cd_syscebnl_account_account(self):
        return self._parse_csv('cd_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_cd
from . import template_cd_syscebnl

```

