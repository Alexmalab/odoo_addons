# Odoo Module: l10n_cm

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Cameroon - Accounting',
    'countries': ['cm'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Cameroon.
===========================================================

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
    <record id="account_tax_report_cm" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.cm"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_cm_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_cm_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_cm_turnover" model="account.report.line">
                <field name="name">2-Turnover achieved</field>
                <field name="code">CM_TURNOVER</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cm_turnover_normal" model="account.report.line">
                        <field name="name">10. Taxable operations at normal rate</field>
                        <field name="code">CM_NORMAL</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_turnover_normal_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_10_base</field>
                            </record>
                            <record id="account_tax_report_line_cm_turnover_normal_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_10_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_turnover_excises" model="account.report.line">
                        <field name="name">11. Amount of Excise Duty</field>
                        <field name="code">CM_EXCISE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_turnover_excises_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="account_tax_report_line_cm_turnover_excises_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_turnover_other" model="account.report.line">
                        <field name="name">12. Other taxable operations</field>
                        <field name="code">CM_OTHER</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_turnover_other_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_12_base</field>
                            </record>
                            <record id="account_tax_report_line_cm_turnover_other_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_12_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_turnover_export" model="account.report.line">
                        <field name="name">13. Exports</field>
                        <field name="code">CM_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_turnover_export_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_13</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_turnover_exempt" model="account.report.line">
                        <field name="name">14. Exempted turnover</field>
                        <field name="code">CM_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_turnover_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_14</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_turnover_global" model="account.report.line">
                        <field name="name">15. Global turnover</field>
                        <field name="code">CM_GLOBAL</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_turnover_global_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_NORMAL.base + CM_EXCISE.base + CM_OTHER.base + CM_EXPORT.base + CM_EXEMPT.base</field>
                            </record>
                            <record id="account_tax_report_line_cm_turnover_global_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_NORMAL.tax + CM_EXCISE.tax + CM_OTHER.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cm_deductible" model="account.report.line">
                <field name="name">3-Deductible VAT</field>
                <field name="code">CM_DEDUCTIBLE</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cm_credit_report" model="account.report.line">
                        <field name="name">17. Previous credit's report</field>
                        <field name="code">CM_CREDIT_REPORTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_credit_report_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_CREDIT_REPORTED._applied_carryover_tax</field>
                            </record>
                            <record id="account_tax_report_line_cm_credit_report_tax_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_credit_deductible_local_purchase" model="account.report.line">
                        <field name="name">18. Deductible VAT on local purchases</field>
                        <field name="code">CM_LOCAL_PURCHASE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_credit_deductible_local_purchase_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_18</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_credit_deductible_local_service" model="account.report.line">
                        <field name="name">19. Deductible VAT on local services</field>
                        <field name="code">CM_LOCAL_SERVICE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_credit_deductible_local_service_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_19</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_credit_deductible_foreign_purchase" model="account.report.line">
                        <field name="name">20. Deductible VAT on foreign purchases</field>
                        <field name="code">CM_FOREIGN_PURCHASE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_credit_deductible_foreign_purchase_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_20</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_credit_deductible_foreign_service" model="account.report.line">
                        <field name="name">21. Deductible VAT on services and other remuneration paid abroad</field>
                        <field name="code">CM_FOREIGN_SERVICE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_credit_deductible_foreign_service_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CM_21</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_credit_deductible_total" model="account.report.line">
                        <field name="name">22. Total VAT deductible (L17+L18+L19+L20+L21)</field>
                        <field name="code">CM_DEDUCTIBLE_VAT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_credit_deductible_total_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_CREDIT_REPORTED.tax + CM_LOCAL_PURCHASE.tax + CM_LOCAL_SERVICE.tax + CM_FOREIGN_PURCHASE.tax + CM_FOREIGN_SERVICE.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cm_adjustment" model="account.report.line">
                <field name="name">4-Exceptional adjustments</field>
                <field name="code">CM_ADJUSTMENTS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cm_adjustment_deductible" model="account.report.line">
                        <field name="name">24. Adjustment of deductible VAT or VAT already withheld at source</field>
                        <field name="code">CM_ADJUSTMENT_DEDUCTIBLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_adjustment_deductible_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_adjustment_state" model="account.report.line">
                        <field name="name">25. Adjustment of VAT borne by the State</field>
                        <field name="code">CM_ADJUSTMENT_STATE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_adjustment_state_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_adjustment_fixed" model="account.report.line">
                        <field name="name">26. Adjustment on disposal of fixed assets to be repaid</field>
                        <field name="code">CM_ADJUSTMENT_FIXED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_adjustment_fixed_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_adjustment_other" model="account.report.line">
                        <field name="name">27. Adjustment of VAT to be repaid and others</field>
                        <field name="code">CM_ADJUSTMENT_OTHER</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_adjustment_other_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cm_to_pay" model="account.report.line">
                <field name="name">5-VAT payable or VAT credit</field>
                <field name="code">CM_TO_PAY</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cm_collected" model="account.report.line">
                        <field name="name">28. Collected VAT (L10+L11+L12)</field>
                        <field name="code">CM_COLLECTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_collected_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_GLOBAL.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_deductible_vat" model="account.report.line">
                        <field name="name">29. Deductible VAT (L22)</field>
                        <field name="code">CM_DEDUCTIBLE_VAT_29</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_deductible_vat_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_DEDUCTIBLE_VAT.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_adjustment_to_deduct" model="account.report.line">
                        <field name="name">30. Adjustment of VAT to be deducted (L24+L25)</field>
                        <field name="code">CM_ADJUSTMENT_TO_DEDUCT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_adjustment_to_deduct_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_ADJUSTMENT_DEDUCTIBLE.tax + CM_ADJUSTMENT_STATE.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_adjustment_to_pay" model="account.report.line">
                        <field name="name">31. VAT adjustment to be repaid (L26+L27)</field>
                        <field name="code">CM_ADJUSTMENT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_adjustment_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_ADJUSTMENT_FIXED.tax + CM_ADJUSTMENT_OTHER.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_vat_to_pay" model="account.report.line">
                        <field name="name">32. VAT to pay (L28-L29-L30+L31)</field>
                        <field name="code">CM_VAT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_vat_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_COLLECTED.tax - CM_DEDUCTIBLE_VAT_29.tax - CM_ADJUSTMENT_TO_DEDUCT.tax + CM_ADJUSTMENT_TO_PAY.tax</field>
                                <field name="subformula">if_above(XAF(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_vat_credit" model="account.report.line">
                        <field name="name">33. VAT credit</field>
                        <field name="code">CM_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_vat_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_DEDUCTIBLE_VAT_29.tax + CM_ADJUSTMENT_TO_DEDUCT.tax - CM_COLLECTED.tax - CM_ADJUSTMENT_TO_PAY.tax</field>
                                <field name="subformula">if_above(XAF(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_reimbursement" model="account.report.line">
                        <field name="name">34. Reimbursement asked</field>
                        <field name="code">CM_REIMBURSEMENT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_reimbursement_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cm_credit_to_report" model="account.report.line">
                        <field name="name">35. Credit to report (L33-L34)</field>
                        <field name="code">CM_CREDIT_REPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cm_vat_credit_to_report_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_CREDIT.tax - CM_REIMBURSEMENT.tax</field>
                                <field name="subformula">if_above(XAF(0))</field>
                                <field name="carryover_target" eval="False"/>
                            </record>
                            <record id="account_tax_report_line_cm_vat_credit_to_report_carryover" model="account.report.expression">
                                <field name="label">_carryover_tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CM_CREDIT_REPORT.tax</field>
                                <field name="carryover_target">CM_CREDIT_REPORTED._applied_carryover_tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-cm.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.cm","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_19_25","tva_export_0"
"","","","","","","","tva_purchase_good_19_25","tva_import_0"
"","","","","","","","tva_purchase_services_19_25","tva_import_0"

```

## File: data\template\account.fiscal.position-cm_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.cm","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_19_25","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_good_19_25","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_services_19_25","syscebnl_tva_import_0"

```

## File: data\template\account.tax-cm.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"tva_sale_19_25","19.25%","","True","","19.25","percent","sale","tax_group_19_25","base","invoice","+CM_10_base","","","",""
"","","","","","","","","","tax","invoice","+CM_10_tax","pcg_4431","","",""
"","","","","","","","","","base","refund","-CM_10_base","","","",""
"","","","","","","","","","tax","refund","-CM_10_tax","pcg_4431","","",""
"tva_purchase_good_19_25","19.25% G","19.25% local goods","True","","19.25","percent","purchase","tax_group_19_25","base","invoice","","","","19.25% sur les achats locaux",""
"","","","","","","","","","tax","invoice","+CM_18","pcg_4452","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CM_18","pcg_4452","","",""
"tva_purchase_services_19_25","19.25% S","19.25% on services","True","","19.25","percent","purchase","tax_group_19_25","base","invoice","","","","19.25% sur les services",""
"","","","","","","","","","tax","invoice","+CM_19","pcg_4452","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CM_19","pcg_4452","","",""
"tva_export_0","0% EX","0% (export)","True","","0.0","","sale","tax_group_0","base","invoice","+CM_13","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CM_13","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","True","","0.0","","purchase","tax_group_0","base","invoice","","","","0% (importation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","tax_group_0","base","invoice","+CM_14","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CM_14","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","True","","0.0","","purchase","tax_group_0","base","invoice","","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-cm_syscebnl.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"syscebnl_tva_sale_19_25","19.25%","","True","","19.25","percent","sale","syscebnl_tax_group_19_25","base","invoice","+CM_10_base","","","",""
"","","","","","","","","","tax","invoice","+CM_10_tax","syscebnl_443","","",""
"","","","","","","","","","base","refund","-CM_10_base","","","",""
"","","","","","","","","","tax","refund","-CM_10_tax","syscebnl_443","","",""
"syscebnl_tva_purchase_good_19_25","19.25% G","19.25% local goods","True","","19.25","percent","purchase","syscebnl_tax_group_19_25","base","invoice","","","","19.25% sur les achats locaux",""
"","","","","","","","","","tax","invoice","+CM_18","syscebnl_445","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CM_18","syscebnl_445","","",""
"syscebnl_tva_purchase_services_19_25","19.25% S","19.25% on services","True","","19.25","percent","purchase","syscebnl_tax_group_19_25","base","invoice","","","","19.25% sur les services",""
"","","","","","","","","","tax","invoice","+CM_19","syscebnl_445","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-CM_19","syscebnl_445","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","True","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+CM_13","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CM_13","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","True","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","0% (importation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+CM_14","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-CM_14","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","True","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-cm.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_19_25","VAT 19.25%","T.V.A. 19.25%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-cm_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_19_25","VAT 19.25%","T.V.A. 19.25%","syscebnl_443","syscebnl_445"

```

## File: models\template_cm.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cm')
    def _get_cm_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('cm', 'res.company')
    def _get_cm_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cm',
                'account_sale_tax_id': 'tva_sale_19_25',
                'account_purchase_tax_id': 'tva_purchase_good_19_25',
            }
        )
        return company_values

    @template('cm', 'account.account')
    def _get_cm_account_account(self):
        return self._parse_csv('cm', 'account.account', module='l10n_syscohada')

```

## File: models\template_cm_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cm_syscebnl')
    def _get_cm_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('cm_syscebnl', 'res.company')
    def _get_cm_syscebnl_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cm',
                'account_sale_tax_id': 'syscebnl_tva_sale_19_25',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_good_19_25',
            }
        )
        return company_values

    @template('cm_syscebnl', 'account.account')
    def _get_cm_syscebnl_account_account(self):
        return self._parse_csv('cm_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_cm
from . import template_cm_syscebnl

```

