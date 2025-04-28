# Odoo Module: l10n_gq

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Guinea Equatorial - Accounting",
    'countries': ['gq'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Guinea Equatorial.
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
    <record id="account_tax_report_gq" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.gq"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_gq_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_gq_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_gq_sales" model="account.report.line">
                <field name="name">Outgoing</field>
                <field name="code">GQ_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gq_sales_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GQ_TAXABLE_15.base + GQ_TAXABLE_30.base + GQ_TAXABLE_6.base + GQ_EXPORT.base + GQ_SALE_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_gq_sales_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GQ_TAXABLE_15.tax + GQ_TAXABLE_30.tax + GQ_TAXABLE_6.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gq_sales_15" model="account.report.line">
                        <field name="name">Taxable operations at 15%</field>
                        <field name="code">GQ_TAXABLE_15</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_sales_15_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_15</field>
                            </record>
                            <record id="account_tax_report_line_gq_sales_15_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_15</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gq_sales_30" model="account.report.line">
                        <field name="name">Taxable operations at 30%</field>
                        <field name="code">GQ_TAXABLE_30</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_sales_30_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_30</field>
                            </record>
                            <record id="account_tax_report_line_gq_sales_30_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_30</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gq_sales_6" model="account.report.line">
                        <field name="name">Taxable operations at 6%</field>
                        <field name="code">GQ_TAXABLE_6</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_sales_6_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_6</field>
                            </record>
                            <record id="account_tax_report_line_gq_sales_6_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_6</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gq_sales_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">GQ_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_sales_export_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">export</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gq_sales_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">GQ_SALE_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_sales_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_gq_purchases" model="account.report.line">
                <field name="name">Incoming</field>
                <field name="code">GQ_VAT_DEDUCT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gq_purchases_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GQ_PURC_TAXABLE.base + GQ_IMPORT.base + GQ_PURC_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_gq_purchase_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GQ_PURC_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gq_purchases_taxable" model="account.report.line">
                        <field name="name">Taxable</field>
                        <field name="code">GQ_PURC_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_purchases_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_base</field>
                            </record>
                            <record id="account_tax_report_line_gq_purchases_taxable_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gq_purchases_import" model="account.report.line">
                        <field name="name">Import</field>
                        <field name="code">GQ_IMPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_purchases_import_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_base</field>
                            </record>
                            <record id="account_tax_report_line_gq_purchases_import_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gq_purchases_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">GQ_PURC_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_purchases_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_gq_net" model="account.report.line">
                <field name="name">Net VAT</field>
                <field name="code">GQ_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gq_net_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GQ_VAT_CREDIT.tax + GQ_VAT_TO_PAY.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gq_credit" model="account.report.line">
                        <field name="name">VAT Credit</field>
                        <field name="code">GQ_VAT_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GQ_VAT_DEDUCT.tax - GQ_SALES.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gq_to_pay" model="account.report.line">
                        <field name="name">VAT to pay</field>
                        <field name="code">GQ_VAT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gq_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GQ_SALES.tax - GQ_VAT_DEDUCT.tax</field>
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

## File: data\template\account.fiscal.position-gq.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.gq","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_15","tva_export_0"
"","","","","","","","tva_sale_6","tva_export_0"
"","","","","","","","tva_sale_30","tva_export_0"
"","","","","","","","tva_purchase_15","tva_import_0"
"","","","","","","","tva_purchase_6","tva_import_0"
"","","","","","","","tva_purchase_30","tva_import_0"

```

## File: data\template\account.fiscal.position-gq_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.gq","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_15","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_6","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_30","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_15","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_6","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_30","syscebnl_tva_import_0"

```

## File: data\template\account.tax-gq.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_15","15%","","","15.0","percent","sale","tax_group_15","base","invoice","","+base_15","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_15","","",""
"","","","","","","","","base","refund","","-base_15","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_15","","",""
"tva_purchase_15","15%","","","15.0","percent","purchase","tax_group_15","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_sale_30","30%","","","30.0","percent","sale","tax_group_30","base","invoice","","+base_30","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_30","","",""
"","","","","","","","","base","refund","","-base_30","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_30","","",""
"tva_purchase_30","30%","","","30.0","percent","purchase","tax_group_30","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_sale_6","6%","","","6.0","percent","sale","tax_group_6","base","invoice","","+base_6","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_6","","",""
"","","","","","","","","base","refund","","-base_6","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_6","","",""
"tva_purchase_6","6%","","","6.0","percent","purchase","tax_group_6","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-export","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_import_0","0% EX","0% (import)","","0.0","","purchase","tax_group_0","base","invoice","","+import_base","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-import_base","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","tax_group_0","base","invoice","","+purc_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-purc_exempt","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-gq_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_15","15%","","","15.0","percent","sale","syscebnl_tax_group_15","base","invoice","","+base_15","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_15","","",""
"","","","","","","","","base","refund","","-base_15","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_15","","",""
"syscebnl_tva_purchase_15","15%","","","15.0","percent","purchase","syscebnl_tax_group_15","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_sale_30","30%","","","30.0","percent","sale","syscebnl_tax_group_30","base","invoice","","+base_30","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_30","","",""
"","","","","","","","","base","refund","","-base_30","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_30","","",""
"syscebnl_tva_purchase_30","30%","","","30.0","percent","purchase","syscebnl_tax_group_30","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_sale_6","6%","","","6.0","percent","sale","syscebnl_tax_group_6","base","invoice","","+base_6","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_6","","",""
"","","","","","","","","base","refund","","-base_6","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_6","","",""
"syscebnl_tva_purchase_6","6%","","","6.0","percent","purchase","syscebnl_tax_group_6","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-export","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","+import_base","","","0% (importation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-import_base","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","","+purc_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-purc_exempt","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-gq.csv

```csv
"id","name","name@es","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","I.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_6","VAT 6%","I.V.A. 6%","pcg_4431","pcg_4452"
"tax_group_15","VAT 15%","I.V.A. 15%","pcg_4431","pcg_4452"
"tax_group_30","VAT 30%","I.V.A. 30%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-gq_syscebnl.csv

```csv
"id","name","name@es","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","I.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_6","VAT 6%","I.V.A. 6%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_15","VAT 15%","I.V.A. 15%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_30","VAT 30%","I.V.A. 30%","syscebnl_443","syscebnl_445"

```

## File: models\template_gq.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gq')
    def _get_gq_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('gq', 'res.company')
    def _get_gq_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.gq',
                'account_sale_tax_id': 'tva_sale_15',
                'account_purchase_tax_id': 'tva_purchase_15',
            }
        )
        return company_values

    @template('gq', 'account.account')
    def _get_gq_account_account(self):
        return self._parse_csv('gq', 'account.account', module='l10n_syscohada')

```

## File: models\template_gq_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gq_syscebnl')
    def _get_gq_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('gq_syscebnl', 'res.company')
    def _get_gq_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.gq',
                'account_sale_tax_id': 'syscebnl_tva_sale_15',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_15',
            }
        )
        return company_values

    @template('gq_syscebnl', 'account.account')
    def _get_gq_syscebnl_account_account(self):
        return self._parse_csv('gq_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_gq
from . import template_gq_syscebnl

```

