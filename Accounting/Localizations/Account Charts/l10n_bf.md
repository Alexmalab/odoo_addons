# Odoo Module: l10n_bf

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Burkina Faso - Accounting",
    'countries': ['bf'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Burkina Faso.
=================================================================

The Chart of Accounts is from SYSCOHADA.

    """,
    'depends': [
        'l10n_syscohada',
        'account',
    ],
    'auto_install': ['account'],
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
    <record id="account_tax_report_bf" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.bf"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_bf_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_bf_turnover" model="account.report.line">
                <field name="name">III. Global Turnover Without VAT</field>
                <field name="code">BF_TURNOVER</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_bf_taxable" model="account.report.line">
                        <field name="name">C1. Taxable operations</field>
                        <field name="code">BF_TAXABLE</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_bf_taxable_day" model="account.report.line">
                                <field name="name">a. Day-to-day operations</field>
                                <field name="code">BF_DAY</field>
                                <field name="children_ids">
                                     <record id="account_tax_report_line_bf_taxable_day_sales" model="account.report.line">
                                        <field name="name">01 Sales, services, building works</field>
                                        <field name="code">BF_DAY_SALES</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_day_sales_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_01</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_day_self" model="account.report.line">
                                        <field name="name">02 Self Delivery (SD)</field>
                                        <field name="code">BF_SELF</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_day_self_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_02</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_day_assets" model="account.report.line">
                                        <field name="name">03 Sale of fixed assets</field>
                                        <field name="code">BF_ASSETS</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_day_assets_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_03</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_day_sales_10" model="account.report.line">
                                        <field name="name">04 Taxable operations at a rate of 10%</field>
                                        <field name="code">BF_DAY_SALES_10</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_day_sales_10_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_04</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_day_sales_other" model="account.report.line">
                                        <field name="name">05 Other taxable operations</field>
                                        <field name="code">BF_DAY_SALES_OTHER</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_day_sales_other_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_05</field>
                                            </record>
                                        </field>
                                     </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_bf_taxable_contract" model="account.report.line">
                                <field name="name">b. Contracts, order letters, other public and private contracts</field>
                                <field name="code">BF_CONTRACT</field>
                                <field name="children_ids">
                                     <record id="account_tax_report_line_bf_taxable_contract_sales" model="account.report.line">
                                        <field name="name">06 Sales</field>
                                        <field name="code">BF_CONTRACT_SALES</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_contract_sales_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_06</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_contract_services" model="account.report.line">
                                        <field name="name">07 Services</field>
                                        <field name="code">BF_CONTRACT_SERVICES</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_contract_services_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_07</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_contract_immo" model="account.report.line">
                                        <field name="name">08 Building works, public works</field>
                                        <field name="code">BF_CONTRACT_WORKS</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_contract_immo_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_08</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_contract_sales_10" model="account.report.line">
                                        <field name="name">09 Taxable operations at a rate of 10%</field>
                                        <field name="code">BF_CONTRACT_SALES_10</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_contract_sales_10_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_09</field>
                                            </record>
                                        </field>
                                     </record>
                                     <record id="account_tax_report_line_bf_taxable_contract_sales_other" model="account.report.line">
                                        <field name="name">10 Other taxable operations</field>
                                        <field name="code">BF_CONTRACT_SALES_OTHER</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_bf_taxable_contract_sales_other_balance" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">BF_10</field>
                                            </record>
                                        </field>
                                     </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_non_taxable" model="account.report.line">
                        <field name="name">C2. Non-taxable operations</field>
                        <field name="code">BF_NON_TAXABLE</field>
                        <field name="children_ids">
                             <record id="account_tax_report_line_bf_non_taxable_export" model="account.report.line">
                                <field name="name">11 Exports</field>
                                <field name="code">BF_EXPORT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_bf_non_taxable_export_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BF_11</field>
                                    </record>
                                </field>
                             </record>
                             <record id="account_tax_report_line_bf_non_taxable_foreign" model="account.report.line">
                                <field name="name">12 Other foreign trade transactions, tax-suspended sales</field>
                                <field name="code">BF_FOREIGN</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_bf_non_taxable_foreign_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BF_12</field>
                                    </record>
                                </field>
                             </record>
                             <record id="account_tax_report_line_bf_non_taxable_other" model="account.report.line">
                                <field name="name">13 Other non-taxable operations</field>
                                <field name="code">BF_NON_TAXABLE_OTHER</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_bf_non_taxable_other_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BF_13</field>
                                    </record>
                                </field>
                             </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_total_operations" model="account.report.line">
                        <field name="name">14 Total amount of operations</field>
                        <field name="code">BF_TOTAL_OPERATIONS</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_total_operations_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BF_DAY_SALES.balance + BF_SELF.balance + BF_ASSETS.balance + BF_DAY_SALES_10.balance + BF_DAY_SALES_OTHER.balance + BF_CONTRACT_SALES.balance + BF_CONTRACT_SERVICES.balance + BF_CONTRACT_WORKS.balance + BF_CONTRACT_SALES_10.balance + BF_CONTRACT_SALES_OTHER.balance + BF_EXPORT.balance + BF_FOREIGN.balance + BF_NON_TAXABLE_OTHER.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_bf_gross" model="account.report.line">
                <field name="name">IV. Gross VAT</field>
                <field name="code">BF_GROSS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                     <record id="account_tax_report_line_bf_taxable_18" model="account.report.line">
                        <field name="name">15 At normal rate (18%)</field>
                        <field name="code">BF_TAXABLE_18</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_bf_taxable_18_base" model="account.report.line">
                                <field name="name">Base tax excluded</field>
                                <field name="code">BF_TAXABLE_18_BASE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_bf_taxable_18_base_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BF_15_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_bf_taxable_18_tax" model="account.report.line">
                                <field name="name">Gross VAT amount</field>
                                <field name="code">BF_TAXABLE_18_TAX</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_bf_taxable_18_tax_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BF_15_tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                     </record>
                     <record id="account_tax_report_line_bf_taxable_10" model="account.report.line">
                        <field name="name">16 At reduced rate (10%)</field>
                        <field name="code">BF_TAXABLE_10</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_bf_taxable_10_base" model="account.report.line">
                                <field name="name">Base tax excluded</field>
                                <field name="code">BF_TAXABLE_10_BASE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_bf_taxable_10_base_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BF_16_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_bf_taxable_10_tax" model="account.report.line">
                                <field name="name">Gross VAT amount</field>
                                <field name="code">BF_TAXABLE_10_TAX</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_bf_taxable_10_tax_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BF_16_tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                     </record>
                     <record id="account_tax_report_line_bf_tax_omitted" model="account.report.line">
                        <field name="name">17 Previously omitted gross VAT to be repaid</field>
                        <field name="code">BF_TAX_OMITTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_tax_omitted_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BF_17</field>
                            </record>
                        </field>
                     </record>
                     <record id="account_tax_report_line_bf_tax_deducted" model="account.report.line">
                        <field name="name">18 VAT previously deducted to be repaid</field>
                        <field name="code">BF_TAX_DEDUCTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_tax_deducted_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BF_18</field>
                            </record>
                        </field>
                     </record>
                     <record id="account_tax_report_line_bf_gross_total" model="account.report.line">
                        <field name="name">19 Total gross VAT amount</field>
                        <field name="code">BF_GROSS_TOTAL</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_gross_total_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BF_TAXABLE_18_TAX.balance + BF_TAXABLE_10_TAX.balance + BF_TAX_OMITTED.balance + BF_TAX_DEDUCTED.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_bf_net" model="account.report.line">
                <field name="name">V. Net VAT</field>
                <field name="code">BF_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_bf_deductible" model="account.report.line">
                        <field name="name">20 Deductible VAT for the period</field>
                        <field name="code">BF_DEDUCTIBLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_deductible_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BF_20</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_credit_reported" model="account.report.line">
                        <field name="name">21 VAT credit from previous period</field>
                        <field name="code">BF_CREDIT_REPORTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_credit_reported_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BF_CREDIT_REPORTED._applied_carryover_balance</field>
                            </record>
                            <record id="account_tax_report_line_bf_credit_reported_balance_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_credit_asked" model="account.report.line">
                        <field name="name">22 VAT credit claimed for reimbursement</field>
                        <field name="code">BF_CREDIT_ASKED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_credit_asked_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_credit_not_asked" model="account.report.line">
                        <field name="name">23 VAT credit not claimed for reimbursement</field>
                        <field name="code">BF_CREDIT_NOT_ASKED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_credit_not_asked_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_irrecoverable" model="account.report.line">
                        <field name="name">24 VAT on unpaid sales or services (definitively irrecoverable debts)</field>
                        <field name="code">BF_IRRECOVERABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_irrecoverable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">BF_24</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_cancelled" model="account.report.line">
                        <field name="name">25 VAT paid on terminated or cancelled sales or services</field>
                        <field name="code">BF_CANCELLED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_cancelled_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BF_25</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_other_deduction" model="account.report.line">
                        <field name="name">26 Other deductions available to the company</field>
                        <field name="code">BF_OTHER_DEDUCTION</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_other_deduction_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BF_26</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bf_unpaid_credit_carried" model="account.report.line">
                        <field name="name">27 Unpaid VAT credit carried forward</field>
                        <field name="code">BF_UNPAID_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bf_unpaid_credit_carried_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BF_27</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_bf_net_vat_to_pay" model="account.report.line">
                <field name="name">Net VAT amount to pay [19 – (20+21-22+25+26+27)]</field>
                <field name="code">BF_NET_TO_PAY</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_bf_net_vat_to_pay_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">BF_GROSS_TOTAL.balance - (BF_DEDUCTIBLE.balance + BF_CREDIT_REPORTED.balance - BF_CREDIT_ASKED.balance + BF_CANCELLED.balance + BF_OTHER_DEDUCTION.balance + BF_UNPAID_CREDIT.balance)</field>
                        <field name="subformula">if_above(XOF(0))</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_bf_credit_to_report" model="account.report.line">
                <field name="name">Credit VAT to report [(20+21-22+25+26+27) - 19]</field>
                <field name="code">BF_CREDIT_TO_REPORT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_bf_credit_to_report_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">BF_DEDUCTIBLE.balance + BF_CREDIT_REPORTED.balance - BF_CREDIT_ASKED.balance + BF_CANCELLED.balance + BF_UNPAID_CREDIT.balance + BF_OTHER_DEDUCTION.balance - BF_GROSS_TOTAL.balance</field>
                        <field name="subformula">if_above(XOF(0))</field>
                        <field name="carryover_target" eval="False"/>
                    </record>
                    <record id="account_tax_report_line_bf_credit_to_report_carryover" model="account.report.expression">
                        <field name="label">_carryover_balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">BF_CREDIT_TO_REPORT.balance</field>
                        <field name="carryover_target">BF_CREDIT_REPORTED._applied_carryover_balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-bf.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.bf","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18","tva_export_0"
"","","","","","","","tva_sale_10","tva_export_0"
"","","","","","","","tva_purchase_18","tva_import_0"
"","","","","","","","tva_purchase_10","tva_import_0"

```

## File: data\template\account.fiscal.position-bf_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.bf","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_10","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_18","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_10","syscebnl_tva_import_0"

```

## File: data\template\account.tax-bf.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_18","18%","","","18.0","percent","sale","tax_group_18","base","invoice","","+BF_01||+BF_15_base","","",""
"","","","","","","","","tax","invoice","pcg_4431","+BF_15_tax","","",""
"","","","","","","","","base","refund","","-BF_01||-BF_15_base","","",""
"","","","","","","","","tax","refund","pcg_4431","+BF_25","","",""
"tva_sale_10","10%","10% services","","10.0","percent","sale","tax_group_10","base","invoice","","+BF_04||+BF_16_base","","",""
"","","","","","","","","tax","invoice","pcg_4431","+BF_16_tax","","",""
"","","","","","","","","base","refund","","-BF_04||-BF_16_base","","",""
"","","","","","","","","tax","refund","pcg_4431","+BF_25","","",""
"tva_sale_self_18","18% SD","18% Self-delivery","False","18.0","percent","sale","tax_group_18","base","invoice","","+BF_02||+BF_15_base","","18% LASM","18% Livraison à soi-même"
"","","","","","","","","tax","invoice","pcg_4431","-BF_15_tax","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+BF_20","","",""
"","","","","","","","","base","refund","","-BF_02||-BF_15_base","","",""
"","","","","","","","","tax","refund","pcg_4431","+BF_25","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-BF_20","","",""
"tva_purchase_18","18%","","","18.0","percent","purchase","tax_group_18","base","invoice","","","","",""
"","","","","","","","","tax","invoice","pcg_4452","+BF_20","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4452","-BF_20","","",""
"tva_purchase_10","10%","","","10.0","percent","purchase","tax_group_10","base","invoice","","","","",""
"","","","","","","","","tax","invoice","pcg_4452","+BF_20","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4452","-BF_20","","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","","+BF_11","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BF_11","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","","+BF_13","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BF_13","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-bf_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_18","18%","","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","","+BF_01||+BF_15_base","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+BF_15_tax","","",""
"","","","","","","","","base","refund","","-BF_01||-BF_15_base","","",""
"","","","","","","","","tax","refund","syscebnl_443","+BF_25","","",""
"syscebnl_tva_sale_10","10%","10% services","","10.0","percent","sale","syscebnl_tax_group_10","base","invoice","","+BF_04||+BF_16_base","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+BF_16_tax","","",""
"","","","","","","","","base","refund","","-BF_04||-BF_16_base","","",""
"","","","","","","","","tax","refund","syscebnl_443","+BF_25","","",""
"syscebnl_tva_sale_self_18","18% SD","18% Self-delivery","False","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","","+BF_02||+BF_15_base","","18% LASM","18% Livraison à soi-même"
"","","","","","","","","tax","invoice","syscebnl_443","-BF_15_tax","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+BF_20","","",""
"","","","","","","","","base","refund","","-BF_02||-BF_15_base","","",""
"","","","","","","","","tax","refund","syscebnl_443","+BF_25","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-BF_20","","",""
"syscebnl_tva_purchase_18","18%","","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","","","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+BF_20","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_445","-BF_20","","",""
"syscebnl_tva_purchase_10","10%","","","10.0","percent","purchase","syscebnl_tax_group_10","base","invoice","","","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+BF_20","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_445","-BF_20","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+BF_11","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BF_11","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+BF_13","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BF_13","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-bf.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_10","VAT 10%","T.V.A. 10%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-bf_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_10","VAT 10%","T.V.A. 10%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_bf.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bf')
    def _get_bf_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('bf', 'res.company')
    def _get_bf_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.bf',
                'account_sale_tax_id': 'tva_sale_18',
                'account_purchase_tax_id': 'tva_purchase_18',
            }
        )
        return company_values

    @template('bf', 'account.account')
    def _get_bf_account_account(self):
        return self._parse_csv('bf', 'account.account', module='l10n_syscohada')

```

## File: models\template_bf_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bf_syscebnl')
    def _get_bf_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('bf_syscebnl', 'res.company')
    def _get_bf_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.bf',
                'account_sale_tax_id': 'syscebnl_tva_sale_18',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_18',
            }
        )
        return company_values

    @template('bf_syscebnl', 'account.account')
    def _get_bf_syscebnl_account_account(self):
        return self._parse_csv('bf_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_bf
from . import template_bf_syscebnl

```

