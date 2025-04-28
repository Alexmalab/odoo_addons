# Odoo Module: l10n_gn

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Guinea - Accounting',
    'countries': ['gn'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This module implements the tax for Guinea.
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
    <record id="account_tax_report_gn" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.gn"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_gn_balance" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="account_tax_report_gn_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_gn_sales" model="account.report.line">
                <field name="name">Outgoing</field>
                <field name="code">GN_SALES</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gn_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GN_TAXABLE.base + GN_EXPORT.base + GN_SALE_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_gn_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GN_TAXABLE.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gn_sales_taxable" model="account.report.line">
                        <field name="name">Taxable operations</field>
                        <field name="code">GN_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_sales_taxable_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_base</field>
                            </record>
                            <record id="account_tax_report_line_gn_sales_taxable_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gn_sales_sales_export" model="account.report.line">
                        <field name="name">Export</field>
                        <field name="code">GN_EXPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_sales_export_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">export</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gn_sales_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">GN_SALE_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_sales_exempt_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">sale_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="account_tax_report_line_gn_purchase" model="account.report.line">
                <field name="name">Incoming</field>
                <field name="code">GN_VAT_DEDUCT</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gn_purchase_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GN_PURC_TAXABLE.base + GN_IMPORT.base + GN_PURC_EXEMPT.base</field>
                    </record>
                    <record id="account_tax_report_line_gn_purchase_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GN_PURC_TAXABLE.tax + GN_IMPORT.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gn_purchase_taxable" model="account.report.line">
                        <field name="name">Taxable</field>
                        <field name="code">GN_PURC_TAXABLE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_purchase_taxable_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GN_GOOD.base + GN_SERVICE.base</field>
                            </record>
                            <record id="account_tax_report_line_gn_purchase_taxable_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GN_GOOD.tax + GN_SERVICE.tax</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_gn_purchase_taxable_goods" model="account.report.line">
                                <field name="name">Goods</field>
                                <field name="code">GN_GOOD</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_gn_purchase_taxable_goods_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">goods_base</field>
                                    </record>
                                    <record id="account_tax_report_line_gn_purchase_taxable_goods_tax_tag" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">goods_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_gn_purchase_taxable_service" model="account.report.line">
                                <field name="name">Services</field>
                                <field name="code">GN_SERVICE</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_gn_purchase_taxable_service_base_tag" model="account.report.expression">
                                        <field name="label">base</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">services_base</field>
                                    </record>
                                    <record id="account_tax_report_line_gn_purchase_taxable_service_tax_tag" model="account.report.expression">
                                        <field name="label">tax</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">services_tax</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gn_purchase_import" model="account.report.line">
                        <field name="name">Import</field>
                        <field name="code">GN_IMPORT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_purchase_import_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_base</field>
                            </record>
                            <record id="account_tax_report_line_gn_purchase_import_tax_tag" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">import_tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gn_purchase_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="code">GN_PURC_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_purchase_exempt_base_tag" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">purc_exempt</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <record id="account_tax_report_line_gn_net" model="account.report.line">
                <field name="name">Net VAT</field>
                <field name="code">GN_NET</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_gn_net_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">GN_VAT_CREDIT.tax + GN_VAT_TO_PAY.tax</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_gn_credit" model="account.report.line">
                        <field name="name">VAT Credit</field>
                        <field name="code">GN_VAT_CREDIT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_credit_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GN_VAT_DEDUCT.tax - GN_SALES.tax</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_gn_to_pay" model="account.report.line">
                        <field name="name">VAT to pay</field>
                        <field name="code">GN_VAT_TO_PAY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_gn_to_pay_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">GN_SALES.tax - GN_VAT_DEDUCT.tax</field>
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

## File: data\template\account.fiscal.position-gn.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_1","1","National","1","","base.gn","","",""
"fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","tva_sale_18","tva_export_0"
"","","","","","","","tva_purchase_good_18","tva_import_0"
"","","","","","","","tva_purchase_service_18","tva_import_0"

```

## File: data\template\account.fiscal.position-gn_syscebnl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"syscebnl_fiscal_position_template_1","1","National","1","","base.gn","","",""
"syscebnl_fiscal_position_template_2","3","International","1","","","","",""
"","","","","","","","syscebnl_tva_sale_18","syscebnl_tva_export_0"
"","","","","","","","syscebnl_tva_purchase_good_18","syscebnl_tva_import_0"
"","","","","","","","syscebnl_tva_purchase_service_18","syscebnl_tva_import_0"

```

## File: data\template\account.tax-gn.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr"
"tva_sale_18","18%","","","18.0","percent","sale","tax_group_18","base","invoice","+sale_base","","",""
"","","","","","","","","tax","invoice","+sale_tax","pcg_4431","",""
"","","","","","","","","base","refund","-sale_base","","",""
"","","","","","","","","tax","refund","-sale_tax","pcg_4431","",""
"tva_purchase_good_18","18% G","18% goods","","18.0","percent","purchase","tax_group_18","base","invoice","+goods_base","","","18% Produits"
"","","","","","","","","tax","invoice","+goods_tax","pcg_4452","",""
"","","","","","","","","base","refund","-goods_base","","",""
"","","","","","","","","tax","refund","-goods_tax","pcg_4452","",""
"tva_purchase_service_18","18% S","18% services","","18.0","percent","purchase","tax_group_18","base","invoice","+services_base","","",""
"","","","","","","","","tax","invoice","+services_tax","pcg_4452","",""
"","","","","","","","","base","refund","-services_base","","",""
"","","","","","","","","tax","refund","-services_tax","pcg_4452","",""
"tva_export_0","0% EX","0% (export)","","0.0","","sale","tax_group_0","base","invoice","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-export","","",""
"","","","","","","","","tax","refund","","","",""
"tva_import_0","0% EX","0% (import)","","0.0","","purchase","tax_group_0","base","invoice","+import_base","","","0% (importation)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-import_base","","",""
"","","","","","","","","tax","refund","","","",""
"tva_exempt_0","0%","0% (exempt)","","0.0","","sale","tax_group_0","base","invoice","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","",""
"tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","tax_group_0","base","invoice","+purc_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-purc_exempt","","",""
"","","","","","","","","tax","refund","","","",""


```

## File: data\template\account.tax-gn_syscebnl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@fr"
"syscebnl_tva_sale_18","18%","","","18.0","percent","sale","syscebnl_tax_group_18","base","invoice","+sale_base","","",""
"","","","","","","","","tax","invoice","+sale_tax","syscebnl_443","",""
"","","","","","","","","base","refund","-sale_base","","",""
"","","","","","","","","tax","refund","-sale_tax","syscebnl_443","",""
"syscebnl_tva_purchase_good_18","18% G","18% goods","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","+goods_base","","","18% Produits"
"","","","","","","","","tax","invoice","+goods_tax","syscebnl_445","",""
"","","","","","","","","base","refund","-goods_base","","",""
"","","","","","","","","tax","refund","-goods_tax","syscebnl_445","",""
"syscebnl_tva_purchase_service_18","18% S","18% services","","18.0","percent","purchase","syscebnl_tax_group_18","base","invoice","+services_base","","",""
"","","","","","","","","tax","invoice","+services_tax","syscebnl_445","",""
"","","","","","","","","base","refund","-services_base","","",""
"","","","","","","","","tax","refund","-services_tax","syscebnl_445","",""
"syscebnl_tva_export_0","0% EX","0% (export)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+export","","","0% (exportation)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-export","","",""
"","","","","","","","","tax","refund","","","",""
"syscebnl_tva_import_0","0% EX","0% (import)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","+import_base","","","0% (importation)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-import_base","","",""
"","","","","","","","","tax","refund","","","",""
"syscebnl_tva_exempt_0","0%","0% (exempt)","","0.0","","sale","syscebnl_tax_group_0","base","invoice","+sale_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-sale_exempt","","",""
"","","","","","","","","tax","refund","","","",""
"syscebnl_tva_purchase_exempt_0","0%","0% (exempt)","","0.0","","purchase","syscebnl_tax_group_0","base","invoice","+purc_exempt","","","0% (exonéré)"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-purc_exempt","","",""
"","","","","","","","","tax","refund","","","",""


```

## File: data\template\account.tax.group-gn.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","VAT 0%","T.V.A. 0%","pcg_4431","pcg_4452"
"tax_group_18","VAT 18%","T.V.A. 18%","pcg_4431","pcg_4452"

```

## File: data\template\account.tax.group-gn_syscebnl.csv

```csv
"id","name","name@fr","tax_payable_account_id","tax_receivable_account_id"
"syscebnl_tax_group_0","VAT 0%","T.V.A. 0%","syscebnl_443","syscebnl_445"
"syscebnl_tax_group_18","VAT 18%","T.V.A. 18%","syscebnl_443","syscebnl_445"

```

## File: models\template_gn.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gn')
    def _get_gn_template_data(self):
        return {
            'name': _('SYSCOHADA for Companies'),
            'parent': 'syscohada',
            'code_digits': '6',
        }

    @template('gn', 'res.company')
    def _get_gn_res_company(self):
        company_values = super()._get_syscohada_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.gn',
                'account_sale_tax_id': 'tva_sale_18',
                'account_purchase_tax_id': 'tva_purchase_good_18',
            }
        )
        return company_values

    @template('gn', 'account.account')
    def _get_gn_account_account(self):
        return self._parse_csv('gn', 'account.account', module='l10n_syscohada')

```

## File: models\template_gn_syscebnl.py

```python
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gn_syscebnl')
    def _get_gn_syscebnl_template_data(self):
        return {
            'name': _('SYSCEBNL for Associations'),
            'parent': 'syscebnl',
            'code_digits': '6',
        }

    @template('gn_syscebnl', 'res.company')
    def _get_gn_syscebnl_res_company(self):
        company_values = super()._get_syscebnl_res_company()
        company_values[self.env.company.id].update(
            {
                'account_fiscal_country_id': 'base.gn',
                'account_sale_tax_id': 'syscebnl_tva_sale_18',
                'account_purchase_tax_id': 'syscebnl_tva_purchase_good_18',
            }
        )
        return company_values

    @template('gn_syscebnl', 'account.account')
    def _get_gn_syscebnl_account_account(self):
        return self._parse_csv('gn_syscebnl', 'account.account', module='l10n_syscohada')

```

## File: models\__init__.py

```python
from . import template_gn
from . import template_gn_syscebnl

```

