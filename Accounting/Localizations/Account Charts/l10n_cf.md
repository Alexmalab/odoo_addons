# Odoo Module: l10n_cf

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Central African Republic - Accounting",
    'countries': ['cf'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Central African Republic.
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
    <record id="account_tax_report_cf" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.cf"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_cf_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_cf_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_cf_sales" model="account.report.line">
                <field name="name">Outgoing</field>
                <field name="code">CF_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_cf_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CF_TAXABLE.base + CF_EXPORT.base + CF_SALE_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_cf_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CF_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cf_sales_taxable" model="account.report.line">
                        <field name="name">Taxable operations</field>
                        <field name="code">CF_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cf_sales_taxable_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CF_TAXABLE_19.base + CF_TAXABLE_5.base</field>
                            </record>
                            <record id="account_tax_report_line_cf_sales_taxable_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CF_TAXABLE_19.tax + CF_TAXABLE_5.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_cf_sales_taxable_19" model="account.report.line">
                                <field name="name">Taxable - normal rate</field>
                                <field name="code">CF_TAXABLE_19</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cf_sales_taxable_19_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_sale_19</field>
                                    </record>
                                    <record id="account_tax_report_line_cf_sales_taxable_19_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_sale_19</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_cf_sales_taxable_5" model="account.report.line">
                                <field name="name">Taxable - reduced rate</field>
                                <field name="code">CF_TAXABLE_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_cf_sales_taxable_5_base" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_sale_5</field>
                                    </record>
                                    <record id="account_tax_report_line_cf_sales_taxable_5_tax" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_sale_5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sales_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">CF_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_export_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">export</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sales_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">CF_SALE_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_exempt_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cf_purchases" model="account.report.line">
                <field name="name">Incoming</field>
                <field name="code">CF_VAT_DEDUCT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_cf_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CF_PURC_TAXABLE.base + CF_IMPORT.base + CF_PURC_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_cf_purchase_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CF_PURC_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cf_purchases_taxable" model="account.report.line">
                        <field name="name">Taxable</field>
                        <field name="code">CF_PURC_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cf_purchases_taxable_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_base</field>
                            </record>
                            <record id="account_tax_report_line_cf_purchases_taxable_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cf_purchases_import" model="account.report.line">
                        <field name="name">Import</field>
                        <field name="code">CF_IMPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cf_purchases_import_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_base</field>
                            </record>
                            <record id="account_tax_report_line_cf_purchases_import_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cf_purchases_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">CF_PURC_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cf_purchases_exempt_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_cf_net" model="account.report.line">
                <field name="name">Net VAT</field>
                <field name="code">CF_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_cf_net_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CF_VAT_CREDIT.tax + CF_VAT_TO_PAY.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_cf_credit" model="account.report.line">
                        <field name="name">VAT Credit</field>
                        <field name="code">CF_VAT_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cf_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CF_VAT_DEDUCT.tax - CF_SALES.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_cf_to_pay" model="account.report.line">
                        <field name="name">VAT to pay</field>
                        <field name="code">CF_VAT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_cf_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CF_SALES.tax - CF_VAT_DEDUCT.tax</field>
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

## File: data\template\account.fiscal.position-cf.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.cf","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_19","tva_export_0"
"","","","","","","","tva_sale_5","tva_export_0"
"","","","","","","","tva_purchase_19","tva_import_0"
"","","","","","","","tva_purchase_5","tva_import_0"

```

## File: data\template\account.fiscal.position-cf_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.cf","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_19","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_5","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_19","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_5","syscebnl_tva_import_0"

```

## File: data\template\account.tax-cf.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"tva_sale_19","19%","","","19.0","percent","sale","tax_group_19","base","invoice","","+base_sale_19","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_sale_19","","",""
"","","","","","","","","base","refund","","-base_sale_19","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_sale_19","","",""
"tva_purchase_19","19%","","","19.0","percent","purchase","tax_group_19","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","pcg_4452","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","pcg_4452","-purc_tax","","",""
"tva_sale_5","5%","","","5.0","percent","sale","tax_group_5","base","invoice","","+base_sale_5","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_sale_5","","",""
"","","","","","","","","base","refund","","-base_sale_5","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_sale_5","","",""
"tva_purchase_5","5%","","","5.0","percent","purchase","tax_group_5","base","invoice","","+purc_base","","",""
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

## File: data\template\account.tax-cf_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","name@fr","description@fr"
"syscebnl_tva_sale_19","19%","","","19.0","percent","sale","syscebnl_tax_group_19","base","invoice","","+base_sale_19","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_sale_19","","",""
"","","","","","","","","base","refund","","-base_sale_19","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_sale_19","","",""
"syscebnl_tva_purchase_19","19%","","","19.0","percent","purchase","syscebnl_tax_group_19","base","invoice","","+purc_base","","",""
"","","","","","","","","tax","invoice","syscebnl_445","+purc_tax","","",""
"","","","","","","","","base","refund","","-purc_base","","",""
"","","","","","","","","tax","refund","syscebnl_445","-purc_tax","","",""
"syscebnl_tva_sale_5","5%","","","5.0","percent","sale","syscebnl_tax_group_5","base","invoice","","+base_sale_5","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_sale_5","","",""
"","","","","","","","","base","refund","","-base_sale_5","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_sale_5","","",""
"syscebnl_tva_purchase_5","5%","","","5.0","percent","purchase","syscebnl_tax_group_5","base","invoice","","+purc_base","","",""
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

## File: data\template\account.tax.group-cf.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_5","VAT 5%","T.V.A. 5%","pcg_4431","pcg_4452"
"tax_group_19","VAT 19%","T.V.A. 19%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-cf_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_5","VAT 5%","T.V.A. 5%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_19","VAT 19%","T.V.A. 19%","syscebnl_443","syscebnl_445"

```

## File: models\template_cf.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cf')
    def _get_cf_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('cf', 'res.company')
    def _get_cf_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cf',
                'account_sale_tax_id': 'tva_sale_19',
                'account_purchase_tax_id': 'tva_purchase_19',
            }
        )
        return company_values

    @template('cf', 'account.account')
    def _get_cf_account_account(self):
        return self._parse_csv('cf', 'account.account', module='l10n_syscohada')


```

## File: models\template_cf_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cf_syscebnl')
    def _get_cf_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('cf_syscebnl', 'res.company')
    def _get_cf_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.cf',
                'account_sale_tax_id': 'syscebnl_tva_sale_19',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_19',
            }
        )
        return company_values

    @template('cf_syscebnl', 'account.account')
    def _get_cf_syscebnl_account_account(self):
        return self._parse_csv('cf_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_cf
from . import template_cf_syscebnl

```

