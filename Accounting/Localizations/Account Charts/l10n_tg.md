# Odoo Module: l10n_tg

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Togo - Accounting',
    'countries': ['tg'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Togo.
===========================================================

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
    <record id="account_tax_report_tg" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.tg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_tg_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_tg_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_tg_sales" model="account.report.line">
                <field name="name">1/11. Outgoing</field>
                <field name="code">TG_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_tg_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TG_NON_TAXABLE.base + TG_TAXABLE.base</field>
                    </record>
                    <record id="account_tax_report_line_tg_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TG_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_tg_sales_non_taxable" model="account.report.line">
                        <field name="name">2. Non Taxable operations</field>
                        <field name="code">TG_NON_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_sales_non_taxable_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TG_SALE_EXEMPT.base + TG_SALE_NOT_IMPOSABLE.base + TG_EXPORT.base</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_tg_sales_non_taxable_exempt" model="account.report.line">
                                <field name="name">3. Exempt</field>
                                <field name="code">TG_SALE_EXEMPT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_tg_sales_non_taxable_exempt_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_tg_sales_non_taxable_not_imposable" model="account.report.line">
                                <field name="name">4. Not imposable</field>
                                <field name="code">TG_SALE_NOT_IMPOSABLE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_tg_sales_non_taxable_not_imposable_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_tg_sales_non_taxable_sales_export" model="account.report.line">
                                <field name="name">5. Export Non imposable</field>
                                <field name="code">TG_EXPORT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_tg_sales_non_taxable_export_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tg_sales_taxable" model="account.report.line">
                        <field name="name">6/12. Taxable operations</field>
                        <field name="code">TG_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_sales_taxable_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TG_TAXABLE_18.base + TG_TAXABLE_PM.base + TG_TAXABLE_SD.base + TG_TAXABLE_EXPORT.base</field>
                            </record>
                            <record id="account_tax_report_line_tg_sales_taxable_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TG_TAXABLE_18.tax + TG_TAXABLE_PM.tax + TG_TAXABLE_SD.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_tg_sales_taxable_18" model="account.report.line">
                                <field name="name">7/13. At 18%</field>
                                <field name="code">TG_TAXABLE_18</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_tg_sales_taxable_18_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_7</field>
                                    </record>
                                    <record id="account_tax_report_line_tg_sales_taxable_18_tax_tag" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_13</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_tg_sales_taxable_public_market" model="account.report.line">
                                <field name="name">8/14. Public Markets</field>
                                <field name="code">TG_TAXABLE_PM</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_tg_sales_taxable_public_market_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_8</field>
                                    </record>
                                    <record id="account_tax_report_line_tg_sales_taxable_public_market_tax_tag" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_14</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_tg_sales_taxable_self_delivery" model="account.report.line">
                                <field name="name">9/15. Self Delivery</field>
                                <field name="code">TG_TAXABLE_SD</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_tg_sales_taxable_self_delivery_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_9</field>
                                    </record>
                                    <record id="account_tax_report_line_tg_sales_taxable_self_delivery_tax_tag" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_15</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_tg_sales_taxable_export" model="account.report.line">
                                <field name="name">10. Exports</field>
                                <field name="code">TG_TAXABLE_EXPORT</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_tg_sales_taxable_taxable_export_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TG_10</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_tg_deductible" model="account.report.line">
                <field name="name">16. Deductible VAT</field>
                <field name="code">TG_VAT_DEDUCT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_tg_deductible_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TG_GOODS_SERVICE.base + TG_Assets.base</field>
                    </record>
                    <record id="account_tax_report_line_tg_deductible_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TG_VAT_REPORTED.tax + TG_GOODS_SERVICE.tax + TG_Assets.tax - TG_ADD_RED.tax + TG_REIMBURSEMENT.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_tg_deductible_reported" model="account.report.line">
                        <field name="name">17. VAT Reported</field>
                        <field name="code">TG_VAT_REPORTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_deductible_reported_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TG_VAT_REPORTED._applied_carryover_tax</field>
                            </record>
                            <record id="account_tax_report_line_tg_deductible_reported_tax_tag_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_tax</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tg_deductible_goods_services" model="account.report.line">
                        <field name="name">18. On goods and services except assets</field>
                        <field name="code">TG_GOODS_SERVICE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_goods_services_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TG_18_Base</field>
                            </record>
                            <record id="account_tax_report_line_tg_goods_services_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TG_18</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tg_deductible_assets" model="account.report.line">
                        <field name="name">19. On assets</field>
                        <field name="code">TG_Assets</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_assets_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TG_19_base</field>
                            </record>
                            <record id="account_tax_report_line_tg_assets_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TG_19</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tg_deductible_complement" model="account.report.line">
                        <field name="name">20. Additional reduction asked</field>
                        <field name="code">TG_ADD_RED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_complement_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tg_deductible_reimbursement" model="account.report.line">
                        <field name="name">21. Reimbursement</field>
                        <field name="code">TG_REIMBURSEMENT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_reimbursement_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_tg_net_vat" model="account.report.line">
                <field name="name">23. Net VAT</field>
                <field name="code">TG_NET_VAT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_tg_net_vat_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TG_SALES.tax - TG_VAT_DEDUCT.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_tg_to_pay" model="account.report.line">
                        <field name="name">26. Net vat to pay</field>
                        <field name="code">TG_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_to_pay_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TG_SALES.tax - TG_VAT_DEDUCT.tax</field>
                                <field name="subformula">if_above(XOF(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tg_to_report" model="account.report.line">
                        <field name="name">27. Credit to report</field>
                        <field name="code">TG_REPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_tg_to_report_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TG_VAT_DEDUCT.tax - TG_SALES.tax</field>
                                <field name="subformula">if_above(XOF(0))</field>
                                <field name="carryover_target" eval="False"/>
                            </record>
                            <record id="account_tax_report_line_tg_to_report_tax_carryover" model="account.report.expression">
                                <field name="label">_carryover_tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TG_REPORT.tax</field>
                                <field name="carryover_target">TG_VAT_REPORTED._applied_carryover_tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-tg.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.tg","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18","tva_export_0"
"","","","","","","","tva_purchase_good_18","tva_import_0"
"","","","","","","","tva_purchase_assets_18","tva_import_0"
"","","","","","","","tva_exempt_0","tva_export_exempt_0"

```

## File: data\template\account.fiscal.position-tg_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.tg","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_good_18","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_assets_18","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_exempt_0","syscebnl_tva_export_exempt_0"

```

## File: data\template\account.tax-tg.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"tva_sale_18","18%","","True","","18.0","percent","sale","tax_group_18","base","invoice","+TG_7","","","",""
"","","","","","","","","","tax","invoice","+TG_13","pcg_4431","","",""
"","","","","","","","","","base","refund","-TG_7","","","",""
"","","","","","","","","","tax","refund","-TG_13","pcg_4431","","",""
"tva_sale_18_pm","18% PM","18% Public Markets","False","","18.0","percent","sale","tax_group_18","base","invoice","+TG_8","","","18% Marchés publics",""
"","","","","","","","","","tax","invoice","+TG_14","pcg_4431","","",""
"","","","","","","","","","base","refund","-TG_8","","","",""
"","","","","","","","","","tax","refund","-TG_14","pcg_4431","","",""
"tva_sale_18_sd","18% SD","18% Self Delivery","False","","18.0","percent","sale","tax_group_18","base","invoice","+TG_9","","","18% Livraison à soi-meme","18 LASM"
"","","","","","","","","","tax","invoice","+TG_15","pcg_4431","","",""
"","","","","","","","","","base","refund","-TG_9","","","",""
"","","","","","","","","","tax","refund","-TG_15","pcg_4431","","",""
"tva_purchase_good_18","18%","18% goods and services except assets","True","","18.0","percent","purchase","tax_group_18","base","invoice","+TG_18_Base","","","18% Produits et services sauf assets",""
"","","","","","","","","","tax","invoice","+TG_18","pcg_4452","","",""
"","","","","","","","","","base","refund","-TG_18_Base","","","",""
"","","","","","","","","","tax","refund","-TG_18","pcg_4452","","",""
"tva_purchase_assets_18","18% Asset","18% assets","True","","18.0","percent","purchase","tax_group_18","base","invoice","+TG_19_base","","","18% immobilisations","18% Immo"
"","","","","","","","","","tax","invoice","+TG_19","pcg_4451","","",""
"","","","","","","","","","base","refund","-TG_19_base","","","",""
"","","","","","","","","","tax","refund","-TG_19","pcg_4451","","",""
"tva_export_0","0% EX","0% (export)","True","","0.0","","sale","tax_group_0","base","invoice","+TG_10","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_10","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_export_exempt_0","0% EX Exempt","0% (export of exempted product)","True","","0.0","","sale","tax_group_0","base","invoice","+TG_5","","","0% (exportation de produits exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_5","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","True","","0.0","","purchase","tax_group_0","base","invoice","","","","0% (importation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_non_imposable_0","0% NT","0% (non taxable)","False","","0.0","","sale","tax_group_0","base","invoice","+TG_4","","","0% (non taxable)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_4","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","tax_group_0","base","invoice","+TG_3","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_3","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","True","","0.0","","purchase","tax_group_0","base","invoice","","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-tg_syscebnl.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"syscebnl_tva_sale_18","18%","","True","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","+TG_7","","","",""
"","","","","","","","","","tax","invoice","+TG_13","syscebnl_443","","",""
"","","","","","","","","","base","refund","-TG_7","","","",""
"","","","","","","","","","tax","refund","-TG_13","syscebnl_443","","",""
"syscebnl_tva_sale_18_pm","18% PM","18% Public Markets","False","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","+TG_8","","","18% Marchés publics",""
"","","","","","","","","","tax","invoice","+TG_14","syscebnl_443","","",""
"","","","","","","","","","base","refund","-TG_8","","","",""
"","","","","","","","","","tax","refund","-TG_14","syscebnl_443","","",""
"syscebnl_tva_sale_18_sd","18% SD","18% Self Delivery","False","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","+TG_9","","","18% Livraison à soi-meme","18 LASM"
"","","","","","","","","","tax","invoice","+TG_15","syscebnl_443","","",""
"","","","","","","","","","base","refund","-TG_9","","","",""
"","","","","","","","","","tax","refund","-TG_15","syscebnl_443","","",""
"syscebnl_tva_purchase_good_18","18%","18% goods and services except assets","True","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","+TG_18_Base","","","18% Produits et services sauf assets",""
"","","","","","","","","","tax","invoice","+TG_18","syscebnl_445","","",""
"","","","","","","","","","base","refund","-TG_18_Base","","","",""
"","","","","","","","","","tax","refund","-TG_18","syscebnl_445","","",""
"syscebnl_tva_purchase_assets_18","18% Asset","18% assets","True","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","+TG_19_base","","","18% immobilisations","18% Immo"
"","","","","","","","","","tax","invoice","+TG_19","syscebnl_445","","",""
"","","","","","","","","","base","refund","-TG_19_base","","","",""
"","","","","","","","","","tax","refund","-TG_19","syscebnl_445","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","True","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+TG_10","","","0% (exportation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_10","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_export_exempt_0","0% EX Exempt","0% (export of exempted product)","True","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+TG_5","","","0% (exportation de produits exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_5","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","True","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","0% (importation)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_non_imposable_0","0% NT","0% (non taxable)","False","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+TG_4","","","0% (non taxable)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_4","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+TG_3","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","-TG_3","","","",""
"","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","True","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","0% (exonéré)",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-tg.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-tg_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_tg.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('tg')
    def _get_tg_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('tg', 'res.company')
    def _get_tg_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.tg',
                'account_sale_tax_id': 'tva_sale_18',
                'account_purchase_tax_id': 'tva_purchase_good_18',
            }
        )
        return company_values

    @template('tg', 'account.account')
    def _get_tg_account_account(self):
        return self._parse_csv('tg', 'account.account', module='l10n_syscohada')

```

## File: models\template_tg_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('tg_syscebnl')
    def _get_tg_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('tg_syscebnl', 'res.company')
    def _get_tg_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.tg',
                'account_sale_tax_id': 'syscebnl_tva_sale_18',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_good_18',
            }
        )
        return company_values

    @template('tg_syscebnl', 'account.account')
    def _get_tg_syscebnl_account_account(self):
        return self._parse_csv('tg_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_tg
from . import template_tg_syscebnl

```

