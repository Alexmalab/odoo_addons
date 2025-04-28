# Odoo Module: l10n_sn

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Sénégal - Accounting",
    'countries': ['sn'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the taxes for Sénégal.
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
    <record id="account_tax_report_sn" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.sn"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_sn_base" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_sn_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_sn_operations" model="account.report.line">
                <field name="name">Total amount of operations</field>
                <field name="code">SN_OPE</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_sn_operations_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">SN_Export.base + SN_NON_TAXED.base + SN_SUSPENSION.base + SN_EXEMPT.base + SN_SELF.base + SN_TAXABLE.base</field>
                    </record>
                    <record id="account_tax_report_line_sn_operations_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">SN_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_sn_exportation" model="account.report.line">
                        <field name="name">Exportations</field>
                        <field name="code">SN_Export</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_exportation_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">export</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_non_taxed" model="account.report.line">
                        <field name="name">Non Taxed Domestic Operation</field>
                        <field name="code">SN_NON_TAXED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_non_taxed_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">non_taxed</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_suspension" model="account.report.line">
                        <field name="name">Operation done under suspension of VAT</field>
                        <field name="code">SN_SUSPENSION</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_suspension_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">suspension</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_exempt" model="account.report.line">
                        <field name="name">Operations exempted</field>
                        <field name="code">SN_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_exempt</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_self_delivery" model="account.report.line">
                        <field name="name">Self delivery or service</field>
                        <field name="code">SN_SELF</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_self_delivery_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_self</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_taxable" model="account.report.line">
                        <field name="name">Taxable operations</field>
                        <field name="code">SN_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SN_TAXABLE_18.base + SN_TAXABLE_10.base</field>
                            </record>
                            <record id="account_tax_report_line_sn_taxable_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SN_TAXABLE_18.tax + SN_TAXABLE_10.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_sn_taxable_18" model="account.report.line">
                                <field name="name">Taxable - normal rate</field>
                                <field name="code">SN_TAXABLE_18</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_sn_taxable_18_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_sale_18</field>
                                    </record>
                                    <record id="account_tax_report_line_sn_taxable_18_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_sale_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_sn_taxable_10" model="account.report.line">
                                <field name="name">Taxable - reduced rate</field>
                                <field name="code">SN_TAXABLE_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_sn_taxable_10_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_sale_10</field>
                                    </record>
                                    <record id="account_tax_report_line_sn_taxable_10_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_sale_10</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_sn_deductible" model="account.report.line">
                <field name="name">Deductible</field>
                <field name="code">SN_VAT_DEDUCT</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_sn_deductible_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">SN_PREPAY.base + SN_IMPORT.base + SN_DOMESTIC.base</field>
                    </record>
                    <record id="account_tax_report_line_sn_deductible_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">SN_PREPAY.tax + SN_IMPORT.tax + SN_DOMESTIC.tax + SN_REIMBURSEMENT_ACCEPTED.tax + SN_CREDIT.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_sn_prepayment" model="account.report.line">
                        <field name="name">Tax Prepayment</field>
                        <field name="code">SN_PREPAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_prepayment_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SN_WITHHOLDING.base</field>
                            </record>
                            <record id="account_tax_report_line_sn_prepayment_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SN_WITHHOLDING.tax + SN_DDI.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_sn_withholding" model="account.report.line">
                                <field name="name">Withholding</field>
                                <field name="code">SN_WITHHOLDING</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_sn_withholding_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_withholding</field>
                                    </record>
                                    <record id="account_tax_report_line_sn_withholding_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_withholding</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_sn_ddi" model="account.report.line">
                                <field name="name">DDI checks</field>
                                <field name="code">SN_DDI</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_sn_ddi_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ddi</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_deductible_import" model="account.report.line">
                        <field name="name">Importations</field>
                        <field name="code">SN_IMPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_deductible_import_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_import</field>
                            </record>
                            <record id="account_tax_report_line_sn_deductible_import_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_import</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_deductible_taxable" model="account.report.line">
                        <field name="name">Domestic purchases</field>
                        <field name="code">SN_DOMESTIC</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_deductible_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_purchase</field>
                            </record>
                            <record id="account_tax_report_line_sn_deductible_taxable_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_purchase</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_reimbursement_accepted" model="account.report.line">
                        <field name="name">Reimbursements accepted</field>
                        <field name="code">SN_REIMBURSEMENT_ACCEPTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_reimbursement_accepted_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sn_tva_credit" model="account.report.line">
                        <field name="name">Last month's credit</field>
                        <field name="code">SN_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sn_tva_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SN_CREDIT._applied_carryover_tax</field>
                            </record>
                            <record id="account_tax_report_line_sn_tva_credit_tax_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="account_tax_report_line_sn_due" model="account.report.line">
                <field name="name">Balance due</field>
                <field name="code">SN_DUE</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_sn_due_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">SN_OPE.tax - SN_VAT_DEDUCT.tax</field>
                        <field name="subformula">if_above(XOF(0))</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_sn_to_report" model="account.report.line">
                <field name="name">Credit to report</field>
                <field name="code">SN_REPORT</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_sn_to_report_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">SN_VAT_DEDUCT.tax - SN_OPE.tax</field>
                        <field name="subformula">if_above(XOF(0))</field>
                        <field name="carryover_target" eval="False"/>
                    </record>
                    <record id="account_tax_report_line_sn_to_report_carryover" model="account.report.expression">
                        <field name="label">_carryover_tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">SN_REPORT.tax</field>
                        <field name="carryover_target">SN_CREDIT._applied_carryover_tax</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_sn_reimbursement_review" model="account.report.line">
                <field name="name">Reimbursement under review</field>
                <field name="code">SN_REIMBURSEMENT_REVIEW</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_sn_reimbursement_review_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-sn.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.td","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18","tva_export_0"
"","","","","","","","tva_sale_10","tva_export_0"
"","","","","","","","tva_purchase_18","tva_import_0"
"","","","","","","","tva_purchase_10","tva_import_0"

```

## File: data\template\account.fiscal.position-sn_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.td","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_10","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_18","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_10","syscebnl_tva_import_0"

```

## File: data\template\account.tax-sn.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_18","18%","","","18.0","percent","sale","tax_group_18","base","invoice","","+base_sale_18","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_sale_18","","",""
"","","","","","","","","base","refund","","-base_sale_18","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_sale_18","","",""
"tva_purchase_18","18%","","","18.0","percent","purchase","tax_group_18","base","invoice","","+base_purchase","","",""
"","","","","","","","","tax","invoice","pcg_4452","+tax_purchase","","",""
"","","","","","","","","base","refund","","-base_purchase","","",""
"","","","","","","","","tax","refund","pcg_4452","-tax_purchase","","",""
"tva_sale_10","10%","","","10.0","percent","sale","tax_group_10","base","invoice","","+base_sale_10","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_sale_10","","",""
"","","","","","","","","base","refund","","-base_sale_10","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_sale_10","","",""
"tva_purchase_10","10%","","","10.0","percent","purchase","tax_group_10","base","invoice","","+base_purchase","","",""
"","","","","","","","","tax","invoice","pcg_4452","+tax_purchase","","",""
"","","","","","","","","base","refund","","-base_purchase","","",""
"","","","","","","","","tax","refund","pcg_4452","-tax_purchase","","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-export","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_non_impos_0","0% NT","0% (non taxable)","","0.0","","sale","tax_group_0","base","invoice","","+non_taxed","","","0% (non taxable)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-non_taxed","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_sale_18_sd","0% SD","0% Self Delivery","","0.0","percent","sale","tax_group_0","base","invoice","","+sale_self","","0% Livraison à soi-meme","0 LASM"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_self","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-sn_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_18","18%","","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","","+base_sale_18","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_sale_18","","",""
"","","","","","","","","base","refund","","-base_sale_18","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_sale_18","","",""
"syscebnl_tva_purchase_18","18%","","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","","+base_purchase","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+tax_purchase","","",""
"","","","","","","","","base","refund","","-base_purchase","","",""
"","","","","","","","","tax","refund","syscebnl_445","-tax_purchase","","",""
"syscebnl_tva_sale_10","10%","","","10.0","percent","sale","syscebnl_tax_group_10","base","invoice","","+base_sale_10","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_sale_10","","",""
"","","","","","","","","base","refund","","-base_sale_10","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_sale_10","","",""
"syscebnl_tva_purchase_10","10%","","","10.0","percent","purchase","syscebnl_tax_group_10","base","invoice","","+base_purchase","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+tax_purchase","","",""
"","","","","","","","","base","refund","","-base_purchase","","",""
"","","","","","","","","tax","refund","syscebnl_445","-tax_purchase","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-export","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_non_impos_0","0% NT","0% (non taxable)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+non_taxed","","","0% (non taxable)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-non_taxed","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_sale_18_sd","0% SD","0% Self Delivery","","0.0","percent","sale","syscebnl_tax_group_0","base","invoice","","+sale_self","","0% Livraison à soi-meme","0 LASM"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_self","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-sn.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_10","VAT 10%","T.V.A. 10%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-sn_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_10","VAT 10%","T.V.A. 10%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_sn.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('sn')
    def _get_sn_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('sn', 'res.company')
    def _get_sn_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.sn',
                'account_sale_tax_id': 'tva_sale_18',
                'account_purchase_tax_id': 'tva_purchase_18',
            }
        )
        return company_values

    @template('sn', 'account.account')
    def _get_sn_account_account(self):
        return self._parse_csv('sn', 'account.account', module='l10n_syscohada')

```

## File: models\template_sn_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('sn_syscebnl')
    def _get_sn_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('sn_syscebnl', 'res.company')
    def _get_sn_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.sn',
                'account_sale_tax_id': 'syscebnl_tva_sale_18',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_18',
            }
        )
        return company_values

    @template('sn_syscebnl', 'account.account')
    def _get_sn_syscebnl_account_account(self):
        return self._parse_csv('sn_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_sn
from . import template_sn_syscebnl

```

