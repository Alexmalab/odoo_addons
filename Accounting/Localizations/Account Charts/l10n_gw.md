# Odoo Module: l10n_gw

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Guinea-Bissau - Accounting",
    'countries': ['gw'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Guinea-Bissau.
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
    <record id="account_tax_report_gw" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.gw"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_gw_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_gw_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_gw_sales" model="account.report.line">
                <field name="name">Outgoing</field>
                <field name="code">GW_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gw_sales_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GW_TAXABLE_19.base + GW_TAXABLE_10.base + GW_EXPORT.base + GW_SALE_EXEMPT.base + GW_SIMPLIFIED.base</field>
                    </record>
                    <record id="account_tax_report_line_gw_sales_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GW_TAXABLE_19.tax + GW_TAXABLE_10.tax + GW_SIMPLIFIED.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gw_sales_19" model="account.report.line">
                        <field name="name">Taxable operations at 19%</field>
                        <field name="code">GW_TAXABLE_19</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_sales_19_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_19</field>
                            </record>
                            <record id="account_tax_report_line_gw_sales_19_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_19</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gw_sales_10" model="account.report.line">
                        <field name="name">Taxable operations at 10%</field>
                        <field name="code">GW_TAXABLE_10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_sales_10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_10</field>
                            </record>
                            <record id="account_tax_report_line_gw_sales_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_10</field>
                            </record>
                        </field>
                    </record>

                    <record id="account_tax_report_line_gw_simplified_5" model="account.report.line">
                        <field name="name">Operations at 5% under simplified regime</field>
                        <field name="code">GW_SIMPLIFIED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_simplified_5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">GW_5_base</field>
                            </record>
                            <record id="account_tax_report_line_gw_simplified_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">GW_5_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gw_sales_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">GW_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_sales_export_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">export</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gw_sales_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">GW_SALE_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_sales_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_gw_purchases" model="account.report.line">
                <field name="name">Incoming</field>
                <field name="code">GW_VAT_DEDUCT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gw_purchases_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GW_PURC_TAXABLE.base + GW_IMPORT.base + GW_PURC_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_gw_purchase_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GW_PURC_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gw_purchases_taxable" model="account.report.line">
                        <field name="name">Taxable</field>
                        <field name="code">GW_PURC_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_purchases_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_base</field>
                            </record>
                            <record id="account_tax_report_line_gw_purchases_taxable_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gw_purchases_import" model="account.report.line">
                        <field name="name">Import</field>
                        <field name="code">GW_IMPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_purchases_import_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_base</field>
                            </record>
                            <record id="account_tax_report_line_gw_purchases_import_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gw_purchases_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">GW_PURC_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_purchases_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="account_tax_report_line_gw_withholding" model="account.report.line">
                <field name="name">Tax withheld</field>
                <field name="code">GW_WITHHOLDING</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gw_withholding_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">GW_withholding</field>
                    </record>
                </field>
            </record>

            <record id="account_tax_report_line_gw_net" model="account.report.line">
                <field name="name">Net VAT</field>
                <field name="code">GW_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gw_net_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GW_VAT_CREDIT.tax + GW_VAT_TO_PAY.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gw_credit" model="account.report.line">
                        <field name="name">VAT Credit</field>
                        <field name="code">GW_VAT_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GW_VAT_DEDUCT.tax - GW_SALES.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gw_to_pay" model="account.report.line">
                        <field name="name">VAT to pay</field>
                        <field name="code">GW_VAT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gw_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GW_SALES.tax - GW_VAT_DEDUCT.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>
```

## File: data\template\account.fiscal.position-gw.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.td","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_19","tva_export_0"
"","","","","","","","tva_sale_10","tva_export_0"
"","","","","","","","tva_sale_5","tva_export_0"
"","","","","","","","tva_purchase_19","tva_import_0"
"","","","","","","","tva_purchase_10","tva_import_0"

```

## File: data\template\account.fiscal.position-gw_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.td","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_19","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_10","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_5","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_19","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_10","syscebnl_tva_import_0"

```

## File: data\template\account.tax-gw.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr","price_include_override"
"tva_sale_19","19%","","","","19.0","percent","sale","tax_group_19","base","invoice","","+base_19","","","",""
"","","","","","","","","","tax","invoice","pcg_4431","+tax_19","","","",""
"","","","","","","","","","base","refund","","-base_19","","","",""
"","","","","","","","","","tax","refund","pcg_4431","-tax_19","","","",""
"tva_purchase_19","19%","","","","19.0","percent","purchase","tax_group_19","base","invoice","","+purc_base","","","",""
"","","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","","",""
"","","","","","","","","","base","refund","","-purc_base","","","",""
"","","","","","","","","","tax","refund","pcg_4452","-purc_tax","","","",""
"tva_sale_10","10%","","","","10.0","percent","sale","tax_group_10","base","invoice","","+base_10","","","",""
"","","","","","","","","","tax","invoice","pcg_4431","+tax_10","","","",""
"","","","","","","","","","base","refund","","-base_10","","","",""
"","","","","","","","","","tax","refund","pcg_4431","-tax_10","","","",""
"tva_purchase_10","10%","","","","10.0","percent","purchase","tax_group_10","base","invoice","","+purc_base","","","",""
"","","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","","",""
"","","","","","","","","","base","refund","","-purc_base","","","",""
"","","","","","","","","","tax","refund","pcg_4452","-purc_tax","","","",""
"tva_sale_5","5%","","","","5.0","percent","sale","tax_group_5","base","invoice","","+GW_5_base","","","",""
"","","","","","","","","","tax","invoice","pcg_4431","+GW_5_tax","","","",""
"","","","","","","","","","base","refund","","-GW_5_base","","","",""
"","","","","","","","","","tax","refund","pcg_4431","-GW_5_tax","","","",""
"tva_purchase_5","5%","","","","5.0","percent","purchase","tax_group_5","base","invoice","","+purc_base","","","",""
"","","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","","",""
"","","","","","","","","","base","refund","","-purc_base","","","",""
"","","","","","","","","","tax","refund","pcg_4452","-purc_tax","","","",""
"tva_export_0","0% EX","0% (export)","","","0.0","","sale","tax_group_0","base","invoice","","+export","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","-export","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"tva_import_0","0% EX","0% (import)","","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (importation)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"tva_exempt_0","0%","0% (exempt)","","","0.0","","sale","tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","-sale_exempt","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","","0.0","","purchase","tax_group_0","base","invoice","","+purc_exempt","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","-purc_exempt","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"tva_withholding_19","19% WH","","False","","-19.0","percent","sale","tax_group_19","base","invoice","","","","","","tax_excluded"
"","","","","","","","","","tax","invoice","pcg_4452","+GW_withholding","","","",""
"","","","","","","","","","base","refund","","","","","",""
"","","","","","","","","","tax","refund","pcg_4452","-GW_withholding","","","",""
"tva_withholding_10","10% WH","","False","","-10.0","percent","sale","tax_group_10","base","invoice","","","","","","tax_excluded"
"","","","","","","","","","tax","invoice","pcg_4452","+GW_withholding","","","",""
"","","","","","","","","","base","refund","","","","","",""
"","","","","","","","","","tax","refund","pcg_4452","-GW_withholding","","","",""

```

## File: data\template\account.tax-gw_syscebnl.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr","price_include_override"
"syscebnl_tva_sale_19","19%","","","","19.0","percent","sale","syscebnl_tax_group_19","base","invoice","","+base_19","","","",""
"","","","","","","","","","tax","invoice","syscebnl_443","+tax_19","","","",""
"","","","","","","","","","base","refund","","-base_19","","","",""
"","","","","","","","","","tax","refund","syscebnl_443","-tax_19","","","",""
"syscebnl_tva_purchase_19","19%","","","","19.0","percent","purchase","syscebnl_tax_group_19","base","invoice","","+purc_base","","","",""
"","","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","","",""
"","","","","","","","","","base","refund","","-purc_base","","","",""
"","","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","","",""
"syscebnl_tva_sale_10","10%","","","","10.0","percent","sale","syscebnl_tax_group_10","base","invoice","","+base_10","","","",""
"","","","","","","","","","tax","invoice","syscebnl_443","+tax_10","","","",""
"","","","","","","","","","base","refund","","-base_10","","","",""
"","","","","","","","","","tax","refund","syscebnl_443","-tax_10","","","",""
"syscebnl_tva_purchase_10","10%","","","","10.0","percent","purchase","syscebnl_tax_group_10","base","invoice","","+purc_base","","","",""
"","","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","","",""
"","","","","","","","","","base","refund","","-purc_base","","","",""
"","","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","","",""
"syscebnl_tva_sale_5","5%","","","","5.0","percent","sale","syscebnl_tax_group_5","base","invoice","","+GW_5_base","","","",""
"","","","","","","","","","tax","invoice","syscebnl_443","+GW_5_tax","","","",""
"","","","","","","","","","base","refund","","-GW_5_base","","","",""
"","","","","","","","","","tax","refund","syscebnl_443","-GW_5_tax","","","",""
"syscebnl_tva_purchase_5","5%","","","","5.0","percent","purchase","syscebnl_tax_group_5","base","invoice","","+purc_base","","","",""
"","","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","","",""
"","","","","","","","","","base","refund","","-purc_base","","","",""
"","","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+export","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","-export","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (importation)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","-sale_exempt","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","+purc_exempt","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","","",""
"","","","","","","","","","base","refund","","-purc_exempt","","","",""
"","","","","","","","","","tax","refund","","","","","",""
"syscebnl_tva_withholding_19","19% WH","","False","","-19.0","percent","sale","syscebnl_tax_group_19","base","invoice","","","","","","tax_excluded"
"","","","","","","","","","tax","invoice","syscebnl_445","+GW_withholding","","","",""
"","","","","","","","","","base","refund","","","","","",""
"","","","","","","","","","tax","refund","syscebnl_445","-GW_withholding","","","",""
"syscebnl_tva_withholding_10","10% WH","","False","","-10.0","percent","sale","syscebnl_tax_group_10","base","invoice","","","","","","tax_excluded"
"","","","","","","","","","tax","invoice","syscebnl_445","+GW_withholding","","","",""
"","","","","","","","","","base","refund","","","","","",""
"","","","","","","","","","tax","refund","syscebnl_445","-GW_withholding","","","",""

```

## File: data\template\account.tax.group-gw.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_5","VAT 5%","T.V.A. 5%","pcg_4431","pcg_4452"
"tax_group_10","VAT 10%","T.V.A. 10%","pcg_4431","pcg_4452"
"tax_group_19","VAT 19%","T.V.A. 19%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-gw_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_5","VAT 5%","T.V.A. 5%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_10","VAT 10%","T.V.A. 10%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_19","VAT 19%","T.V.A. 19%","syscebnl_443","syscebnl_445"

```

## File: models\template_gw.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gw')
    def _get_gw_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('gw', 'res.company')
    def _get_gw_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.gw',
                'account_sale_tax_id': 'tva_sale_5',
                'account_purchase_tax_id': 'tva_purchase_5',
            }
        )
        return company_values

    @template('gw', 'account.account')
    def _get_gw_account_account(self):
        return self._parse_csv('gw', 'account.account', module='l10n_syscohada')

```

## File: models\template_gw_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gw_syscebnl')
    def _get_gw_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('gw_syscebnl', 'res.company')
    def _get_gw_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.gw',
                'account_sale_tax_id': 'syscebnl_tva_sale_5',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_5',
            }
        )
        return company_values

    @template('gw_syscebnl', 'account.account')
    def _get_gw_syscebnl_account_account(self):
        return self._parse_csv('gw_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_gw
from . import template_gw_syscebnl

```

