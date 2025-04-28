# Odoo Module: l10n_td

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Tchad - Accounting",
    'countries': ['td'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Tchad.
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
    <record id="account_tax_report_td" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.td"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_td_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_td_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_td_sales" model="account.report.line">
                <field name="name">Outgoing</field>
                <field name="code">TD_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_td_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TD_TAXABLE_18.base + TD_TAXABLE_9.base + TD_EXPORT.base + TD_SALE_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_td_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TD_TAXABLE_18.tax + TD_TAXABLE_9.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_sales_18" model="account.report.line">
                        <field name="name">Taxable operations at 18%</field>
                        <field name="code">TD_TAXABLE_18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_18_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_18</field>
                            </record>
                            <record id="account_tax_report_line_sales_18_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_18</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sales_9" model="account.report.line">
                        <field name="name">Taxable operations at 9%</field>
                        <field name="code">TD_TAXABLE_9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_9_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">base_9</field>
                            </record>
                            <record id="account_tax_report_line_sales_9_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_9</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_sales_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">TD_EXPORT</field>
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
                        <field name="code">TD_SALE_EXEMPT</field>
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
            <record id="account_tax_report_line_td_purchases" model="account.report.line">
                <field name="name">Incoming</field>
                <field name="code">TD_VAT_DEDUCT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_td_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TD_PURC_TAXABLE.base + TD_IMPORT.base + TD_PURC_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_td_purchase_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TD_PURC_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_td_purchases_taxable" model="account.report.line">
                        <field name="name">Taxable</field>
                        <field name="code">TD_PURC_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_td_purchases_taxable_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_base</field>
                            </record>
                            <record id="account_tax_report_line_td_purchases_taxable_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_td_purchases_import" model="account.report.line">
                        <field name="name">Import</field>
                        <field name="code">TD_IMPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_td_purchases_import_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_base</field>
                            </record>
                            <record id="account_tax_report_line_td_purchases_import_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_td_purchases_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">TD_PURC_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_td_purchases_exempt_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="account_tax_report_line_td_net" model="account.report.line">
                <field name="name">Net VAT</field>
                <field name="code">TD_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_td_net_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">TD_VAT_CREDIT.tax + TD_VAT_TO_PAY.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_td_credit" model="account.report.line">
                        <field name="name">VAT Credit</field>
                        <field name="code">TD_VAT_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_td_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TD_VAT_DEDUCT.tax - TD_SALES.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_td_to_pay" model="account.report.line">
                        <field name="name">VAT to pay</field>
                        <field name="code">TD_VAT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_td_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TD_SALES.tax - TD_VAT_DEDUCT.tax</field>
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

## File: data\template\account.fiscal.position-td.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.td","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18","tva_export_0"
"","","","","","","","tva_sale_9","tva_export_0"
"","","","","","","","tva_purchase_18","tva_import_0"
"","","","","","","","tva_purchase_9","tva_import_0"

```

## File: data\template\account.fiscal.position-td_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.td","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_sale_9","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_18","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_9","syscebnl_tva_import_0"

```

## File: data\template\account.tax-td.csv

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
"tva_sale_9","9%","","","9.0","percent","sale","tax_group_9","base","invoice","","+base_9","","",""
"","","","","","","","","tax","invoice","pcg_4431","+tax_9","","",""
"","","","","","","","","base","refund","","-base_9","","",""
"","","","","","","","","tax","refund","pcg_4431","-tax_9","","",""
"tva_purchase_9","9%","","","9.0","percent","purchase","tax_group_9","base","invoice","","+purc_base","","",""
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

## File: data\template\account.tax-td_syscebnl.csv

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
"syscebnl_tva_sale_9","9%","","","9.0","percent","sale","syscebnl_tax_group_9","base","invoice","","+base_9","","",""
"","","","","","","","","tax","invoice","syscebnl_443","+tax_9","","",""
"","","","","","","","","base","refund","","-base_9","","",""
"","","","","","","","","tax","refund","syscebnl_443","-tax_9","","",""
"syscebnl_tva_purchase_9","9%","","","9.0","percent","purchase","syscebnl_tax_group_9","base","invoice","","+purc_base","","",""
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

## File: data\template\account.tax.group-td.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_9","VAT 9%","T.V.A. 9%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-td_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_9","VAT 9%","T.V.A. 9%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_td.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('td')
    def _get_td_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('td', 'res.company')
    def _get_td_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.td',
                'account_sale_tax_id': 'tva_sale_18',
                'account_purchase_tax_id': 'tva_purchase_18',
            }
        )
        return company_values

    @template('td', 'account.account')
    def _get_td_account_account(self):
        return self._parse_csv('td', 'account.account', module='l10n_syscohada')

```

## File: models\template_td_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('td_syscebnl')
    def _get_td_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('td_syscebnl', 'res.company')
    def _get_td_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.td',
                'account_sale_tax_id': 'syscebnl_tva_sale_18',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_18',
            }
        )
        return company_values

    @template('td_syscebnl', 'account.account')
    def _get_td_syscebnl_account_account(self):
        return self._parse_csv('td_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_td
from . import template_td_syscebnl

```

