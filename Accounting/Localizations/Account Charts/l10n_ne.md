# Odoo Module: l10n_ne

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Niger - Accounting",
    'countries': ['ne'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Niger.
=================================================================

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
    <record id="account_tax_report_ne" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ne"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_ne_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_ne_turnover" model="account.report.line">
                <field name="name">I. Turnover (Without Tax)</field>
                <field name="code">NE_TURNOVER</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ne_sales_goods" model="account.report.line">
                        <field name="name">1. Goods sales</field>
                        <field name="code">NE_GOODS</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_sales_goods_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_sales_service" model="account.report.line">
                        <field name="name">2. Service sales</field>
                        <field name="code">NE_SERVICE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_sales_service_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_2</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_sales_self" model="account.report.line">
                        <field name="name">3. Self Deliveries</field>
                        <field name="code">NE_SELF</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_sales_self_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_3</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_sales_imposable" model="account.report.line">
                        <field name="name">4. Taxable turnover (1+2+3)</field>
                        <field name="code">NE_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_sales_imposable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">NE_GOODS.balance + NE_SERVICE.balance + NE_SELF.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_sales_export" model="account.report.line">
                        <field name="name">5. Export turnover</field>
                        <field name="code">NE_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_sales_export_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_5</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_sales_exempt" model="account.report.line">
                        <field name="name">6. Other exempted turnover</field>
                        <field name="code">NE_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_sales_exempt_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_6</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_sales_total_turnover" model="account.report.line">
                        <field name="name">7. Total turnover (4+5+6)</field>
                        <field name="code">NE_TOTAL_TURNOVER</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_sales_total_turnover_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">NE_TAXABLE.balance + NE_EXPORT.balance + NE_EXEMPT.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ne_gross" model="account.report.line">
                <field name="name">II. Gross VAT </field>
                <field name="code">NE_GROSS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ne_gross_vat" model="account.report.line">
                        <field name="name">8. Gross VAT (line 4x19%)</field>
                        <field name="code">NE_GROSS_VAT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_gross_vat_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_8</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ne_deductible" model="account.report.line">
                <field name="name">III. Deductible VAT</field>
                <field name="code">NE_DEDUCTIBLE</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ne_deductible_imported_invest" model="account.report.line">
                        <field name="name">10. VAT on imported investments</field>
                        <field name="code">NE_IMPORT_INVEST</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_imported_invest_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_deductible_local_invest" model="account.report.line">
                        <field name="name">11. VAT on local investments</field>
                        <field name="code">NE_LOCAL_INVEST</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_local_invest_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_11</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_deductible_imported_goods_services" model="account.report.line">
                        <field name="name">12. VAT on imported goods and services</field>
                        <field name="code">NE_IMPORT_GOODS_SERVICES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_imported_goods_services_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_12</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_deductible_local_goods_services" model="account.report.line">
                        <field name="name">13. VAT on local goods and services</field>
                        <field name="code">NE_LOCAL_GOODS_SERVICES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_local_goods_services_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_13</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_deductible_withheld" model="account.report.line">
                        <field name="name">14. VAT withheld by customers</field>
                        <field name="code">NE_WITHHELD</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_withheld_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_14</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_deductible_reported" model="account.report.line">
                        <field name="name">15. Credit reported (line 21 of last month)</field>
                        <field name="code">NE_REPORTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_reported_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">NE_REPORTED._applied_carryover_balance</field>
                            </record>
                            <record id="account_tax_report_line_ne_deductible_reported_balance_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_deductible_self" model="account.report.line">
                        <field name="name">16. VAT on self deliveries</field>
                        <field name="code">NE_VAT_SELF_DELIVERIES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_self_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NE_16</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_deductible_total" model="account.report.line">
                        <field name="name">17. Total deductible VAT (10 to 16)</field>
                        <field name="code">NE_DEDUCTIBLE_TOTAL</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_deductible_total_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">NE_IMPORT_INVEST.balance + NE_LOCAL_INVEST.balance + NE_IMPORT_GOODS_SERVICES.balance + NE_LOCAL_GOODS_SERVICES.balance + NE_WITHHELD.balance + NE_REPORTED.balance + NE_VAT_SELF_DELIVERIES.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ne_net_vat" model="account.report.line">
                <field name="name">IV. Net VAT</field>
                <field name="code">NE_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ne_net_regularisation" model="account.report.line">
                        <field name="name">Regularisation</field>
                        <field name="code">NE_REGULARISATION</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_ne_net_deductible_addition" model="account.report.line">
                                <field name="name">18. - Additional deductible vat</field>
                                <field name="code">NE_DEDUCTIBLE_ADDITION</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ne_net_deductible_addition_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ne_net_repay" model="account.report.line">
                                <field name="name">19. - VAT to repay</field>
                                <field name="code">NE_REPAY</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ne_net_repay_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_net_to_pay" model="account.report.line">
                        <field name="name">20. Net VAT to pay (8+19-17-18)</field>
                        <field name="code">NE_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_net_to_pay_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="subformula">if_above(XOF(0))</field>
                                <field name="formula">NE_GROSS_VAT.balance + NE_REPAY.balance - NE_DEDUCTIBLE_TOTAL.balance - NE_DEDUCTIBLE_ADDITION.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ne_credit_to_report" model="account.report.line">
                        <field name="name">21. Credit VAT to report (17+18-8-19)</field>
                        <field name="code">NE_CREDIT_TO_REPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ne_credit_to_report_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="subformula">if_above(XOF(0))</field>
                                <field name="formula">NE_DEDUCTIBLE_TOTAL.balance + NE_DEDUCTIBLE_ADDITION.balance - NE_GROSS_VAT.balance - NE_REPAY.balance</field>
                                <field name="carryover_target" eval="False"/>
                            </record>
                            <record id="account_tax_report_line_ne_credit_to_report_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">NE_CREDIT_TO_REPORT.balance</field>
                                <field name="carryover_target">NE_REPORTED._applied_carryover_balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-ne.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.ne","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_19","tva_export_0"
"","","","","","","","tva_sale_service_19","tva_export_0"
"","","","","","","","tva_purchase_19","tva_import_19"
"","","","","","","","tva_purchase_invest_19","tva_import_invest_19"

```

## File: data\template\account.fiscal.position-ne_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.ne","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_19","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_service_19","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_19","syscebnl_tva_import_19"
"","","","","","","","syscebnl_tva_purchase_invest_19","syscebnl_tva_import_invest_19"

```

## File: data\template\account.tax-ne.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_19","19% G","19% goods","","19.0","percent","sale","tax_group_19","base","invoice","","+NE_1","","","19% biens"
"","","","","","","","","tax","invoice","pcg_4431","+NE_8","","",""
"","","","","","","","","base","refund","","-NE_1","","",""
"","","","","","","","","tax","refund","pcg_4431","-NE_8","","",""
"tva_sale_service_19","19% S","19% services","","19.0","percent","sale","tax_group_19","base","invoice","","+NE_2","","",""
"","","","","","","","","tax","invoice","pcg_4431","+NE_8","","",""
"","","","","","","","","base","refund","","-NE_2","","",""
"","","","","","","","","tax","refund","pcg_4431","-NE_8","","",""
"tva_sale_self_19","19% SD","19% Self-delivery","","19.0","percent","sale","tax_group_19","base","invoice","","+NE_3","","19% LASM","19% Livraison à soi-même"
"","","","","","","","","tax","invoice","pcg_4431","-NE_8","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+NE_16","","",""
"","","","","","","","","base","refund","","-NE_3","","",""
"","","","","","","","","tax","refund","pcg_4431","+NE_8","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-NE_16","","",""
"tva_purchase_19","19%","","","19.0","percent","purchase","tax_group_19","base","invoice","","","","",""
"","","","","","","","","tax","invoice","pcg_4452","+NE_13","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4452","-NE_13","","",""
"tva_purchase_invest_19","19% INV","19% (local invest purchases)","","19.0","percent","purchase","tax_group_19","base","invoice","","","","","19% (achats d'investissements locaux)"
"","","","","","","","","tax","invoice","pcg_4452","+NE_11","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4452","-NE_11","","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","","+NE_5","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-NE_5","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_import_19","19% EX","19% (import)","","19.0","","purchase","tax_group_19","base","invoice","","","","","19% (importation)"
"","","","","","","","","tax","invoice","pcg_4431","-NE_8","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+NE_12","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4431","+NE_8","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-NE_12","","",""
"tva_import_invest_19","19% INV EX","19% (import of invest purchases)","","19.0","","purchase","tax_group_19","base","invoice","","","","","19% (importation d'achats d'investissements)"
"","","","","","","","","tax","invoice","pcg_4431","-NE_8","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+NE_10","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4431","+NE_8","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-NE_10","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","","+NE_6","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-NE_6","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-ne_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_19","19% G","19% goods","","19.0","percent","sale","syscebnl_tax_group_19","base","invoice","","+NE_1","","","19% biens"
"","","","","","","","","tax","invoice","syscebnl_443","+NE_8","","",""
"","","","","","","","","base","refund","","-NE_1","","",""
"","","","","","","","","tax","refund","syscebnl_443","-NE_8","","",""
"syscebnl_tva_sale_service_19","19% S","19% services","","19.0","percent","sale","syscebnl_tax_group_19","base","invoice","","+NE_2","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+NE_8","","",""
"","","","","","","","","base","refund","","-NE_2","","",""
"","","","","","","","","tax","refund","syscebnl_443","-NE_8","","",""
"syscebnl_tva_sale_self_19","19% SD","19% Self-delivery","","19.0","percent","sale","syscebnl_tax_group_19","base","invoice","","+NE_3","","19% LASM","19% Livraison à soi-même"
"","","","","","","","","tax","invoice","syscebnl_443","-NE_8","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+NE_16","","",""
"","","","","","","","","base","refund","","-NE_3","","",""
"","","","","","","","","tax","refund","syscebnl_443","+NE_8","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-NE_16","","",""
"syscebnl_tva_purchase_19","19%","","","19.0","percent","purchase","syscebnl_tax_group_19","base","invoice","","","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+NE_13","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_445","-NE_13","","",""
"syscebnl_tva_purchase_invest_19","19% INV","19% (local invest purchases)","","19.0","percent","purchase","syscebnl_tax_group_19","base","invoice","","","","","19% (achats d'investissements locaux)"
"","","","","","","","","tax","invoice","syscebnl_445","+NE_11","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_445","-NE_11","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+NE_5","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-NE_5","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_19","19% EX","19% (import)","","19.0","","purchase","syscebnl_tax_group_19","base","invoice","","","","","19% (importation)"
"","","","","","","","","tax","invoice","syscebnl_443","-NE_8","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+NE_12","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_443","+NE_8","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-NE_12","","",""
"syscebnl_tva_import_invest_19","19% INV EX","19% (import of invest purchases)","","19.0","","purchase","syscebnl_tax_group_19","base","invoice","","","","","19% (importation d'achats d'investissements)"
"","","","","","","","","tax","invoice","syscebnl_443","-NE_8","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+NE_10","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_443","+NE_8","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-NE_10","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+NE_6","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-NE_6","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-ne.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_19","VAT 19%","T.V.A. 19%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-ne_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_19","VAT 19%","T.V.A. 19%","syscebnl_443","syscebnl_445"

```

## File: models\template_ne.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ne')
    def _get_ne_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('ne', 'res.company')
    def _get_ne_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.ne',
                'account_sale_tax_id': 'tva_sale_19',
                'account_purchase_tax_id': 'tva_purchase_19',
            }
        )
        return company_values

    @template('ne', 'account.account')
    def _get_ne_account_account(self):
        return self._parse_csv('ne', 'account.account', module='l10n_syscohada')

```

## File: models\template_ne_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ne_syscebnl')
    def _get_ne_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('ne_syscebnl', 'res.company')
    def _get_ne_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.ne',
                'account_sale_tax_id': 'syscebnl_tva_sale_19',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_19',
            }
        )
        return company_values

    @template('ne_syscebnl', 'account.account')
    def _get_ne_syscebnl_account_account(self):
        return self._parse_csv('ne_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_ne
from . import template_ne_syscebnl

```

