# Odoo Module: l10n_et

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
    'name': 'Ethiopia - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['et'],
    'version': '2.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Base Module for Ethiopian Localization
======================================

This is the latest Ethiopian Odoo localization and consists of:
    - Chart of Accounts
    - VAT tax structure
    - Withholding tax structure
    - Regional State listings
    """,
    'author': 'Michael Telahun Makonnen <mmakonnen@gmail.com>',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
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
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.et"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_purch_vt" model="account.report.line">
                <field name="name">Taxable Purchases - VAT</field>
                <field name="aggregation_formula">ET_TAXABLE_PURCHASE_VAT_OUT_OF_SCOPE.balance + ET_TAXABLE_PURCHASE_VAT_EXEMPT.balance + ET_TAXABLE_PURCHASE_VAT_RATED_0.balance + ET_TAXABLE_PURCHASE_VAT_RATED_15.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_purch_vt_out_scope" model="account.report.line">
                        <field name="name">Taxable Purchase VAT Out of Scope</field>
                        <field name="code">ET_TAXABLE_PURCHASE_VAT_OUT_OF_SCOPE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_purch_vt_out_scope_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Purchase VAT Out of Scope</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_purch_vt_exmpt" model="account.report.line">
                        <field name="name">Taxable Purchase VAT Exempt</field>
                        <field name="code">ET_TAXABLE_PURCHASE_VAT_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_purch_vt_exmpt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Purchase VAT Exempt</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_purch_vt_ratd_0" model="account.report.line">
                        <field name="name">Taxable Purchase VAT Rated 0%</field>
                        <field name="code">ET_TAXABLE_PURCHASE_VAT_RATED_0</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_purch_vt_ratd_0_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Purchase VAT Rated 0%</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_purch_vt_ratd_15" model="account.report.line">
                        <field name="name">Taxable Purchase VAT Rated 15%</field>
                        <field name="code">ET_TAXABLE_PURCHASE_VAT_RATED_15</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_purch_vt_ratd_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Purchase VAT Rated 15%</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_purch_withholding" model="account.report.line">
                <field name="name">Taxable Purchases - Witholding</field>
                <field name="aggregation_formula">ET_TAXABLE_ET_2_WITHHOLDING_ON_PURCHASES.balance + ET_TAXABLE_ET_35_WITHHOLDING_ON_PURCHASES.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_purch_withholding_2" model="account.report.line">
                        <field name="name">Taxable 2% Withholding on Purchases</field>
                        <field name="code">ET_TAXABLE_ET_2_WITHHOLDING_ON_PURCHASES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_purch_withholding_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable 2% Withholding on Purchases</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_purch_withholding_35" model="account.report.line">
                        <field name="name">Taxable 35% Withholding on Purchases</field>
                        <field name="code">ET_TAXABLE_ET_35_WITHHOLDING_ON_PURCHASES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_purch_withholding_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable 35% Withholding on Purchases</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_sale_vat" model="account.report.line">
                <field name="name">Taxable Sales - VAT</field>
                <field name="aggregation_formula">ET_TAXABLE_SALES_VAT_OUT_OF_SCOPE_SALES.balance + ET_TAXABLE_SALES_VAT_EXEMPT.balance + ET_TAXABLE_SALES_VAT_RATED_0.balance + ET_TAXABLE_ET_SALES_VAT_RATED_15.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_sale_vat_out_scope" model="account.report.line">
                        <field name="name">Taxable Sales VAT Out of Scope (Sales)</field>
                        <field name="code">ET_TAXABLE_SALES_VAT_OUT_OF_SCOPE_SALES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_sale_vat_out_scope_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Sales VAT Out of Scope (Sales)</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_sale_vat_exmpt" model="account.report.line">
                        <field name="name">Taxable Sales VAT Exempt</field>
                        <field name="code">ET_TAXABLE_SALES_VAT_EXEMPT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_sale_vat_exmpt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Sales VAT Exempt</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_sale_vat_rated_0" model="account.report.line">
                        <field name="name">Taxable Sales VAT Rated 0%</field>
                        <field name="code">ET_TAXABLE_SALES_VAT_RATED_0</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_sale_vat_rated_0_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Sales VAT Rated 0%</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_sale_vat_rated_15" model="account.report.line">
                        <field name="name">Taxable Sales VAT Rated 15%</field>
                        <field name="code">ET_TAXABLE_ET_SALES_VAT_RATED_15</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_sale_vat_rated_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable Sales VAT Rated 15%</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_sale_withholding" model="account.report.line">
                <field name="name">Taxable Sales - Withholding</field>
                <field name="aggregation_formula">ET_TAXABLE_2_WITHHOLDING_ON_SALES.balance + ET_TAXABLE_35_WITHHOLDING_ON_SALES.balance + ET_TAXABLE_VAT_WITHHOLDING_ON_SALES.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_sale_withholding_2" model="account.report.line">
                        <field name="name">Taxable 2% Withholding on Sales</field>
                        <field name="code">ET_TAXABLE_2_WITHHOLDING_ON_SALES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_sale_withholding_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable 2% Withholding on Sales</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_sale_withholding_35" model="account.report.line">
                        <field name="name">Taxable 35% Withholding on Sales</field>
                        <field name="code">ET_TAXABLE_35_WITHHOLDING_ON_SALES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_sale_withholding_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable 35% Withholding on Sales</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_sale_vat_withholding" model="account.report.line">
                        <field name="name">Taxable VAT Withholding on Sales</field>
                        <field name="code">ET_TAXABLE_VAT_WITHHOLDING_ON_SALES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_sale_vat_withholding_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Taxable VAT Withholding on Sales</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_net_vat" model="account.report.line">
                <field name="name">Net VAT to be Paid/Reclaimed</field>
                <field name="aggregation_formula">ET_PURCHASE_VAT.balance + ET_SALES_VAT.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_purch_vat" model="account.report.line">
                        <field name="name">Purchase VAT</field>
                        <field name="code">ET_PURCHASE_VAT</field>
                        <field name="aggregation_formula">ET_PURCHASE_VAT_RATED_15.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_purch_vat_rated_15" model="account.report.line">
                                <field name="name">Purchase VAT Rated 15%</field>
                                <field name="code">ET_PURCHASE_VAT_RATED_15</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_purch_vat_rated_15_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Purchase VAT Rated 15%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_net_sale_vat" model="account.report.line">
                        <field name="name">Sales VAT</field>
                        <field name="code">ET_SALES_VAT</field>
                        <field name="aggregation_formula">ET_SALES_VAT_RATED_15.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_net_sale_vat_15" model="account.report.line">
                                <field name="name">Sales VAT Rated 15%</field>
                                <field name="code">ET_SALES_VAT_RATED_15</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_net_sale_vat_15_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Sales VAT Rated 15%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_net_purch_witholding" model="account.report.line">
                <field name="name">Withholding on Purchases</field>
                <field name="aggregation_formula">ET_2_WITHHOLDING_ON_PURCHASES.balance + ET_35_WITHHOLDING_ON_PURCHASES.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_net_purch_witholding_2" model="account.report.line">
                        <field name="name">2% Withholding on Purchases</field>
                        <field name="code">ET_2_WITHHOLDING_ON_PURCHASES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_net_purch_witholding_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2% Withholding on Purchases</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_net_purch_witholding_35" model="account.report.line">
                        <field name="name">35% Withholding on Purchases</field>
                        <field name="code">ET_35_WITHHOLDING_ON_PURCHASES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_net_purch_witholding_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">35% Withholding on Purchases</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_net_sale_witholding" model="account.report.line">
                <field name="name">Withholding on Sales</field>
                <field name="aggregation_formula">ET_2_WITHHELD_ON_SALES.balance + ET_35_WITHHELD_ON_SALES.balance + ET_VAT_WITHHELD_ON_SALES.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_net_sale_witheld_2" model="account.report.line">
                        <field name="name">2% Withheld on Sales</field>
                        <field name="code">ET_2_WITHHELD_ON_SALES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_net_sale_witheld_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2% Withheld on Sales</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_net_sale_witheld_35" model="account.report.line">
                        <field name="name">35% Withheld on Sales</field>
                        <field name="code">ET_35_WITHHELD_ON_SALES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_net_sale_witheld_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">35% Withheld on Sales</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_net_sale_vat_witheld" model="account.report.line">
                        <field name="name">VAT Withheld on Sales</field>
                        <field name="code">ET_VAT_WITHHELD_ON_SALES</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_net_sale_vat_witheld_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Withheld on Sales</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-et.csv

```csv
"id","code","name","account_type","reconcile"
"l10n_et1100","1100","Sales of Goods and Services","income",""
"l10n_et120001","120001","Cash Discount Gain","income_other",""
"l10n_et2102","2102","Cash at bank in foreigh currency","asset_current",""
"l10n_et2104","2104","Letter of Credit restricted account","asset_current",""
"l10n_et2201","2201","Suspense","asset_receivable","True"
"l10n_et2202","2202","Cash shortage","asset_receivable","True"
"l10n_et2203","2203","Advance to staff","asset_receivable","True"
"l10n_et2204","2204","Cash Registers","asset_receivable","True"
"l10n_et2211","2211","Trade Debtors","asset_receivable","True"
"l10n_et2212","2212","VAT Receivable on Purchases","liability_current",""
"l10n_et2213","2213","Withholding Receivable on Sales","liability_current",""
"l10n_et2214","2214","VAT Withholding Receivable on Sales","liability_current",""
"l10n_et2215","2215","Trade Debtors (PoS)","asset_receivable","True"
"l10n_et2251","2251","Advance to contractors","asset_current",""
"l10n_et2252","2252","Advance to consultant","asset_current",""
"l10n_et2253","2253","Advance to supplier","asset_current",""
"l10n_et2274","2254","Other Debtors","asset_current",""
"l10n_et2301","2301","Goods in Transit","asset_current",""
"l10n_et2351","2351","Stock","asset_current",""
"l10n_et2411","2411","Work in Progress","asset_current",""
"l10n_et2412","2412","Finished Goods","asset_current",""
"l10n_et2501","2501","Construction of buildings","asset_fixed",""
"l10n_et2503","2503","Construction of infrastructure","asset_fixed",""
"l10n_et2521","2521","Vehicles and other vehicular transport","asset_fixed",""
"l10n_et2522","2522","Aircraft, etc","asset_fixed",""
"l10n_et2523","2523","Plant machinery and equipment","asset_fixed",""
"l10n_et2525","2525","Buildings","asset_fixed",""
"l10n_et2527","2527","Infrastructure","asset_fixed",""
"l10n_et2529","2529","Furnishings and fixtures","asset_fixed",""
"l10n_et2530","2530","Livestock and tansport animals","asset_fixed",""
"l10n_et3001","3001","Grace period payables","liability_payable","True"
"l10n_et3002","3002","Trade Creditors","liability_payable","True"
"l10n_et3003","3003","Pension contribution payable","liability_payable","True"
"l10n_et3004","3004","Salary payable","liability_payable","True"
"l10n_et3006","3006","Witholding Payable","asset_current",""
"l10n_et3007","3007","VAT Payable","asset_current",""
"l10n_et3008","3008","Federal Income Tax","asset_current",""
"l10n_et3054","3054","Other deposits","liability_current",""
"l10n_et3061","3061","Retention on contract","liability_current",""
"l10n_et3103","3103","Commercial Loan","liability_current",""
"l10n_et3153","3153","Commercial Loan","liability_current",""
"l10n_et4001","4010","Share capital / equity","equity",""
"l10n_et4004","4020","Reserves","equity",""
"l10n_et5111","5111","Cost of Goods and Services","expense",""
"l10n_et5901","5901","Inventory Adjustments","expense",""
"l10n_et5911","5911","Purchase Returns and Allowances","expense",""
"l10n_et5921","5921","Other","expense",""
"l10n_et6111","6111","Salaries to permanent staff","expense",""
"l10n_et6113","6113","Wages to contract staff","expense",""
"l10n_et6114","6114","Wages to casual staff","expense",""
"l10n_et6115","6115","Wages to external contract staff","expense",""
"l10n_et6116","6116","Miscellaneous payments to staff","expense",""
"l10n_et6121","6121","Allowances to permanent staff","expense",""
"l10n_et6123","6123","Allowances to contract staff","expense",""
"l10n_et6124","6124","Allowances to external contract staff","expense",""
"l10n_et6131","6131","Contribution to permanent staff pensions","expense",""
"l10n_et6211","6211","Uniforms, bedding","expense",""
"l10n_et6212","6212","Office supplies","expense",""
"l10n_et6213","6213","Printing","expense",""
"l10n_et6214","6214","Medical supplies","expense",""
"l10n_et6215","6215","Educational supplies","expense",""
"l10n_et6216","6216","Food","expense",""
"l10n_et6217","6217","Fuel and lubricants","expense",""
"l10n_et6218","6218","Other material and supplies","expense",""
"l10n_et6219","6219","Miscellaneous equipment","expense",""
"l10n_et6221","6221","Agriculture, forestry and marine inputs","expense",""
"l10n_et6222","6222","Veterinary supplies and drugs","expense",""
"l10n_et6223","6223","Research and development supplies","expense",""
"l10n_et6231","6231","Per diem","expense",""
"l10n_et6232","6232","Transport fees","expense",""
"l10n_et6233","6233","Official entertainment","expense",""
"l10n_et6241","6241","Maintenance and repair of vehicles and other transport","expense",""
"l10n_et6243","6243","Maintenance and repair of plant, and equipment","expense",""
"l10n_et6244","6244","Maintenance and repair of buildings, furnishings and fixtures","expense",""
"l10n_et6245","6245","Maintenance and repair of infrastructure","expense",""
"l10n_et6251","6251","Contracted professional services","expense",""
"l10n_et6252","6252","Rent","expense",""
"l10n_et6253","6253","Advertising","expense",""
"l10n_et6254","6254","Insurance","expense",""
"l10n_et6255","6255","Freight","expense",""
"l10n_et6256","6256","Fees and charges","expense",""
"l10n_et6257","6257","Electricity charges","expense",""
"l10n_et6258","6258","Telecommunication charges","expense",""
"l10n_et6259","6259","Water and other utilities","expense",""
"l10n_et6271","6271","Local training","expense",""
"l10n_et6272","6272","External training","expense",""
"l10n_et6311","6311","Depreciation of vehicles and other vehicular transport","expense",""
"l10n_et6313","6313","Depreciation of plant, machinery and equipment","expense",""
"l10n_et6314","6314","Depreciation of buildings, furnishings and fixtures","expense",""
"l10n_et6315","6315","Depreciation of livestock and transport animals","expense",""
"l10n_et6321","6321","Pre-construction activities","expense",""
"l10n_et6322","6322","Construction of buildings","expense",""
"l10n_et6324","6324","Construction of infrastructure","expense",""
"l10n_et6431","6431","Payments on the principal of foreign debt","expense",""
"l10n_et6432","6432","Payments of interest and bank charges on foreign debt","expense",""
"l10n_et6433","6433","Payments on the principal of local debt","expense",""
"l10n_et6434","6434","Payments of interest and bank charges on local debt","expense",""
"l10n_et6435","1200","Foreign Exchange Currency Gain Account","income_other",""
"l10n_et6436","6260","Foreign Exchange Currency Loss Account","expense",""
"l10n_et626001","626001","Cash Discount Loss","expense",""

```

## File: data\template\account.tax-et.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","price_include_override"
"id_tax03","15%","VAT 15% rated sales","tax03","15.0","percent","sale","tax_group_vat_15","base","invoice","-Taxable Sales VAT Rated 15%","",""
"","","","","","","","","tax","invoice","-Sales VAT Rated 15%","l10n_et3007",""
"","","","","","","","","base","refund","+Taxable Sales VAT Rated 15%","",""
"","","","","","","","","tax","refund","+Sales VAT Rated 15%","l10n_et3007",""
"id_tax04","0%","VAT 0% rated sales","tax04","0.0","percent","sale","tax_group_vat_0","base","invoice","-Taxable Sales VAT Rated 0%","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","+Taxable Sales VAT Rated 0%","",""
"","","","","","","","","tax","refund","","",""
"id_tax06","0% EXEMPT","VAT Exempt rated sales","tax06","0.0","percent","sale","tax_group_vat_0","base","invoice","+Taxable Sales VAT Exempt","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Taxable Sales VAT Exempt","",""
"","","","","","","","","tax","refund","","",""
"id_tax11","0% Out","VAT Out of Scope rated sales","tax11","0.0","percent","sale","tax_group_vat_0","base","invoice","+Taxable Sales VAT Out of Scope (Sales)","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Taxable Sales VAT Out of Scope (Sales)","",""
"","","","","","","","","tax","refund","","",""
"id_tax02","2% WH","Withholding 2% rated sales","tax02","-2.0","percent","sale","tax_group_withh_2","base","invoice","-Taxable 2% Withholding on Sales","","tax_excluded"
"","","","","","","","","tax","invoice","-2% Withheld on Sales","l10n_et3006",""
"","","","","","","","","base","refund","+Taxable 2% Withholding on Sales","",""
"","","","","","","","","tax","refund","+2% Withheld on Sales","l10n_et3006",""
"id_tax13","35% WH","Withholding 35% rated sales","tax13","-35.0","percent","sale","tax_group_withh_35","base","invoice","+Taxable 35% Withholding on Sales","","tax_excluded"
"","","","","","","","","tax","invoice","+35% Withheld on Sales","l10n_et3006",""
"","","","","","","","","base","refund","-Taxable 35% Withholding on Sales","",""
"","","","","","","","","tax","refund","-35% Withheld on Sales","l10n_et3006",""
"id_tax14","15% WH","Withholding VAT 15% rated sales","tax14","-15.0","percent","sale","tax_group_withh_15","base","invoice","+Taxable VAT Withholding on Sales","","tax_excluded"
"","","","","","","","","tax","invoice","+VAT Withheld on Sales","l10n_et3006",""
"","","","","","","","","base","refund","-Taxable VAT Withholding on Sales","",""
"","","","","","","","","tax","refund","-VAT Withheld on Sales","l10n_et3006",""
"id_tax08","15%","VAT 15% rated purchases","tax08","15.0","percent","purchase","tax_group_vat_15","base","invoice","+Taxable Purchase VAT Rated 15%","",""
"","","","","","","","","tax","invoice","+Purchase VAT Rated 15%","l10n_et2212",""
"","","","","","","","","base","refund","-Taxable Purchase VAT Rated 15%","",""
"","","","","","","","","tax","refund","-Purchase VAT Rated 15%","l10n_et2212",""
"id_tax07","0%","VAT 0% rated purchases","tax07","0.0","percent","purchase","tax_group_vat_0","base","invoice","+Taxable Purchase VAT Rated 0%","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Taxable Purchase VAT Rated 0%","",""
"","","","","","","","","tax","refund","","",""
"id_tax10","0% EXEMPT","VAT Exempt rated purchases","tax10","0.0","percent","purchase","tax_group_vat_0","base","invoice","+Taxable Purchase VAT Exempt","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Taxable Purchase VAT Exempt","",""
"","","","","","","","","tax","refund","","",""
"id_tax09","0% Out","VAT Out of Scope rated purchases","tax09","0.0","percent","purchase","tax_group_vat_0","base","invoice","+Taxable Purchase VAT Out of Scope","",""
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Taxable Purchase VAT Out of Scope","",""
"","","","","","","","","tax","refund","","",""
"id_tax05","2% WH","Withholding 2% rated purchases","tax05","-2.0","percent","purchase","tax_group_withh_2","base","invoice","+Taxable 2% Withholding on Purchases","",""
"","","","","","","","","tax","invoice","+2% Withholding on Purchases","l10n_et2213",""
"","","","","","","","","base","refund","-Taxable 2% Withholding on Purchases","",""
"","","","","","","","","tax","refund","-2% Withholding on Purchases","l10n_et2213",""
"id_tax12","35% WH","Withholding 35% rated purchases","tax12","-35.0","percent","purchase","tax_group_withh_35","base","invoice","+Taxable 35% Withholding on Purchases","",""
"","","","","","","","","tax","invoice","+35% Withholding on Purchases","l10n_et2213",""
"","","","","","","","","base","refund","-Taxable 35% Withholding on Purchases","",""
"","","","","","","","","tax","refund","-35% Withholding on Purchases","l10n_et2213",""

```

## File: data\template\account.tax.group-et.csv

```csv
"id","name","country_id"
"tax_group_vat_0","VAT 0%","base.et"
"tax_group_vat_15","VAT 15%","base.et"
"tax_group_withh_2","Withholding 2%","base.et"
"tax_group_withh_15","Withholding 15%","base.et"
"tax_group_withh_35","Withholding 35%","base.et"

```

## File: models\template_et.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('et')
    def _get_et_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'l10n_et2211',
            'property_account_payable_id': 'l10n_et3002',
            'property_account_expense_categ_id': 'l10n_et2301',
            'property_account_income_categ_id': 'l10n_et1100',
        }

    @template('et', 'res.company')
    def _get_et_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.et',
                'bank_account_code_prefix': '211',
                'cash_account_code_prefix': '211',
                'transfer_account_code_prefix': '212',
                'account_default_pos_receivable_account_id': 'l10n_et2215',
                'income_currency_exchange_account_id': 'l10n_et6435',
                'expense_currency_exchange_account_id': 'l10n_et6436',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_et626001',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_et120001',
                'account_sale_tax_id': 'id_tax03',
                'account_purchase_tax_id': 'id_tax08',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_et

```

