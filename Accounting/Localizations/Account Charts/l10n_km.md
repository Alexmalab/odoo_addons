# Odoo Module: l10n_km

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Comoros - Accounting",
    'countries': ['km'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Comoros.
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
    <record id="account_tax_report_km" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.km"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_km_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_km_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_km_operations" model="account.report.line">
                <field name="name">Operations</field>
                <field name="code">KM_OPERATIONS</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_km_sales_base" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">KM_25.base + KM_10.base + KM_7_5.base + KM_5.base + KM_3.base + KM_EXEMPT.base + KM_EXPORT.base</field>
                    </record>
                    <record id="account_tax_report_line_km_sales_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">KM_25.tax + KM_10.tax + KM_7_5.tax + KM_5.tax + KM_3.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_km_operations_25" model="account.report.line">
                        <field name="name">Taxable at 25%</field>
                        <field name="code">KM_25</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_km_operations_25_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_25</field>
                            </record>
                            <record id="account_tax_report_line_km_operations_25_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_25</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_km_operations_10" model="account.report.line">
                        <field name="name">Taxable at 10%</field>
                        <field name="code">KM_10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_km_operations_10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_10</field>
                            </record>
                            <record id="account_tax_report_line_km_operations_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_km_operations_7_5" model="account.report.line">
                        <field name="name">Taxable at 7.5%</field>
                        <field name="code">KM_7_5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_km_operations_7_5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_7_5</field>
                            </record>
                            <record id="account_tax_report_line_km_operations_7_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_7_5</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_km_operations_5" model="account.report.line">
                        <field name="name">Taxable at 5%</field>
                        <field name="code">KM_5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_km_operations_5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_5</field>
                            </record>
                            <record id="account_tax_report_line_km_operations_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_5</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_km_operations_3" model="account.report.line">
                        <field name="name">Taxable at 3%</field>
                        <field name="code">KM_3</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_km_operations_3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_3</field>
                            </record>
                            <record id="account_tax_report_line_km_operations_3_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_3</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_km_operations_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">KM_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_km_operations_exempt_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">km_exempt</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_km_operations_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">KM_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_km_operations_export_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">km_export</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_km_prepayment" model="account.report.line">
                <field name="name">Prepayments made</field>
                <field name="code">KM_PREPAYMENTS</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_km_prepayment_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">km_prepayments</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_km_credit_reported" model="account.report.line">
                <field name="name">Credit balance from previous return</field>
                <field name="code">KM_CREDIT_REPORTED</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_km_credit_reported_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">KM_CREDIT_REPORTED._applied_carryover_tax</field>
                    </record>
                    <record id="account_tax_report_line_km_credit_reported_tax_carryover" model="account.report.expression">
                        <field name="label">_applied_carryover_tax</field>
                        <field name="engine">external</field>
                        <field name="formula">most_recent</field>
                        <field name="date_scope">previous_tax_period</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_km_due" model="account.report.line">
                <field name="name">Due tax for the month</field>
                <field name="code">KM_DUE</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_km_due_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">KM_OPERATIONS.tax - KM_PREPAYMENTS.tax - KM_CREDIT_REPORTED.tax</field>
                        <field name="subformula">if_above(KMF(0))</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_km_credit_to_report" model="account.report.line">
                <field name="name">Credit balance to report for the next return</field>
                <field name="code">KM_TO_REPORT</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_km_credit_to_report_tax" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">KM_PREPAYMENTS.tax + KM_CREDIT_REPORTED.tax - KM_OPERATIONS.tax</field>
                        <field name="subformula">if_above(KMF(0))</field>
                        <field name="carryover_target" eval="False"/>
                    </record>
                    <record id="account_tax_report_line_km_credit_to_report_carryover" model="account.report.expression">
                        <field name="label">_carryover_tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">KM_TO_REPORT.tax</field>
                        <field name="carryover_target">KM_CREDIT_REPORTED._applied_carryover_tax</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.fiscal.position-km.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.km","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_25","tva_export_0"
"","","","","","","","tva_sale_10","tva_export_0"
"","","","","","","","tva_sale_7_5","tva_export_0"
"","","","","","","","tva_sale_5","tva_export_0"
"","","","","","","","tva_sale_3","tva_export_0"

```

## File: data\template\account.fiscal.position-km_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.km","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_25","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_10","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_7_5","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_5","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_3","syscebnl_tva_export_0"

```

## File: data\template\account.tax-km.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_25","25%","","","25.0","percent","sale","tax_group_25","base","invoice","","+base_25","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_25","","",""
"","","","","","","","","base","refund","","-base_25","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_25","","",""
"tva_sale_10","10%","","","10.0","percent","sale","tax_group_10","base","invoice","","+base_10","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_10","","",""
"","","","","","","","","base","refund","","-base_10","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_10","","",""
"tva_sale_7_5","7.5%","","","7.5","percent","sale","tax_group_7_5","base","invoice","","+base_7_5","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_7_5","","",""
"","","","","","","","","base","refund","","-base_7_5","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_7_5","","",""
"tva_sale_5","5%","","","5.0","percent","sale","tax_group_5","base","invoice","","+base_5","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_5","","",""
"","","","","","","","","base","refund","","-base_5","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_5","","",""
"tva_sale_3","3%","","","3.0","percent","sale","tax_group_3","base","invoice","","+base_3","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_3","","",""
"","","","","","","","","base","refund","","-base_3","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_3","","",""
"tva_purchase","0%","","","0.0","percent","purchase","tax_group_0","base","invoice","","","","",""
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","","+km_export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-km_export","","",""
"","","","","","","","","tax","refund","","","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","","+km_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-km_exempt","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax-km_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_25","25%","","","25.0","percent","sale","syscebnl_tax_group_25","base","invoice","","+base_25","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_25","","",""
"","","","","","","","","base","refund","","-base_25","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_25","","",""
"syscebnl_tva_sale_10","10%","","","10.0","percent","sale","syscebnl_tax_group_10","base","invoice","","+base_10","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_10","","",""
"","","","","","","","","base","refund","","-base_10","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_10","","",""
"syscebnl_tva_sale_7_5","7.5%","","","7.5","percent","sale","syscebnl_tax_group_7_5","base","invoice","","+base_7_5","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_7_5","","",""
"","","","","","","","","base","refund","","-base_7_5","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_7_5","","",""
"syscebnl_tva_sale_5","5%","","","5.0","percent","sale","syscebnl_tax_group_5","base","invoice","","+base_5","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_5","","",""
"","","","","","","","","base","refund","","-base_5","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_5","","",""
"syscebnl_tva_sale_3","3%","","","3.0","percent","sale","syscebnl_tax_group_3","base","invoice","","+base_3","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_3","","",""
"","","","","","","","","base","refund","","-base_3","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_3","","",""
"syscebnl_tva_purchase","0%","","","0.0","percent","purchase","syscebnl_tax_group_0","base","invoice","","","","",""
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+km_export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-km_export","","",""
"","","","","","","","","tax","refund","","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","","+km_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","base","refund","","-km_exempt","","",""
"","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-km.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_3","VAT 3%","T.V.A. 3%","pcg_4431","pcg_4452"
"tax_group_5","VAT 5%","T.V.A. 5%","pcg_4431","pcg_4452"
"tax_group_7_5","VAT 7.5%","T.V.A. 7.5%","pcg_4431","pcg_4452"
"tax_group_10","VAT 10%","T.V.A. 10%","pcg_4431","pcg_4452"
"tax_group_25","VAT 25%","T.V.A. 25%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-km_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_3","VAT 3%","T.V.A. 3%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_5","VAT 5%","T.V.A. 5%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_7_5","VAT 7.5%","T.V.A. 7.5%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_10","VAT 10%","T.V.A. 10%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_25","VAT 25%","T.V.A. 25%","syscebnl_443","syscebnl_445"

```

## File: models\template_km.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('km')
    def _get_km_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('km', 'res.company')
    def _get_km_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.km',
                'account_sale_tax_id': 'tva_sale_10',
                'account_purchase_tax_id': 'tva_purchase',
            }
        )
        return company_values

    @template('km', 'account.account')
    def _get_km_account_account(self):
        return self._parse_csv('km', 'account.account', module='l10n_syscohada')

```

## File: models\template_km_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('km_syscebnl')
    def _get_km_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('km_syscebnl', 'res.company')
    def _get_km_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.km',
                'account_sale_tax_id': 'syscebnl_tva_sale_10',
                'account_purchase_tax_id': 'syscebnl_tva_purchase',
            }
        )
        return company_values

    @template('km_syscebnl', 'account.account')
    def _get_km_syscebnl_account_account(self):
        return self._parse_csv('km_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_km
from . import template_km_syscebnl

```

