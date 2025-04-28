# Odoo Module: l10n_my

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Malaysia - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['my'],
    'author': 'Odoo PS',
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Malaysia in Odoo.
==============================================================================
    """,
    'depends': [
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_tax_report_data.xml',
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
    <record id="tax_report_vat" model="account.report">
        <field name="name">SST-02 (B2)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.my"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_my_tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_my_sst_taxable_per_rate" model="account.report.line">
                <field name="name">11) Total Value of Tax Payable as Per Tax Rate</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_my_sst_taxable_5" model="account.report.line">
                        <field name="name">a) Taxable Goods at 5% Rate</field>
                        <field name="code">SST_5</field>
                       <field name="expression_ids">
                            <record id="l10n_my_sst_taxable_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">A</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_my_sst_taxable_10" model="account.report.line">
                        <field name="name">b) Taxable Goods at 10% Rate</field>
                        <field name="code">SST_10</field>
                       <field name="expression_ids">
                            <record id="l10n_my_sst_taxable_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">B</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_my_sst_services_other_than_group_h" model="account.report.line">
                        <field name="name">c) Taxable Services other than from Group H</field>
                        <field name="code">SST_6</field>
                       <field name="expression_ids">
                            <record id="l10n_my_sst_services_other_than_group_h_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">C</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_my_sst_services_group_h" model="account.report.line">
                        <field name="name">d) Taxable Services from Group H</field>
                        <field name="code">SST_H</field>
                       <field name="expression_ids">
                            <record id="l10n_my_sst_services_group_h_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">D</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_my_sst_tax_payable_total_value" model="account.report.line">
                <field name="name">12) Total Value of Tax Payable</field>
                <field name="hierarchy_level">0</field>
                <field name="code">SST_PTV</field>
                <field name="aggregation_formula">SST_5.balance + SST_10.balance + SST_6.balance + SST_H.balance</field>
            </record>
            <record id="l10n_my_sst_tax_deduction" model="account.report.line">
                <field name="name">13) Amount of Tax Deduction</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_my_sst_tax_deduction_from_cn" model="account.report.line">
                        <field name="name">a) Tax Deduction from Credit Note</field>
                        <field name="code">SST_CN</field>
                       <field name="expression_ids">
                            <record id="l10n_my_sst_tax_deduction_cn_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CN</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_my_sst_sale_tax_deduction" model="account.report.line">
                        <field name="name">b) Sales Tax Deduction</field>
                        <field name="code">SST_SAD</field>
                       <field name="expression_ids">
                            <record id="l10n_my_sst_sale_tax_deduction_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAD</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_my_sst_service_tax_deduction" model="account.report.line">
                        <field name="name">c) Service Tax Deduction</field>
                        <field name="code">SST_SED</field>
                       <field name="expression_ids">
                            <record id="l10n_my_sst_service_tax_deduction_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SED</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_my_sst_tax_deduction_adjustment" model="account.report.line">
                <field name="name">13A) Adjustment under Sales Tax Deduction</field>
                <field name="hierarchy_level">0</field>
                <field name="code">SST_TDA</field>
                <field name="expression_ids">
                    <record id="l10n_my_sst_ttax_deduction_adjustment_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">DA</field>
                    </record>
                </field>
            </record>
            <record id="l10n_my_sst_tax_payable_total_before_penalty" model="account.report.line">
                <field name="name">14) Total Tax Payable Before Penalty Imposed</field>
                <field name="hierarchy_level">0</field>
                <field name="code">SST_PTBP</field>
                <field name="aggregation_formula">SST_PTV.balance - SST_CN.balance + SST_SAD.balance + SST_SED.balance + SST_TDA.balance</field>
            </record>
            <record id="l10n_my_sst_penalty_amount" model="account.report.line">
                <field name="name">15) Penalty Rate / Penalty Amount</field>
                <field name="hierarchy_level">0</field>
                <field name="code">SST_P</field>
                <field name="expression_ids">
                    <record id="l10n_my_sst_penalty_amount_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">P</field>
                    </record>
                </field>
            </record>
            <record id="l10n_my_sst_total_including_penalty" model="account.report.line">
                <field name="name">16) Total of Tax Payable Inclusive Penalty</field>
                <field name="hierarchy_level">0</field>
                <field name="aggregation_formula">SST_PTBP.balance + SST_P.balance</field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-my.csv

```csv
"id","name","code","account_type","tag_ids","reconcile"
"l10n_my_11","Fixed Assets","11","asset_fixed","","False"
"l10n_my_1110","Furniture and Fixtures","1110","asset_non_current","","False"
"l10n_my_1130","Equipment","1130","asset_non_current","","False"
"l10n_my_1140","Decoration","1140","asset_non_current","","False"
"l10n_my_1160","Investments","1160","asset_non_current","","False"
"l10n_my_12","Current Assets","12","asset_current","","False"
"l10n_my_1240","Account Receivable","1240","asset_receivable","","True"
"l10n_my_1241","Utility & Rental Deposit","1241","asset_current","","True"
"l10n_my_1242","Supplier Prepayments","1242","asset_current","","False"
"l10n_my_1243","Account Receivable (PoS)","1243","asset_receivable","","True"
"l10n_my_1245","Sundry Deposits","1245","asset_current","","False"
"l10n_my_1246","Other Receivable","1246","asset_receivable","","True"
"l10n_my_1250","Stock Interim Account (Received)","1250","asset_current","","False"
"l10n_my_1260","Stock Interim Account (Delivered)","1260","asset_current","","False"
"l10n_my_1270","Stock Valuation Account","1270","asset_current","","False"
"l10n_my_21","Non-current Liabilities","21","liability_non_current","","False"
"l10n_my_22","Current Liabilities","22","liability_current","","False"
"l10n_my_2210","Accruals","2210","liability_current","","False"
"l10n_my_22101","Accruals - KWSP","22101","liability_current","","False"
"l10n_my_22102","Accruals - PCB","22102","liability_current","","False"
"l10n_my_22103","Accruals - SOCSO","22103","liability_current","","False"
"l10n_my_22104","Accruals - EIS","22104","liability_current","","False"
"l10n_my_2211","Account Payable","2211","liability_payable","","True"
"l10n_my_2212","Receipt in Advance (Customer Prepayments)","2212","liability_current","","False"
"l10n_my_2213","SST Control Account","2213","liability_current","","False"
"l10n_my_2214","Provision for Taxation","2214","liability_current","","False"
"l10n_my_2215","Proposed Dividend","2215","liability_current","","False"
"l10n_my_2216","Other Payable","2216","liability_payable","","True"
"l10n_my_31","Paid Capital","31","liability_current","","False"
"l10n_my_32","Accumulated Profit & Loss","32","liability_current","","False"
"l10n_my_33","Profit & Loss Account","33","liability_current","","False"
"l10n_my_34","Short-term Borrowing","34","liability_current","","False"
"l10n_my_41","Trade Income","41","income","account.account_tag_operating","False"
"l10n_my_42","Other Income and Gains","42","income","account.account_tag_operating","False"
"l10n_my_4210","Sundry Income","4210","income","account.account_tag_operating","False"
"l10n_my_4220","Exchange Adjustment","4220","income","account.account_tag_operating","False"
"l10n_my_4230","Bank Interest Income","4230","income","account.account_tag_operating","False"
"l10n_my_4240","Foreign Exchange Gain","4240","income","account.account_tag_operating","False"
"l10n_my_51","Costs","51","expense","account.account_tag_operating","False"
"l10n_my_5101","Trade Costs","5101","expense","account.account_tag_operating","False"
"l10n_my_5105","Misc. Costs","5105","expense","account.account_tag_operating","False"
"l10n_my_5106","Costs of Transportation","5106","expense","account.account_tag_operating","False"
"l10n_my_5111","Declaration Fees","5111","expense","account.account_tag_operating","False"
"l10n_my_5112","Packing Fees","5112","expense","account.account_tag_operating","False"
"l10n_my_52","Expenses","52","expense","account.account_tag_operating","False"
"l10n_my_5201","Bank Charges","5201","expense","account.account_tag_operating","False"
"l10n_my_5202","Entertainment","5202","expense","account.account_tag_operating","False"
"l10n_my_5203","Electricity & Water Fees","5203","expense","account.account_tag_operating","False"
"l10n_my_5205","Postage & Stamps","5205","expense","account.account_tag_operating","False"
"l10n_my_5206","Printing & Stationery","5206","expense","account.account_tag_operating","False"
"l10n_my_5207","Rent & Rates","5207","expense","account.account_tag_operating","False"
"l10n_my_5208","Sundry Expenses","5208","expense","account.account_tag_operating","False"
"l10n_my_5209","Telecommunication Expenses","5209","expense","account.account_tag_operating","False"
"l10n_my_5210","Traffic Fees","5210","expense","account.account_tag_operating","False"
"l10n_my_5211","IT Expenses","5211","expense","account.account_tag_operating","False"
"l10n_my_5214","Insurance","5214","expense","account.account_tag_operating","False"
"l10n_my_5215","Sales Commission","5215","expense","account.account_tag_operating","False"
"l10n_my_5216","Overseas Traveling","5216","expense","account.account_tag_operating","False"
"l10n_my_5218","Wages & Salaries","5218","expense","account.account_tag_operating","False"
"l10n_my_52181","KWSP Contribution","52181","expense","account.account_tag_operating","False"
"l10n_my_52182","PCB Contribution","52182","expense","account.account_tag_operating","False"
"l10n_my_52183","EIS Contribution","52183","expense","account.account_tag_operating","False"
"l10n_my_5219","Bonus Payment","5219","expense","account.account_tag_operating","False"
"l10n_my_5221","SST","5221","expense","account.account_tag_operating","False"
"l10n_my_5222","Local Delivery","5222","expense","account.account_tag_operating","False"
"l10n_my_5223","Management Fees","5223","expense","account.account_tag_operating","False"
"l10n_my_5224","Depreciation","5224","expense","account.account_tag_operating","False"
"l10n_my_5225","Audit Fees","5225","expense","account.account_tag_operating","False"
"l10n_my_5226","Bad Debts","5226","expense","account.account_tag_operating","False"
"l10n_my_5228","Legal & Professional Fees","5228","expense","account.account_tag_operating","False"
"l10n_my_5229","Dividend","5229","expense","account.account_tag_operating","False"
"l10n_my_5231","Disposal","5231","expense","account.account_tag_operating","False"
"l10n_my_5234","Repair and Maintenance","5234","expense","account.account_tag_operating","False"
"l10n_my_5235","Advertising","5235","expense","account.account_tag_operating","False"
"l10n_my_5240","Foreign Exchange Loss","5240","expense","account.account_tag_operating","False"

```

## File: data\template\account.tax-my.csv

```csv
"id","description","sequence","active","name","invoice_label","type_tax_use","amount_type","tax_scope","amount","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids"
"l10n_my_tax_sale_5","SST 5%","1","True","5%","5%","sale","percent","consu","5.0","tax_group_sst","100","base","invoice","",""
"","","","","","","","","","","","100","tax","invoice","l10n_my_2213","+A"
"","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","100","tax","refund","l10n_my_2213","+CN"
"l10n_my_tax_sale_6","SST 6%","1","False","6%","6%","sale","percent","service","6.0","tax_group_sst","100","base","invoice","",""
"","","","","","","","","","","","100","tax","invoice","l10n_my_2213","+C"
"","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","100","tax","refund","l10n_my_2213","+CN"
"l10n_my_tax_sale_8","SST 8%","1","True","8%","8%","sale","percent","service","8.0","tax_group_sst","100","base","invoice","",""
"","","","","","","","","","","","100","tax","invoice","l10n_my_2213","+C"
"","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","100","tax","refund","l10n_my_2213","+CN"
"l10n_my_tax_sale_10","SST 10%","1","True","10%","10%","sale","percent","consu","10.0","tax_group_sst","100","base","invoice","",""
"","","","","","","","","","","","100","tax","invoice","l10n_my_2213","+B"
"","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","100","tax","refund","l10n_my_2213","+CN"

```

## File: data\template\account.tax.group-my.csv

```csv
"id","name","country_id"
"tax_group_sst","SST","base.my"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    """ Update taxes for existing companies, in order to apply the new tags to them. """
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'my')], order="parent_path"):
        env['account.chart.template'].try_loading('my', company)

```

## File: models\template_my.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('my')
    def _get_my_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_my_1240',
            'property_account_payable_id': 'l10n_my_2211',
            'property_account_income_categ_id': 'l10n_my_41',
            'property_account_expense_categ_id': 'l10n_my_51',
            'code_digits': '6',
        }

    @template('my', 'res.company')
    def _get_my_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.my',
                'bank_account_code_prefix': '1200',
                'cash_account_code_prefix': '1210',
                'transfer_account_code_prefix': '111220',
                'account_default_pos_receivable_account_id': 'l10n_my_1243',
                'income_currency_exchange_account_id': 'l10n_my_4240',
                'expense_currency_exchange_account_id': 'l10n_my_5240',
                'account_sale_tax_id': 'l10n_my_tax_sale_10',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_my

```

