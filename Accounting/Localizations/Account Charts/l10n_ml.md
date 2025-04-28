# Odoo Module: l10n_ml

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Mali - Accounting",
    'countries': ['ml'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Mali.
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
    <record id="account_tax_report_ml" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ml"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_ml_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_ml_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_ml_sales" model="account.report.line">
                <field name="name">Outgoing</field>
                <field name="code">ML_SALES</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_sales_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_TAXABLE_18.base + ML_TAXABLE_5.base + ML_TAXABLE_SD.base + ML_EXPORT.base + ML_SALE_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_ml_sales_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_TAXABLE_18.tax + ML_TAXABLE_5.tax + ML_TAXABLE_SD.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ml_sales_18" model="account.report.line">
                        <field name="name">Taxable operations at 18%</field>
                        <field name="code">ML_TAXABLE_18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_sales_18_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_18</field>
                            </record>
                            <record id="account_tax_report_line_ml_sales_18_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_18</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ml_sales_5" model="account.report.line">
                        <field name="name">Taxable operations at 5%</field>
                        <field name="code">ML_TAXABLE_5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_sales_5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_5</field>
                            </record>
                            <record id="account_tax_report_line_ml_sales_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_5</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ml_sales_sd" model="account.report.line">
                        <field name="name">Self Delivery</field>
                        <field name="code">ML_TAXABLE_SD</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_sales_sd_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">ML_TAXABLE_SD_18.base + ML_TAXABLE_SD_5.base</field>
                            </record>
                            <record id="account_tax_report_line_ml_sales_sd_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">ML_TAXABLE_SD_18.tax + ML_TAXABLE_SD_5.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_ml_sales_sd_18" model="account.report.line">
                                <field name="name">at 18%</field>
                                <field name="code">ML_TAXABLE_SD_18</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ml_sales_sd_18_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_lasm_18</field>
                                    </record>
                                    <record id="account_tax_report_line_ml_sales_sd_18_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_lasm_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_ml_sales_sd_5" model="account.report.line">
                                <field name="name">at 5%</field>
                                <field name="code">ML_TAXABLE_SD_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_ml_sales_sd_5_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_lasm_5</field>
                                    </record>
                                    <record id="account_tax_report_line_ml_sales_sd_5_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_lasm_5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ml_sales_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">ML_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_sales_export_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">export</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ml_sales_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">ML_SALE_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_sales_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_refund_regularisation" model="account.report.line">
                <field name="name">VAT refund following adjustment</field>
                <field name="code">ML_REFUND_REGU</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_refund_regularisation_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_gross_pay" model="account.report.line">
                <field name="name">Gross VAT to pay</field>
                <field name="code">ML_GROSS_PAY</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_gross_pay_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_REFUND_REGU.tax + ML_SALES.tax</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_deductible" model="account.report.line">
                <field name="name">Deductible</field>
                <field name="code">ML_DEDUCTIBLE</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_deductible_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_PURC_TAXABLE.base</field>
                    </record>
                    <record id="account_tax_report_line_ml_deductible_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_PURC_TAXABLE.tax + ML_WITHHELD.tax + ML_DEDU_ADJUSTMENT.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_ml_purchases_taxable" model="account.report.line">
                        <field name="name">Taxable</field>
                        <field name="code">ML_PURC_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_purchases_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_base</field>
                            </record>
                            <record id="account_tax_report_line_ml_purchases_taxable_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ml_purchases_withheld" model="account.report.line">
                        <field name="name">VAT withheld by clients</field>
                        <field name="code">ML_WITHHELD</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_purchases_withheld_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_ml_purchases_deduction_adjustment" model="account.report.line">
                        <field name="name">Additional deduction following adjustment</field>
                        <field name="code">ML_DEDU_ADJUSTMENT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_ml_purchases_deduction_adjustment_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_purchases_credit_reported" model="account.report.line">
                <field name="name">Credit to be carried forward from previous months</field>
                <field name="code">ML_CREDIT_REPORTED</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_purchases_credit_reported_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_CREDIT_REPORTED._applied_carryover_tax</field>
                    </record>
                    <record id="account_tax_report_line_ml_purchases_credit_reported_tax_carryover" model="account.report.expression">
                        <field name="label">_applied_carryover_tax</field>
                        <field name="engine">external</field>
                        <field name="formula">most_recent</field>
                        <field name="date_scope">previous_tax_period</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_deductions" model="account.report.line">
                <field name="name">Total deductions</field>
                <field name="code">ML_DEDUCTIONS</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_deductions_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_DEDUCTIBLE.tax + ML_CREDIT_REPORTED.tax</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_to_pay" model="account.report.line">
                <field name="name">Net VAT to pay</field>
                <field name="code">ML_TO_PAY</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_to_pay_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_SALES.tax - ML_DEDUCTIONS.tax</field>
                        <field name="subformula">if_above(XOF(0))</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_credit_report" model="account.report.line">
                <field name="name">VAT credit to report</field>
                <field name="code">ML_CREDIT_REPORT</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_credit_report_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_DEDUCTIONS.tax - ML_SALES.tax</field>
                        <field name="subformula">if_above(XOF(0))</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_reimbursement" model="account.report.line">
                <field name="name">Reimbursement Asked</field>
                <field name="code">ML_REIMBURSEMENT</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_reimbursement_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ml_credit_report_deductions" model="account.report.line">
                <field name="name">Credit to report coming from deductions</field>
                <field name="code">ML_CREDIT_REPROT_DEDU</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_ml_credit_report_deductions_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_CREDIT_REPORT.tax - ML_REIMBURSEMENT.tax</field>
                        <field name="subformula">if_above(XOF(0))</field>
                        <field name="carryover_target" eval="False"/>
                    </record>
                    <record id="account_tax_report_line_ml_credit_report_deductions_carryover" model="account.report.expression">
                        <field name="label">_carryover_tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">ML_CREDIT_REPROT_DEDU.tax</field>
                        <field name="carryover_target">ML_CREDIT_REPORTED._applied_carryover_tax</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-ml.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.ml","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18","tva_export_0"
"","","","","","","","tva_sale_5","tva_export_0"
"","","","","","","","tva_purchase_18","tva_import_18"
"","","","","","","","tva_purchase_5","tva_import_5"

```

## File: data\template\account.fiscal.position-ml_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.ml","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_5","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_18","syscebnl_tva_import_18"
"","","","","","","","syscebnl_tva_purchase_5","syscebnl_tva_import_5"

```

## File: data\template\account.tax-ml.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_18","18%","","","18.0","percent","sale","tax_group_18","base","invoice","","+base_18","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_18","","",""
"","","","","","","","","base","refund","","-base_18","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_18","","",""
"tva_purchase_18","18%","","","18.0","percent","purchase","tax_group_18","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_sale_5","5%","","","5.0","percent","sale","tax_group_5","base","invoice","","+base_5","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_5","","",""
"","","","","","","","","base","refund","","-base_5","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_5","","",""
"tva_purchase_5","5%","","","5.0","percent","purchase","tax_group_5","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-export","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_import_18","18% EX","18% (import)","","18.0","","purchase","tax_group_18","base","invoice","","+base_18||+purc_base","","","18% (importation)"
"","","","","","","","","tax","invoice","pcg_4431","-tax_18","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_18","","",""
"","","","","","","","","tax","refund","pcg_4431","+tax_18","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_self_delivery_18","18% SD","18% (self delivery)","False","18.0","","purchase","tax_group_18","base","invoice","","+base_lasm_18||+purc_base","","18% LASM","18% (livraison à soi-même)"
"","","","","","","","","tax","invoice","pcg_4431","-tax_lasm_18","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_lasm_18","","",""
"","","","","","","","","tax","refund","pcg_4431","+tax_lasm_18","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_self_delivery_5","5% SD","5% (self delivery)","False","5.0","","purchase","tax_group_5","base","invoice","","+base_lasm_5||+purc_base","","5% LASM","5% (livraison à soi-même)"
"","","","","","","","","tax","invoice","pcg_4431","-tax_lasm_5","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_lasm_5","","",""
"","","","","","","","","tax","refund","pcg_4431","+tax_lasm_5","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_import_5","5% EX","5% (import)","","5.0","","purchase","tax_group_5","base","invoice","","+base_5||+purc_base","","","5% (importation)"
"","","","","","","","","tax","invoice","pcg_4431","-tax_5","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_5","","",""
"","","","","","","","","tax","refund","pcg_4431","+tax_5","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-ml_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_18","18%","","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","","+base_18","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_18","","",""
"","","","","","","","","base","refund","","-base_18","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_18","","",""
"syscebnl_tva_purchase_18","18%","","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_sale_5","5%","","","5.0","percent","sale","syscebnl_tax_group_5","base","invoice","","+base_5","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_5","","",""
"","","","","","","","","base","refund","","-base_5","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_5","","",""
"syscebnl_tva_purchase_5","5%","","","5.0","percent","purchase","syscebnl_tax_group_5","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-export","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_18","18% EX","18% (import)","","18.0","","purchase","syscebnl_tax_group_18","base","invoice","","+base_18||+purc_base","","","18% (importation)"
"","","","","","","","","tax","invoice","syscebnl_443","-tax_18","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_18","","",""
"","","","","","","","","tax","refund","syscebnl_443","+tax_18","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_self_delivery_18","18% SD","18% (self delivery)","False","18.0","","purchase","syscebnl_tax_group_18","base","invoice","","+base_lasm_18||+purc_base","","18% LASM","18% (livraison à soi-même)"
"","","","","","","","","tax","invoice","syscebnl_443","-tax_lasm_18","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_lasm_18","","",""
"","","","","","","","","tax","refund","syscebnl_443","+tax_lasm_18","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_self_delivery_5","5% SD","5% (self delivery)","False","5.0","","purchase","syscebnl_tax_group_5","base","invoice","","+base_lasm_5||+purc_base","","5% LASM","5% (livraison à soi-même)"
"","","","","","","","","tax","invoice","syscebnl_443","-tax_lasm_5","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_lasm_5","","",""
"","","","","","","","","tax","refund","syscebnl_443","+tax_lasm_5","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_import_5","5% EX","5% (import)","","5.0","","purchase","syscebnl_tax_group_5","base","invoice","","+base_5||+purc_base","","","5% (importation)"
"","","","","","","","","tax","invoice","syscebnl_443","-tax_5","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-base_5","","",""
"","","","","","","","","tax","refund","syscebnl_443","+tax_5","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-ml.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_5","VAT 5%","T.V.A. 5%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-ml_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_5","VAT 5%","T.V.A. 5%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_ml.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ml')
    def _get_ml_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('ml', 'res.company')
    def _get_ml_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.ml',
                'account_sale_tax_id': 'tva_sale_18',
                'account_purchase_tax_id': 'tva_purchase_18',
            }
        )
        return company_values

    @template('ml', 'account.account')
    def _get_ml_account_account(self):
        return self._parse_csv('ml', 'account.account', module='l10n_syscohada')

```

## File: models\template_ml_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ml_syscebnl')
    def _get_ml_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('ml_syscebnl', 'res.company')
    def _get_ml_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.ml',
                'account_sale_tax_id': 'syscebnl_tva_sale_18',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_18',
            }
        )
        return company_values

    @template('ml_syscebnl', 'account.account')
    def _get_ml_syscebnl_account_account(self):
        return self._parse_csv('ml_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_ml
from . import template_ml_syscebnl

```

