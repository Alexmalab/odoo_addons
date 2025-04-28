# Odoo Module: l10n_au

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
    'name': 'Australia - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/australia.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['au'],
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Australian Accounting Module
============================

Australian accounting basic charts and localizations.

Also:
    - activates a number of regional currencies.
    - sets up Australian taxes.
    """,
    'depends': ['account'],
    'data': [
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/res_currency_data.xml',
        'views/menuitems.xml',
        'views/report_invoice.xml',
        'views/res_company_views.xml',
        'views/res_partner_bank_views.xml',
        'views/report_payment_receipt_templates.xml',
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
        <field name="name">BAS Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.au"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_gstrpt_sale_total" model="account.report.line">
                <field name="name">GST amounts you owe the Tax Office from sales</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_gstrpt_g1" model="account.report.line">
                        <field name="name">G1: Total Sales (including any GST)</field>
                        <field name="code">G1</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_gstrpt_g1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">G1</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_gstrpt_g2" model="account.report.line">
                                <field name="name">G2: Export sales</field>
                                <field name="code">G2</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_gstrpt_g3" model="account.report.line">
                                <field name="name">G3: Other GST-free sales</field>
                                <field name="code">G3</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_gstrpt_g4" model="account.report.line">
                                <field name="name">G4: Input taxed sales</field>
                                <field name="code">G4</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G4</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_gstrpt_g5" model="account.report.line">
                        <field name="name">G5: G2 + G3 + G4</field>
                        <field name="code">G5</field>
                        <field name="aggregation_formula">G2.balance+G3.balance+G4.balance</field>
                    </record>
                    <record id="account_tax_report_gstrpt_g6" model="account.report.line">
                        <field name="name">G6: Total sales subject to GST (G1 minus G5)</field>
                        <field name="code">G6</field>
                        <field name="aggregation_formula">G1.balance-G5.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_gstrpt_g7" model="account.report.line">
                                <field name="name">G7: Adjustments (if applicable)</field>
                                <field name="code">G7</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G7</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_gstrpt_g8" model="account.report.line">
                        <field name="name">G8: Total sales subject to GST after adjustments (G6 + G7)</field>
                        <field name="code">G8</field>
                        <field name="aggregation_formula">G6.balance+G7.balance</field>
                    </record>
                    <record id="account_tax_report_gstrpt_g9" model="account.report.line">
                        <field name="name">G9: GST on sales (G8 divided by eleven)</field>
                        <field name="code">G9</field>
                        <field name="aggregation_formula">G8.balance/11</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_gstrpt_purchase_total" model="account.report.line">
                <field name="name">GST amounts the Tax Office owes you from purchases</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_gstrpt_g10" model="account.report.line">
                        <field name="name">G10: Capital purchases (including any GST)</field>
                        <field name="code">G10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_gstrpt_g10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">G10</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_gstrpt_g11" model="account.report.line">
                        <field name="name">G11: Non-capital purchases (including GST)</field>
                        <field name="code">G11</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_gstrpt_g11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">G11</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_gstrpt_g12" model="account.report.line">
                        <field name="name">G12: G10 + G11</field>
                        <field name="code">G12</field>
                        <field name="aggregation_formula">G10.balance+G11.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_gstrpt_g13" model="account.report.line">
                                <field name="name">G13: Purchases for making input taxed sales</field>
                                <field name="code">G13</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G13</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_gstrpt_g14" model="account.report.line">
                                <field name="name">G14: Purchases without GST in the price</field>
                                <field name="code">G14</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G14</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_gstrpt_g15" model="account.report.line">
                                <field name="name">G15: Estimated purchases for private use or not income tax deductible</field>
                                <field name="code">G15</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g15_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G15</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_gstrpt_g16" model="account.report.line">
                        <field name="name">G16: G13 + G14 + G15</field>
                        <field name="code">G16</field>
                        <field name="aggregation_formula">G13.balance+G14.balance+G15.balance</field>
                    </record>
                    <record id="account_tax_report_gstrpt_g17" model="account.report.line">
                        <field name="name">G17: Total purchases subject to GST (G12 minus G16) </field>
                        <field name="code">G17</field>
                        <field name="aggregation_formula">G12.balance-G16.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_gstrpt_g18" model="account.report.line">
                                <field name="name">G18: Adjustments (if applicable)</field>
                                <field name="code">G18</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_gstrpt_g18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">G18</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_gstrpt_g19" model="account.report.line">
                        <field name="name">G19: Total purchases subject to GST after adjustments (G17 + G18) </field>
                        <field name="code">G19</field>
                        <field name="aggregation_formula">G17.balance+G18.balance</field>
                    </record>
                    <record id="account_tax_report_gstrpt_g20a" model="account.report.line">
                        <field name="name">GST on purchases (G19 divided by eleven)</field>
                        <field name="code">GP</field>
                        <field name="aggregation_formula">G19.balance/11</field>
                    </record>
                    <record id="account_tax_report_gstrpt_gstonly" model="account.report.line">
                        <field name="name">GST only purchases</field>
                        <field name="code">ONLY</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_gstrpt_gstonly_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ONLY</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_gstrpt_g20b" model="account.report.line">
                        <field name="name">G20: GST on purchases</field>
                        <field name="code">G20</field>
                        <field name="aggregation_formula">GP.balance+ONLY.balance</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_gstrpt_summary" model="account.report.line">
                <field name="name">Summary</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_gstrpt_summary_1a" model="account.report.line">
                        <field name="name">1A: GST on sales</field>
                        <field name="code">1A</field>
                        <field name="aggregation_formula">G9.balance</field>
                    </record>
                    <record id="account_tax_report_gstrpt_summary_1b" model="account.report.line">
                        <field name="name">1B: GST on purchases</field>
                        <field name="code">1B</field>
                        <field name="aggregation_formula">G20.balance</field>
                    </record>
                    <record id="account_tax_report_gstrpt_summary_9" model="account.report.line">
                        <field name="name">9: Your payment</field>
                        <field name="aggregation_formula">1A.balance-1B.balance</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_gstrpt_comparison" model="account.report.line">
                <field name="name">Comparison</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_gstrpt_comparison_worksheet" model="account.report.line">
                        <field name="name">GST from worksheet (G20-G9)</field>
                        <field name="aggregation_formula">G20.balance-G9.balance</field>
                    </record>
                    <record id="account_tax_report_gstrpt_comparison_gl" model="account.report.line">
                        <field name="name">GST from General Ledger</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_gstrpt_comparison_gl_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">GST from General Ledger</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="service_tag" model="account.account.tag">
        <field name="name">Service</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.au"/>
    </record>
    <record id="tax_withheld_tag" model="account.account.tag">
        <field name="name">Tax Withheld</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.au"/>
    </record>
    </odoo>

```

## File: data\res_currency_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
	<data noupdate="1">
		<record id="base.AUD" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<!-- Common trading partner currencies, and especially for NZ, subsidiaries and branches -->

		<record id="base.NZD" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<record id="base.JPY" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<record id="base.INR" model="res.currency">
			<field name="active" eval="True"/>
		</record>

		<record id="base.GBP" model="res.currency">
			<field name="active" eval="True"/>
		</record>
	</data>
</odoo>

```

## File: data\template\account.account-au.csv

```csv
"id","code","name","account_type","reconcile"
"au_11110","11110","Bank","asset_cash","False"
"au_11130","11130","Petty Cash","asset_cash","False"
"au_11140","11140","Cash Drawer","asset_cash","False"
"au_11180","11180","Undeposited Funds","asset_current","True"
"au_11190","11190","Electronic Clearing","asset_current","True"
"au_11200","11200","Trade Debtors","asset_receivable","True"
"au_11201","11201","Trade Debtors (PoS)","asset_receivable","True"
"au_11210","11210","Less Prov'n for Doubtful Debts","asset_current","False"
"au_11310","11310","Raw Materials","asset_current","False"
"au_11320","11320","Finished Goods","asset_current","False"
"au_11330","11330","Trading Stock on Hand","asset_current","False"
"au_11340","11340","Goods Shipped not Invoiced","asset_current","False"
"au_11350","11350","Work In Process (Inventory)","asset_current","False"
"au_12100","12100","Deposits Paid","asset_prepayments","False"
"au_12200","12200","Prepaid Insurance","asset_current","False"
"au_13110","13110","Manufacturing Plant at Cost","asset_fixed","False"
"au_13120","13120","Manufac. Plant Accum Dep","asset_fixed","False"
"au_13130","13130","Manufacturing Equipment Cost","asset_fixed","False"
"au_13140","13140","Manufac. Equip Accum Dep","asset_fixed","False"
"au_13210","13210","Furniture & Fixtures at Cost","asset_fixed","False"
"au_13220","13220","Furniture & Fixtures Accum Dep","asset_fixed","False"
"au_13310","13310","Office Equip at Cost","asset_fixed","False"
"au_13320","13320","Office Equip Accum Dep","asset_fixed","False"
"au_13410","13410","Motor Vehicles at Cost","asset_fixed","False"
"au_13420","13420","Motor Vehicles Accum Dep","asset_fixed","False"
"au_21110","21110","Credit Card","liability_current","False"
"au_21200","21200","Trade Creditors","liability_payable","True"
"au_21210","21210","Goods Received not Billed","liability_current","False"
"au_21300","21300","Wages & Salaries","liability_current","True"
"au_21310","21310","GST Collected","liability_current","False"
"au_21320","21320","BAS Payments","liability_current","False"
"au_21330","21330","GST Paid","asset_current","False"
"au_21350","21350","Fuel Tax Credits Accrued","liability_current","False"
"au_21355","21355","WET Payable","liability_current","False"
"au_21360","21360","Import Duty Payable","liability_current","False"
"au_21370","21370","Voluntary Withholdings Payable","liability_current","False"
"au_21380","21380","ABN Withholdings Payable","liability_current","False"
"au_21400","21400","Superannuation","liability_current","False"
"au_21410","21410","Payroll Accruals Payable","liability_current","False"
"au_21420","21420","PAYG Withholding Payable","liability_current","False"
"au_21500","21500","Child Support","liability_current","False"
"au_21600","21600","Customer Deposits","liability_current","False"
"au_21700","21700","Other Current Liabilities","liability_current","False"
"au_22100","22100","Mortgages Payable","liability_non_current","False"
"au_22200","22200","Notes Payable","liability_non_current","False"
"au_22300","22300","Other Long Term Liabilities","liability_non_current","False"
"au_31100","31100","Capital Investment","equity","False"
"au_31200","31200","Capital Drawings","equity","False"
"au_38000","38000","Retained Earnings","equity","False"
"au_39000","39000","Current Year Earnings","equity_unaffected","False"
"au_39999","39999","Historical Balancing","equity","False"
"au_41110","41110","Sales Product #1","income","False"
"au_41120","41120","Sales Product #2","income","False"
"au_41130","41130","Sales Product #3","income","False"
"au_42000","42000","Wholesale Sales","income","False"
"au_43000","43000","Consignment Sales","income","False"
"au_44000","44000","Freight Income","income","False"
"au_45000","45000","Late Fees Collected","income_other","False"
"au_46000","46000","Miscellaneous Income","income_other","False"
"au_47000","47000","Fuel Tax Credits","income_other","False"
"au_51110","51110","Cost of Goods Sold #1","expense_direct_cost","False"
"au_51120","51120","Cost of Goods Sold # 2","expense_direct_cost","False"
"au_51130","51130","Cost of Goods Sold # 3","expense_direct_cost","False"
"au_52000","52000","Wholesale Cost of Sales","expense_direct_cost","False"
"au_53000","53000","Consignment Cost of Sales","expense_direct_cost","False"
"au_54000","54000","Wages for Production Labour","expense_direct_cost","False"
"au_55000","55000","Materials & Supplies","expense_direct_cost","False"
"au_56000","56000","Freight","expense_direct_cost","False"
"au_57000","57000","Other Costs","expense_direct_cost","False"
"au_61000","61000","Advertising","expense","False"
"au_61200","61200","Car & Truck Expenses","expense","False"
"au_61300","61300","Commissions Paid","expense","False"
"au_61500","61500","Depreciation Expense","expense_depreciation","False"
"au_61610","61610","Discounts Given","expense","False"
"au_61620","61620","Discounts Taken","expense","False"
"au_61630","61630","Exchange Rate Loss","expense","False"
"au_61640","61640","Exchange Rate Gain","income","False"
"au_61700","61700","Freight Paid","expense","False"
"au_61800","61800","Insurance","expense","False"
"au_61910","61910","Overdraft Interest","expense","False"
"au_61920","61920","Mortgage Interest","expense","False"
"au_61930","61930","Other Interest","expense","False"
"au_62000","62000","Late Fees Paid","expense","False"
"au_62110","62110","Machinery & Equipment","expense","False"
"au_62120","62120","Other Business Property","expense","False"
"au_62200","62200","Legal & Professional Services","expense","False"
"au_62300","62300","Office Expenses","expense","False"
"au_62410","62410","Staff Amenities","expense","False"
"au_62420","62420","Superannuation","expense","False"
"au_62430","62430","Wages & Salaries","expense","False"
"au_62440","62440","Workers' Compensation","expense","False"
"au_62450","62450","Other Employer Expenses","expense","False"
"au_62460","62460","Child Support","expense","False"
"au_62500","62500","Repairs","expense","False"
"au_62550","62550","Shrinkage/Spoilage","expense","False"
"au_62600","62600","Supplies","expense","False"
"au_62700","62700","Taxes","expense","False"
"au_62800","62800","Telephone","expense","False"
"au_62910","62910","Gas","expense","False"
"au_62920","62920","Electricity","expense","False"
"au_62930","62930","Water","expense","False"
"au_63110","63110","Travel","expense","False"
"au_63120","63120","Meals & Entertainment","expense","False"
"au_81000","81000","Interest Income","income_other","False"
"au_91000","91000","Interest Expense","expense","False"
"au_92000","92000","Income Tax Expense","expense","False"

```

## File: data\template\account.fiscal.position-au.csv

```csv
"id","name","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_os_partner","OS Partner","au_tax_sale_10","au_tax_sale_0"
"","","au_tax_sale_inc_10","au_tax_sale_0"
"","","au_tax_purchase_10_service","au_tax_purchase_0_service"
"","","au_tax_purchase_inc_10_service","au_tax_purchase_0_service"
"","","au_tax_purchase_capital_service","au_tax_purchase_0_service"
"fiscal_position_tpar_partner","TPAR","au_tax_purchase_10_service","au_tax_purchase_10_service_tpar"
"","","au_tax_purchase_inc_10_service","au_tax_purchase_inc_10_service_tpar"
"","","au_tax_purchase_capital_service","au_tax_purchase_capital_service_tpar"
"","","au_tax_purchase_0_service","au_tax_purchase_0_service_tpar"
"","","au_tax_purchase_input_service","au_tax_purchase_input_service_tpar"
"","","au_tax_purchase_private_service","au_tax_purchase_private_service_tpar"
"","","au_tax_purchase_gst_only_service","au_tax_purchase_gst_only_service_tpar"
"","","au_tax_purchase_adj_service","au_tax_purchase_adj_service_tpar"
"fiscal_position_tpar_partner_no_abn","TPAR without ABN","au_tax_purchase_10_service","au_tax_purchase_10_service_tpar_no_abn"
"","","au_tax_purchase_inc_10_service","au_tax_purchase_inc_10_service_tpar_no_abn"
"","","au_tax_purchase_capital_service","au_tax_purchase_capital_service_tpar_no_abn"
"","","au_tax_purchase_0_service","au_tax_purchase_0_service_tpar_no_abn"
"","","au_tax_purchase_input_service","au_tax_purchase_input_service_tpar_no_abn"
"","","au_tax_purchase_private_service","au_tax_purchase_private_service_tpar_no_abn"
"","","au_tax_purchase_gst_only_service","au_tax_purchase_gst_only_service_tpar_no_abn"
"","","au_tax_purchase_adj_service","au_tax_purchase_adj_service_tpar_no_abn"

```

## File: data\template\account.tax-au.csv

```csv
"id","name","sequence","description","invoice_label","type_tax_use","tax_scope","amount_type","amount","price_include","tax_group_id","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","active"
"au_tax_witheld","47% WH","1000","Tax Withheld for Partners without ABN","Tax Withheld for Partners without ABN","purchase","service","percent","-47.0","False","tax_group_withheld","","base","invoice","","","True"
"","","","","","","","","","","","","tax","invoice","au_21380","l10n_au.tax_withheld_tag",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","au_21380","l10n_au.tax_withheld_tag",""
"au_tax_sale_10","10%","1","GST Sales","GST Sales","sale","","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G1","True"
"","","","","","","","","","","","","tax","invoice","au_21310","+G1||-GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","-G1",""
"","","","","","","","","","","","","tax","refund","au_21310","-G1||+GST from General Ledger",""
"au_tax_sale_inc_10","10% INC","2","GST Inclusive Sales","GST Inclusive Sales","sale","","percent","10.0","True","tax_group_gst_10","","base","invoice","","+G1","True"
"","","","","","","","","","","","","tax","invoice","au_21310","+G1||-GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","-G1",""
"","","","","","","","","","","","","tax","refund","au_21310","-G1||+GST from General Ledger",""
"au_tax_sale_0","0% EX","3","Zero Rated (Export) Sales","Zero Rated (Export) Sales","sale","","percent","0.0","False","tax_group_gst_0","","base","invoice","","+G1||+G2","True"
"","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","base","refund","","-G1||-G2",""
"","","","","","","","","","","","","tax","refund","","",""
"au_tax_sale_exempt","0% EXEMPT","4","Exempt Sales","Exempt Sales","sale","","percent","0.0","False","tax_group_gst_0","","base","invoice","","+G1||+G3","True"
"","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","base","refund","","-G1||-G3",""
"","","","","","","","","","","","","tax","refund","","",""
"au_tax_sale_input","0% I","5","Input Taxed Sales","Input Taxed Sales","sale","","percent","0.0","False","tax_group_gst_0","","base","invoice","","+G1||+G4","True"
"","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","base","refund","","-G1||-G4",""
"","","","","","","","","","","","","tax","refund","","",""
"au_tax_sale_adj","10% Adj","6","Tax Adjustments (Sales)","Tax Adjustments (Sales)","sale","","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G7","True"
"","","","","","","","","","","","","tax","invoice","au_21310","+G7||-GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","-G7",""
"","","","","","","","","","","","","tax","refund","au_21310","-G7||+GST from General Ledger",""
"au_tax_purchase_10_service","10%","1","GST Purchases","GST Purchases","purchase","","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G11","True"
"","","","","","","","","","","","","tax","invoice","au_21330","+G11||+GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","-G11",""
"","","","","","","","","","","","","tax","refund","au_21330","-G11||-GST from General Ledger",""
"au_tax_purchase_10_service_tpar","10% TPAR","1","GST Purchases","GST Purchases","purchase","service","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G11","False"
"","","","","","","","","","","","","tax","invoice","au_21330","+G11||+GST from General Ledger||Service",""
"","","","","","","","","","","","","base","refund","","-G11",""
"","","","","","","","","","","","","tax","refund","au_21330","-G11||-GST from General Ledger||Service",""
"au_tax_purchase_10_service_tpar_no_abn","10% TPAR NO ABN","1","GST Purchases","GST Purchases","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_10_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_inc_10_service","10% INC","2","GST Inclusive Purchases","GST Inclusive Purchases","purchase","","percent","10.0","True","tax_group_gst_10","","base","invoice","","+G11","True"
"","","","","","","","","","","","","tax","invoice","au_21330","+G11||+GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","-G11",""
"","","","","","","","","","","","","tax","refund","au_21330","-G11||-GST from General Ledger",""
"au_tax_purchase_inc_10_service_tpar","10% INC TPAR","2","GST Inclusive Purchases","GST Inclusive Purchases","purchase","service","percent","10.0","True","tax_group_gst_10","","base","invoice","","+G11","False"
"","","","","","","","","","","","","tax","invoice","au_21330","+G11||+GST from General Ledger||Service",""
"","","","","","","","","","","","","base","refund","","-G11",""
"","","","","","","","","","","","","tax","refund","au_21330","-G11||-GST from General Ledger||Service",""
"au_tax_purchase_inc_10_service_tpar_no_abn","10% INC TPAR N ABN","2","GST Inclusive Purchases","GST Inclusive Purchases","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_inc_10_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_capital_service","10% C","3","Capital Purchases","Capital Purchases","purchase","","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G10","True"
"","","","","","","","","","","","","tax","invoice","au_21330","+G10||+GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","-G10",""
"","","","","","","","","","","","","tax","refund","au_21330","-G10||-GST from General Ledger",""
"au_tax_purchase_capital_service_tpar","10% C TPAR","3","Capital Purchases","Capital Purchases","purchase","service","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G10","False"
"","","","","","","","","","","","","tax","invoice","au_21330","+G10||+GST from General Ledger||Service",""
"","","","","","","","","","","","","base","refund","","-G10",""
"","","","","","","","","","","","","tax","refund","au_21330","-G10||-GST from General Ledger||Service",""
"au_tax_purchase_capital_service_tpar_no_abn","10% C TPAR N ABN","3","Capital Purchases","Capital Purchases","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_capital_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_0_service","0% C","4","Zero Rated Purch","Zero Rated Purch","purchase","","percent","0.0","False","tax_group_gst_0","","base","invoice","","+G11||+G14","True"
"","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","base","refund","","-G11||-G14",""
"","","","","","","","","","","","","tax","refund","","",""
"au_tax_purchase_0_service_tpar","0% C TPAR","4","Zero Rated Purch TPAR","Zero Rated Purch TPAR","purchase","service","percent","0.0","False","tax_group_gst_0","","base","invoice","","+G11||+G14","False"
"","","","","","","","","","","","","tax","invoice","","l10n_au.service_tag",""
"","","","","","","","","","","","","base","refund","","-G11||-G14",""
"","","","","","","","","","","","","tax","refund","","l10n_au.service_tag",""
"au_tax_purchase_0_service_tpar_no_abn","0% C TPAR N ABN","4","Zero Rated Purch TPAR without ABN","Zero Rated Purch TPAR without ABN","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_0_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_taxable_import_service","100% T EX","5","Purchase (Taxable Imports) - Tax Paid Separately","Purchase (Taxable Imports) - Tax Paid Separately","purchase","","division","100.0","True","tax_group_gst_100000000","","base","invoice","","","True"
"","","","","","","","","","","","","tax","invoice","","+G11||+G14",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","-G11||-G14",""
"au_tax_purchase_taxable_import_service_tpar","100% T EX TPAR","5","Purchase (Taxable Imports) - Tax Paid Separately","Purchase (Taxable Imports) - Tax Paid Separately","purchase","service","division","100.0","True","tax_group_gst_100000000","","base","invoice","","","False"
"","","","","","","","","","","","","tax","invoice","","l10n_au.service_tag||+G11||+G14",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_au.service_tag||-G11||-G14",""
"au_tax_purchase_taxable_import_service_tpar_no_abn","100% T EX TPAR N ABN","5","Purchase (Taxable Imports) - Tax Paid Separately","Purchase (Taxable Imports) - Tax Paid Separately","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_taxable_import_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_input_service","10% I","6","Purchases for Input Taxed Sales","Purchases for Input Taxed Sales","purchase","","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G11||+G13","True"
"","","","","","","","","","","","","tax","invoice","","+G11||+G13",""
"","","","","","","","","","","","","base","refund","","-G11||-G13",""
"","","","","","","","","","","","","tax","refund","","-G11||-G13",""
"au_tax_purchase_input_service_tpar","10% I TPAR","6","Purchases for Input Taxed Sales","Purchases for Input Taxed Sales","purchase","service","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G11||+G13","False"
"","","","","","","","","","","","","tax","invoice","","+G11||+G13||Service",""
"","","","","","","","","","","","","base","refund","","-G11||-G13",""
"","","","","","","","","","","","","tax","refund","","-G11||-G13||Service",""
"au_tax_purchase_input_service_tpar_no_abn","100% I TPAR N ABN","6","Purchases for Input Taxed Sales","Purchases for Input Taxed Sales","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_input_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_private_service","10% P","7","Purchases for Private use or not deductible","Purchases for Private use or not deductible","purchase","","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G11||+G15","True"
"","","","","","","","","","","","","tax","invoice","","+G11||+G15",""
"","","","","","","","","","","","","base","refund","","-G11||-G15",""
"","","","","","","","","","","","","tax","refund","","-G11||-G15",""
"au_tax_purchase_private_service_tpar","10% P TPAR","7","Purchases for Private use or not deductible","Purchases for Private use or not deductible","purchase","service","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G11||+G15","False"
"","","","","","","","","","","","","tax","invoice","","+G11||+G15||Service",""
"","","","","","","","","","","","","base","refund","","-G11||-G15",""
"","","","","","","","","","","","","tax","refund","","-G11||-G15||Service",""
"au_tax_purchase_private_service_tpar_no_abn","10% P TPAR N ABN","7","Purchases for Private use or not deductible","Purchases for Private use or not deductible","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_private_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_gst_only_service","100% EX","8","GST Only on Imports","GST Only on Imports","purchase","","division","100.0","True","tax_group_gst_100000000","","base","invoice","","","True"
"","","","","","","","","","","","","tax","invoice","au_21330","+ONLY||+GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","au_21330","-ONLY||-GST from General Ledger",""
"au_tax_purchase_gst_only_service_tpar","100% EX TPAR","8","GST Only on Imports","GST Only on Imports","purchase","service","division","100.0","True","tax_group_gst_100000000","","base","invoice","","","False"
"","","","","","","","","","","","","tax","invoice","au_21330","+ONLY||+GST from General Ledger||Service",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","au_21330","-ONLY||-GST from General Ledger||Service",""
"au_tax_purchase_gst_only_service_tpar_no_abn","100% EX TPAR N ABN","8","GST Only on Imports","GST Only on Imports","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_gst_only_service_tpar,au_tax_witheld","","","","","False"
"au_tax_purchase_adj_service","10% Adj","9","Tax Adjustments (Purchases)","Tax Adjustments (Purchases)","purchase","","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G18","True"
"","","","","","","","","","","","","tax","invoice","au_21330","+G18||+GST from General Ledger",""
"","","","","","","","","","","","","base","refund","","-G18",""
"","","","","","","","","","","","","tax","refund","au_21330","-G18||-GST from General Ledger",""
"au_tax_purchase_adj_service_tpar","10% Adj TPAR","9","Tax Adjustments (Purchases)","Tax Adjustments (Purchases)","purchase","service","percent","10.0","False","tax_group_gst_10","","base","invoice","","+G18","False"
"","","","","","","","","","","","","tax","invoice","au_21330","+G18||+GST from General Ledger||Service",""
"","","","","","","","","","","","","base","refund","","-G18",""
"","","","","","","","","","","","","tax","refund","au_21330","-G18||-GST from General Ledger||Service",""
"au_tax_purchase_adj_service_tpar_no_abn","10% Adj TPAR N ABN","9","Tax Adjustments (Purchases)","Tax Adjustments (Purchases)","purchase","service","group","100.0","","tax_group_gst_10","au_tax_purchase_adj_service_tpar,au_tax_witheld","","","","","False"

```

## File: data\template\account.tax.group-au.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id"
"tax_group_gst_0","GST 0%","base.au","au_21320","au_21320"
"tax_group_gst_10","GST 10%","base.au","au_21320","au_21320"
"tax_group_gst_100000000","GST 100%","base.au","au_21320","au_21320"
"tax_group_withheld","Withheld","base.au","au_21320","au_21320"

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_name_invoice_report(self):
        if self.company_id.account_fiscal_country_id.code == 'AU':
            return 'l10n_au.report_invoice_document'
        return super()._get_name_invoice_report()

```

## File: models\res_partner_bank.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ResPartnerBank(models.Model):
    _inherit = "res.partner.bank"

    aba_bsb = fields.Char(string='BSB', help='Bank State Branch code - needed if payment is to be made using ABA files')

    @api.model
    def _get_supported_account_types(self):
        rslt = super(ResPartnerBank, self)._get_supported_account_types()
        rslt.append(('aba', _('ABA')))
        return rslt

    @api.constrains('aba_bsb')
    def _validate_aba_bsb(self):
        for record in self:
            if record.aba_bsb:
                test_bsb = re.sub('( |-)', '', record.aba_bsb)
                if len(test_bsb) != 6 or not test_bsb.isdigit():
                    raise ValidationError(_('BSB is not valid (expected format is "NNN-NNN"). Please rectify.'))

    @api.depends('acc_number')
    def _compute_acc_type(self):
        """ Criteria to be an ABA account:
            - Spaces, hypens, digits are valid.
            - Total length must be 9 or less.
            - Cannot be only spaces, zeros or hyphens (must have at least one digit in range 1-9)
        """
        super()._compute_acc_type()
        for rec in self:
            if rec.acc_type == 'bank' and re.match(r"^(?=.*[1-9])[ \-\d]{0,9}$", rec.acc_number or ''):
                rec.acc_type = 'aba'

```

## File: models\template_au.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('au')
    def _get_au_template_data(self):
        return {
            'code_digits': '5',
            'property_account_receivable_id': 'au_11200',
            'property_stock_account_production_cost_id': 'au_11350',
            'property_account_payable_id': 'au_21200',
            'property_account_expense_categ_id': 'au_51110',
            'property_account_income_categ_id': 'au_41110',
            'property_stock_account_input_categ_id': 'au_21210',
            'property_stock_account_output_categ_id': 'au_11340',
            'property_stock_valuation_account_id': 'au_11330',
        }

    @template('au', 'res.company')
    def _get_au_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.au',
                'bank_account_code_prefix': '1111',
                'cash_account_code_prefix': '1113',
                'transfer_account_code_prefix': '11170',
                'account_default_pos_receivable_account_id': 'au_11201',
                'income_currency_exchange_account_id': 'au_61640',
                'expense_currency_exchange_account_id': 'au_61630',
                'account_journal_early_pay_discount_loss_account_id': 'au_61610',
                'account_journal_early_pay_discount_gain_account_id': 'au_61620',
                'fiscalyear_last_month': '6',
                'fiscalyear_last_day': 30,
                # Changing the opening date to the first day of the fiscal year.
                # This way the opening entries will be set to the 30th of June.
                'account_opening_date': fields.Date.context_today(self).replace(month=7, day=1),
                'account_sale_tax_id': 'au_tax_sale_10',
                'account_purchase_tax_id': 'au_tax_purchase_10_service',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import template_au
from . import account_move
from . import res_partner_bank

```

## File: views\menuitems.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <menuitem id="account_reports_au_statements_menu" name="Australia" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//div[hasclass('page')]/h2" position="replace">
            <h2>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'">Tax Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'draft'">Draft Tax Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'cancel'">Cancelled Tax Invoice</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'posted'">Tax Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'draft'">Draft Tax Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'cancel'">Cancelled Tax Credit Note</span>
                <span t-elif="o.move_type == 'in_refund'">Tax Vendor Credit Note</span>
                <span t-elif="o.move_type == 'in_invoice'">Tax Vendor Bill</span>
                <span t-if="o.name != '/'" t-field="o.name"/>
            </h2>
        </xpath>
    </template>

    <!-- Workaround for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_au.report_invoice_document'"
               t-call="l10n_au.report_invoice_document"
               t-lang="lang"/>
        </xpath>
    </template>
</odoo>

```

## File: views\report_payment_receipt_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="report_payment_receipt_document" inherit_id="account.report_payment_receipt_document">
        <xpath expr="//strong[@id='payment_title']" position="replace">
            <!-- Customary in Australia to use the term "Remittance Advice" for payments to suppliers -->
            <t t-if="o.company_id.account_fiscal_country_id.code == 'AU' and o.partner_type == 'supplier'">
                <strong>Remittance Advice: <span t-field="o.name"/></strong>
            </t>
            <t t-else="">
                <strong>Payment Receipt: <span t-field="o.name"/></strong>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_company_form" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_au</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="attributes">
                <attribute name="invisible" add="country_code == 'AU'" separator=" or "/> 
            </xpath>
            <xpath expr="//field[@name='vat']" position="after">
                <field name="vat" string="ABN" invisible="country_code != 'AU'"/>
            </xpath>
            <xpath expr="//field[@name='company_registry']" position="attributes">
                <attribute name="invisible" add="country_code == 'AU'" separator=" or "/> 
            </xpath>
            <xpath expr="//field[@name='company_registry']" position="after">
                <field name="company_registry" string="ACN" invisible="country_code != 'AU'"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_bank_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_bank_form" model="ir.ui.view">
        <field name="name">aba.res.partner.bank.form</field>
        <field name="model">res.partner.bank</field>
        <field name='inherit_id' ref='base.view_partner_bank_form'/>
        <field name="arch" type="xml">
            <field name="acc_holder_name" position='after'>
                <field name='aba_bsb'/>
            </field>
        </field>
    </record>
</odoo>

```

