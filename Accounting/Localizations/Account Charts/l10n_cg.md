# Odoo Module: l10n_cg

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Congo - Accounting',
    'category': 'Accounting/Localizations/Account Charts',
    'countries': ['cg'],
    'description': """
This module implements the tax for Congo.
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
    <record id="account_tax_report_cg" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.cg"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_cg_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_cg_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_cg_sales" model="account.report.line">
                <field name="name">Outgoing</field>
                <field name="code">CG_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_cg_sales_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CG_TAXABLE_18.base + CG_TAXABLE_9.base + CG_EXPORT.base + CG_SALE_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_cg_sales_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CG_TAXABLE_18.tax + CG_TAXABLE_9.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_sales_18" model="account.report.line">
                        <field name="name">Taxable operations at 18%</field>
                        <field name="code">CG_TAXABLE_18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_18_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_18</field>
                            </record>
                            <record id="account_tax_report_line_sales_18_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_18</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sales_additional_cents" model="account.report.line">
                        <field name="name">Additional cents</field>
                        <field name="code">CG_TAXABLE_9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_cents_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_cents</field>
                            </record>
                            <record id="account_tax_report_line_sales_cents_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_cents</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sales_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">CG_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_export_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">export</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sales_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">CG_SALE_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cg_purchases" model="account.report.line">
                <field name="name">Incoming</field>
                <field name="code">CG_VAT_DEDUCT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_cg_purchases_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CG_PURC_TAXABLE.base + CG_IMPORT.base + CG_PURC_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_cg_purchase_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CG_PURC_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cg_purchases_taxable" model="account.report.line">
                        <field name="name">Deductible</field>
                        <field name="code">CG_PURC_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cg_purchases_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_base</field>
                            </record>
                            <record id="account_tax_report_line_cg_purchases_taxable_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cg_purchases_import" model="account.report.line">
                        <field name="name">Import</field>
                        <field name="code">CG_IMPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cg_purchases_import_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_base</field>
                            </record>
                            <record id="account_tax_report_line_cg_purchases_import_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cg_purchases_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">CG_PURC_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cg_purchases_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cg_net" model="account.report.line">
                <field name="name">Net VAT</field>
                <field name="code">CG_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_cg_net_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CG_VAT_CREDIT.tax + CG_VAT_TO_PAY.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cg_credit" model="account.report.line">
                        <field name="name">VAT Credit</field>
                        <field name="code">CG_VAT_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cg_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CG_VAT_DEDUCT.tax - CG_SALES.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cg_to_pay" model="account.report.line">
                        <field name="name">VAT to pay</field>
                        <field name="code">CG_VAT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cg_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CG_SALES.tax - CG_VAT_DEDUCT.tax</field>
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

## File: data\template\account.fiscal.position-cg.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.cg","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18_9","tva_export_0"
"","","","","","","","tva_purchase_18_9","tva_import_0"

```

## File: data\template\account.fiscal.position-cg_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.cg","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18_9","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_18_9","syscebnl_tva_import_0"

```

## File: data\template\account.tax-cg.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"tva_sale_18_9","18.9%","","True","","","group","sale","tax_group_18_9","tva_sale_18,tva_sale_09","","","","","","",""
"tva_purchase_18_9","18.9%","","True","","","group","purchase","tax_group_18_9","tva_purchase_18,tva_purchase_09","","","","","","",""
"tva_export_0","0% EX","0% (export)","True","","0.0","","sale","tax_group_0","","base","invoice","+export","","","0% (exportation)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-export","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","tax_group_0","","base","invoice","+sale_exempt","","","0% (exonéré)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-sale_exempt","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","True","","0.0","","purchase","tax_group_0","","base","invoice","","","","0% (importation)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","True","","0.0","","purchase","tax_group_0","","base","invoice","+purc_exempt","","","0% (exonéré)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-purc_exempt","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"tva_sale_18","18%","","True","","18.0","percent","sale","tax_group_18","","base","invoice","+base_18","","","",""
"","","","","","","","","","","tax","invoice","+tax_18","pcg_4431","","",""
"","","","","","","","","","","base","refund","-base_18","","","",""
"","","","","","","","","","","tax","refund","-tax_18","pcg_4431","","",""
"tva_purchase_18","18%","","True","","18.0","percent","purchase","tax_group_18","","base","invoice","+purc_base","","","",""
"","","","","","","","","","","tax","invoice","+purc_tax","pcg_4452","","",""
"","","","","","","","","","","base","refund","-purc_base","","","",""
"","","","","","","","","","","tax","refund","-purc_tax","pcg_4452","","",""
"tva_sale_09","0.9%","additional cents","True","","0.9","percent","sale","tax_group_09","","base","invoice","+base_cents","","","centimes additionaux",""
"","","","","","","","","","","tax","invoice","+tax_cents","pcg_4431","","",""
"","","","","","","","","","","base","refund","-base_cents","","","",""
"","","","","","","","","","","tax","refund","-tax_cents","pcg_4431","","",""
"tva_purchase_09","0.9%","","True","","0.9","percent","purchase","tax_group_09","","base","invoice","+purc_base","","","",""
"","","","","","","","","","","tax","invoice","","pcg_4452","","",""
"","","","","","","","","","","base","refund","-purc_base","","","",""
"","","","","","","","","","","tax","refund","","pcg_4452","","",""

```

## File: data\template\account.tax-cg_syscebnl.csv

```csv
"id","name","description","active","invoice_label","amount","amount_type","type_tax_use","tax_group_id","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr","name@fr"
"syscebnl_tva_sale_18_9","18.9%","","True","","","group","sale","syscebnl_tax_group_18_9","syscebnl_tva_sale_18,syscebnl_tva_sale_09","","","","","","",""
"syscebnl_tva_purchase_18_9","18.9%","","True","","","group","purchase","syscebnl_tax_group_18_9","syscebnl_tva_purchase_18,syscebnl_tva_purchase_09","","","","","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","True","","0.0","","sale","syscebnl_tax_group_0","","base","invoice","+export","","","0% (exportation)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-export","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","True","","0.0","","sale","syscebnl_tax_group_0","","base","invoice","+sale_exempt","","","0% (exonéré)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-sale_exempt","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","True","","0.0","","purchase","syscebnl_tax_group_0","","base","invoice","","","","0% (importation)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","True","","0.0","","purchase","syscebnl_tax_group_0","","base","invoice","+purc_exempt","","","0% (exonéré)",""
"","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","base","refund","-purc_exempt","","","",""
"","","","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_sale_18","18%","","False","","18.0","percent","sale","syscebnl_tax_group_18","","base","invoice","+base_18","","","",""
"","","","","","","","","","","tax","invoice","+tax_18","syscebnl_443","","",""
"","","","","","","","","","","base","refund","-base_18","","","",""
"","","","","","","","","","","tax","refund","-tax_18","syscebnl_443","","",""
"syscebnl_tva_purchase_18","18%","","False","","18.0","percent","purchase","syscebnl_tax_group_18","","base","invoice","+purc_base","","","",""
"","","","","","","","","","","tax","invoice","+purc_tax","syscebnl_445","","",""
"","","","","","","","","","","base","refund","-purc_base","","","",""
"","","","","","","","","","","tax","refund","-purc_tax","syscebnl_445","","",""
"syscebnl_tva_sale_09","0.9%","additional cents","False","","0.9","percent","sale","syscebnl_tax_group_09","","base","invoice","+base_cents","","","centimes additionaux",""
"","","","","","","","","","","tax","invoice","+tax_cents","syscebnl_443","","",""
"","","","","","","","","","","base","refund","-base_cents","","","",""
"","","","","","","","","","","tax","refund","-tax_cents","syscebnl_443","","",""
"syscebnl_tva_purchase_09","0.9%","","False","","0.9","percent","purchase","syscebnl_tax_group_09","","base","invoice","+purc_base","","","",""
"","","","","","","","","","","tax","invoice","","syscebnl_445","","",""
"","","","","","","","","","","base","refund","-purc_base","","","",""
"","","","","","","","","","","tax","refund","","syscebnl_445","","",""

```

## File: data\template\account.tax.group-cg.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_18_9","VAT 18.9%","T.V.A. 18.9%","pcg_4431","pcg_4452"
"tax_group_09","VAT 0.9%","T.V.A. 0.9%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-cg_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18_9","VAT 18.9%","T.V.A. 18.9%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_09","VAT 0.9%","T.V.A. 0.9%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_cg.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cg')
    def _get_cg_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('cg', 'res.company')
    def _get_cg_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cg',
                'account_sale_tax_id': 'tva_sale_18_9',
                'account_purchase_tax_id': 'tva_purchase_18_9',
            }
        )
        return company_values

    @template('cg', 'account.account')
    def _get_cg_account_account(self):
        return self._parse_csv('cg', 'account.account', module='l10n_syscohada')

```

## File: models\template_cg_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cg_syscebnl')
    def _get_cg_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('cg_syscebnl', 'res.company')
    def _get_cg_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cg',
                'account_sale_tax_id': 'syscebnl_tva_sale_18_9',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_18_9',
            }
        )
        return company_values

    @template('cg_syscebnl', 'account.account')
    def _get_cg_syscebnl_account_account(self):
        return self._parse_csv('cg_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_cg
from . import template_cg_syscebnl

```

