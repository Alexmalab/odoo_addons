# Odoo Module: l10n_bj

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Benin - Accounting",
    'countries': ['bj'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Benin.
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
    <record id="account_tax_report_bj" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.bj"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_bj_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_bj_turnover" model="account.report.line">
                <field name="name">II. Turnover (Without Tax)</field>
                <field name="code">BJ_TURNOVER</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_bj_sales_exempt" model="account.report.line">
                        <field name="name">1. Exempted turnover</field>
                        <field name="code">BJ_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_sales_exempt_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_sales_export" model="account.report.line">
                        <field name="name">2. Export of taxable products</field>
                        <field name="code">BJ_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_sales_export_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_2</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_sales_export_non_taxable" model="account.report.line">
                        <field name="name">3. Export of non-taxable products</field>
                        <field name="code">BJ_EXPORT_NON_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_sales_export_non_taxable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_3</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_sales_taxable" model="account.report.line">
                        <field name="name">4. Taxable operations</field>
                        <field name="code">BJ_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_sales_taxable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_4</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_sales_self" model="account.report.line">
                        <field name="name">5. Self Deliveries</field>
                        <field name="code">BJ_SELF</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_sales_self_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_5</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_sales_total_turnover" model="account.report.line">
                        <field name="name">6. Total turnover without VAT</field>
                        <field name="code">BJ_TOTAL_TURNOVER</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_sales_total_turnover_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BJ_EXEMPT.balance + BJ_EXPORT.balance + BJ_EXPORT_NON_TAXABLE.balance + BJ_TAXABLE.balance + BJ_SELF.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_bj_deductions" model="account.report.line">
                <field name="name">III. Deductions</field>
                <field name="code">BJ_DEDUCTIONS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_bj_deductible_reported" model="account.report.line">
                        <field name="name">7. Credit reported from last month</field>
                        <field name="code">BJ_REPORTED</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_deductible_reported_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BJ_REPORTED._applied_carryover_balance</field>
                            </record>
                            <record id="account_tax_report_line_bj_deductible_reported_balance_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_deductible_goods_services" model="account.report.line">
                        <field name="name">8. Deduction on goods and services (without assets)</field>
                        <field name="code">BJ_GOODS_SERVICES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_deductible_goods_services_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_8</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_deductible_assets" model="account.report.line">
                        <field name="name">9. Deduction on assets</field>
                        <field name="code">BJ_ASSETS</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_deductible_assets_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_9</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_deductible_additional" model="account.report.line">
                        <field name="name">10. Additional deductions</field>
                        <field name="code">BJ_DEDU_ADDITIONAL</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_deductible_additional_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_deductible_repayment" model="account.report.line">
                        <field name="name">11. Repayment to do</field>
                        <field name="code">BJ_DEDU_REPAYMENT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_deductible_repayment_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_deductible_total" model="account.report.line">
                        <field name="name">12. Total</field>
                        <field name="code">BJ_DEDU_TOTAL</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_deductible_total_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BJ_REPORTED.balance + BJ_GOODS_SERVICES.balance + BJ_ASSETS.balance + BJ_DEDU_ADDITIONAL.balance + BJ_DEDU_REPAYMENT.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_bj_net" model="account.report.line">
                <field name="name">IV. NET VAT</field>
                <field name="code">BJ_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_bj_net_gross" model="account.report.line">
                        <field name="name">13. Gross VAT (18% x l.4 + l.5)</field>
                        <field name="code">BJ_GROSS</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_net_gross_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BJ_13</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_net_deductible" model="account.report.line">
                        <field name="name">14. VAT deductible</field>
                        <field name="code">BJ_VAT_DEDUCTIBLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_net_deductible_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BJ_DEDU_TOTAL.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_net_to_pay" model="account.report.line">
                        <field name="name">15. Net VAT to pay</field>
                        <field name="code">BJ_NET_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_net_to_pay_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BJ_GROSS.balance - BJ_VAT_DEDUCTIBLE.balance</field>
                                <field name="subformula">if_above(XOF(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_bj_credit_to_report" model="account.report.line">
                        <field name="name">16. Credit to report</field>
                        <field name="code">BJ_TO_REPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_bj_credit_to_report_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BJ_VAT_DEDUCTIBLE.balance - BJ_GROSS.balance</field>
                                <field name="subformula">if_above(XOF(0))</field>
                                <field name="carryover_target" eval="False"/>
                            </record>
                            <record id="account_tax_report_line_bj_credit_to_report_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">BJ_TO_REPORT.balance</field>
                                <field name="carryover_target">BJ_REPORTED._applied_carryover_balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-bj.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.bj","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18","tva_export_0"
"","","","","","","","tva_sale_service_18","tva_export_0"
"","","","","","","","tva_exempt_0","tva_export_exempt_0"
"","","","","","","","tva_purchase_18","tva_import_0"
"","","","","","","","tva_purchase_asset_18","tva_import_0"

```

## File: data\template\account.fiscal.position-bj_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.bj","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_service_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_exempt_0","syscebnl_tva_export_exempt_0"
"","","","","","","","syscebnl_tva_purchase_18","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_asset_18","syscebnl_tva_import_0"

```

## File: data\template\account.tax-bj.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_18","18% G","18% goods","","18.0","percent","sale","tax_group_18","base","invoice","","+BJ_4","","","18% biens"
"","","","","","","","","tax","invoice","pcg_4431","+BJ_13","","",""
"","","","","","","","","base","refund","","-BJ_4","","",""
"","","","","","","","","tax","refund","pcg_4431","-BJ_13","","",""
"tva_sale_service_18","18% S","18% services","","18.0","percent","sale","tax_group_18","base","invoice","","+BJ_4","","",""
"","","","","","","","","tax","invoice","pcg_4431","+BJ_13","","",""
"","","","","","","","","base","refund","","-BJ_4","","",""
"","","","","","","","","tax","refund","pcg_4431","-BJ_13","","",""
"tva_sale_self_18","18% SD","18% Self-delivery","","18.0","percent","sale","tax_group_18","base","invoice","","+BJ_5","","18% LASM","18% Livraison à soi-même"
"","","","","","","","","tax","invoice","pcg_4431","-BJ_13","-100","",""
"","","","","","","","","tax","invoice","pcg_4452","+BJ_8","","",""
"","","","","","","","","base","refund","","-BJ_5","","",""
"","","","","","","","","tax","refund","pcg_4431","+BJ_13","-100","",""
"","","","","","","","","tax","refund","pcg_4452","-BJ_8","","",""
"tva_purchase_18","18%","","","18.0","percent","purchase","tax_group_18","base","invoice","","","","",""
"","","","","","","","","tax","invoice","pcg_4452","+BJ_8","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4452","-BJ_8","","",""
"tva_purchase_asset_18","18% Asset","18% (asset)","","18.0","percent","purchase","tax_group_18","base","invoice","","","","18% Immo","18% (immobilisation)"
"","","","","","","","","tax","invoice","pcg_4451","+BJ_9","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","pcg_4451","-BJ_9","","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","","+BJ_2","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BJ_2","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_export_exempt_0","0% EX NI","0% (export of non-taxable products)","False","0.0","","sale","tax_group_0","base","invoice","","+BJ_3","","","0% (exportation de produits non taxables)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BJ_3","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","","+BJ_1","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BJ_1","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-bj_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_18","18% G","18% goods","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","","+BJ_4","","","18% biens"
"","","","","","","","","tax","invoice","syscebnl_443","+BJ_13","","",""
"","","","","","","","","base","refund","","-BJ_4","","",""
"","","","","","","","","tax","refund","syscebnl_443","-BJ_13","","",""
"syscebnl_tva_sale_service_18","18% S","18% services","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","","+BJ_4","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+BJ_13","","",""
"","","","","","","","","base","refund","","-BJ_4","","",""
"","","","","","","","","tax","refund","syscebnl_443","-BJ_13","","",""
"syscebnl_tva_sale_self_18","18% SD","18% Self-delivery","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","","+BJ_5","","18% LASM","18% Livraison à soi-même"
"","","","","","","","","tax","invoice","syscebnl_443","-BJ_13","-100","",""
"","","","","","","","","tax","invoice","syscebnl_445","+BJ_8","","",""
"","","","","","","","","base","refund","","-BJ_5","","",""
"","","","","","","","","tax","refund","syscebnl_443","+BJ_13","-100","",""
"","","","","","","","","tax","refund","syscebnl_445","-BJ_8","","",""
"syscebnl_tva_purchase_18","18%","","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","","","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+BJ_8","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_445","-BJ_8","","",""
"syscebnl_tva_purchase_asset_18","18% Asset","18% (asset)","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","","","","18% Immo","18% (immobilisation)"
"","","","","","","","","tax","invoice","syscebnl_445","+BJ_9","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","syscebnl_445","-BJ_9","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+BJ_2","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BJ_2","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_export_exempt_0","0% EX NI","0% (export of non-taxable products)","False","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+BJ_3","","","0% (exportation de produits non taxables)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BJ_3","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+BJ_1","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-BJ_1","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-bj.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-bj_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_bj.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bj')
    def _get_bj_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('bj', 'res.company')
    def _get_bj_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.bj',
                'account_sale_tax_id': 'tva_sale_18',
                'account_purchase_tax_id': 'tva_purchase_18',
            }
        )
        return company_values

    @template('bj', 'account.account')
    def _get_bj_account_account(self):
        return self._parse_csv('bj', 'account.account', module='l10n_syscohada')

```

## File: models\template_bj_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('bj_syscebnl')
    def _get_bj_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('bj_syscebnl', 'res.company')
    def _get_bj_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.bj',
                'account_sale_tax_id': 'syscebnl_tva_sale_18',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_18',
            }
        )
        return company_values

    @template('bj_syscebnl', 'account.account')
    def _get_bj_syscebnl_account_account(self):
        return self._parse_csv('bj_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_bj
from . import template_bj_syscebnl

```

