# Odoo Module: l10n_rw

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Rwanda - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['rw'],
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.0',
    'depends': [
        'account',
    ],
    'description': """
    Rwandan localisation containing:
    - COA
    - Taxes
    - Tax report
    - Fiscal position
    """,
    'data': [
        'data/l10n_rw_chart_data.xml',
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
        <field name="country_id" ref="base.rw"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_base" model="account.report.column">
                <field name="name">base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="tax_report_tax" model="account.report.column">
                <field name="name">tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_value_supplies_during_month" model="account.report.line">
                <field name="name">5- Value of Supplies During the month (VAT Exclusive)</field>
                <field name="code">rw_value_supplies</field>
                <field name="expression_ids">
                    <record id="tax_report_value_supplies_during_month_expr" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">5.base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_exempted_sales" model="account.report.line">
                <field name="name">10- Exempted Sales</field>
                <field name="code">rw_exempted_sales</field>
                <field name="expression_ids">
                    <record id="tax_report_exempted_sales_expr" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">10.base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_zero_rated_sales" model="account.report.line">
                <field name="name">15- Zero Rated Sales</field>
                <field name="code">rw_zero_rated_sales</field>
                <field name="expression_ids">
                    <record id="tax_report_zero_rated_sales_expr" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">15.base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_export" model="account.report.line">
                <field name="name">20- Exports</field>
                <field name="code">rw_export</field>
                <field name="expression_ids">
                    <record id="tax_report_export_expr" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">20.base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_total_not_taxable" model="account.report.line">
                <field name="name">25- Total Not Taxable (Line 10 + 15 + 20)</field>
                <field name="code">rw_not_taxable</field>
                <field name="expression_ids">
                    <record id="tax_report_total_not_taxable_expr" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_exempted_sales.base + rw_zero_rated_sales.base + rw_export.base </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_taxable_sales_subject_vat" model="account.report.line">
                <field name="name">30- Taxable Sales Subject to VAT ( Line 5 - Line 25)</field>
                <field name="code">rw_taxable_sales_subject_vat</field>
                <field name="expression_ids">
                    <record id="tax_report_taxable_sales_subject_vat_expr" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_value_supplies.base - rw_not_taxable.base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_taxable_sales" model="account.report.line">
                <field name="name">35- VAT on Taxable Sales (18% of Line 30)</field>
                <field name="code">rw_vat_taxable_sales</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_taxable_sales_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">35.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_reverse_charge" model="account.report.line">
                <field name="name">40- VAT Reverse Charge</field>
                <field name="code">rw_reverse_charge</field>
                <field name="expression_ids">
                    <record id="tax_report_reverse_charge_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">40.base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_payable" model="account.report.line">
                <field name="name">45- VAT Payable (Line 35 + Line 40)</field>
                <field name="code">rw_vat_payable</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_payable_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_vat_taxable_sales.tax + rw_reverse_charge.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_paid_on_imports" model="account.report.line">
                <field name="name">50- VAT paid on imports</field>
                <field name="code">rw_vat_paid_import</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_paid_on_imports_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">50.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_paid_on_local_purchase" model="account.report.line">
                <field name="name">55- VAT paid on Local Purchase</field>
                <field name="code">rw_vat_paid_purchase</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_paid_on_local_purchase_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">55.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_paid_on_input" model="account.report.line">
                <field name="name">60- VAT paid on Input (Line 50 + Line 55)</field>
                <field name="code">rw_vat_paid_on_input</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_paid_on_input_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_vat_paid_import.tax + rw_vat_paid_purchase.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_reverse_charge_deductible" model="account.report.line">
                <field name="name">65- VAT Reverse Charge deductible</field>
                <field name="code">rw_vat_reverse_charge_deductible</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_reverse_charge_deductible_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">65.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_refund" model="account.report.line">
                <field name="name">70- VAT Payable/Credit Refundable [(Line 45 - (Line 60 + Line 65)]</field>
                <field name="code">rw_vat_refund</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_refund_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_vat_payable.tax - (rw_vat_paid_on_input.tax + rw_vat_reverse_charge_deductible.tax)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_carry_over" model="account.report.line">
                <field name="name">75- Credit carried over from previous month(s) (Not already claimed)</field>
                <field name="code">rw_carry_over</field>
                <field name="expression_ids">
                    <record id="tax_report_carry_over_tag_expr" model="account.report.expression">
                        <field name="label">tag</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">75.tax</field>
                    </record>
                    <record id="tax_report_carry_over_applied_carryover" model="account.report.expression">
                        <field name="label">_applied_carryover_balance</field>
                        <field name="engine">external</field>
                        <field name="formula">most_recent</field>
                        <field name="date_scope">previous_tax_period</field>
                    </record>
                    <record id="tax_report_22_balance" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_carry_over.tag + rw_carry_over._applied_carryover_balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_withholding_retained" model="account.report.line">
                <field name="name">80- VAT Withholding retained by MINECOFIN (not refunded)</field>
                <field name="code">rw_withholding_retained</field>
                <field name="expression_ids">
                    <record id="tax_report_withholding_retained_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_due_credit" model="account.report.line">
                <field name="name">85- VAT Due / Credit Refundable (Line 70 - Line 75)</field>
                <field name="code">rw_vat_due_credit</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_due_credit_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_vat_refund.tax - rw_carry_over.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_refund_claim" model="account.report.line">
                <field name="name">90- VAT Refund Claim</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_refund_claim_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_vat_refund.tax - rw_carry_over.tax</field>
                        <field name="subformula">if_below(RWF(0))</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat_due" model="account.report.line">
                <field name="name">95- VAT Due</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_due_expr" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">rw_vat_refund.tax - rw_carry_over.tax</field>
                        <field name="subformula">if_above(RWF(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_rw_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_rw_statements_menu" name="Rwanda" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\template\account.account-rw.csv

```csv
"id","code","name","account_type","reconcile"
"rw_102","102","Cash Equivalents","asset_cash","False"
"rw_104","104","Internal Transfers of Funds","asset_current","False"
"rw_106","106","Other current assets","asset_current","False"
"rw_107","107","Tax paid","asset_current","False"
"rw_108","108","Tax receivable","asset_current","False"
"rw_110","110","Property, Plant And Equipment","asset_fixed","False"
"rw_111","111","Land And Land Improvements","asset_fixed","False"
"rw_112","112","Buildings, Structures And Improvements","asset_fixed","False"
"rw_113","113","Machinery And Equipment","asset_fixed","False"
"rw_114","114","Furniture And Fixtures","asset_fixed","False"
"rw_115","115","Right Of Use Assets (Classified As PP&E)","asset_fixed","False"
"rw_116","116","Additional Property, Plant And Equipment","asset_fixed","False"
"rw_117","117","Construction In Progress","asset_non_current","False"
"rw_121","121","Investment Property","asset_non_current","False"
"rw_122","122","Investment Property Under Construction Or Development","asset_non_current","False"
"rw_130","130","Goodwill","asset_non_current","False"
"rw_135","135","Advances for Capital Assets","asset_non_current","False"
"rw_140","140","Intangible Assets (Excluding Goodwill)","asset_non_current","False"
"rw_141","141","Intellectual Property","asset_fixed","False"
"rw_142","142","Computer Software","asset_fixed","False"
"rw_143","143","Trade And Distribution Assets","asset_non_current","False"
"rw_144","144","Contracts And Rights","asset_non_current","False"
"rw_145","145","Right To Use Assets (Classified By Type)","asset_non_current","False"
"rw_146","146","Other Intangible Assets","asset_non_current","False"
"rw_147","147","Acquisition In Progress","asset_non_current","False"
"rw_148","148","Deferred tax assets","asset_non_current","False"
"rw_149","149","Available for sale investments","asset_non_current","False"
"rw_150","150","Financial Assets (Investments)","asset_non_current","False"
"rw_151","151","Non-Derivative Financial Assets","asset_receivable","True"
"rw_152","152","Derivative Financial Assets","asset_receivable","True"
"rw_153","153","Restricted Cash And Financial Assets","asset_cash","False"
"rw_154","154","Additional Financial Assets And Investments","asset_non_current","False"
"rw_155","155","Trade receivables (PoS)","asset_receivable","True"
"rw_156","156","Investments in associate","asset_receivable","True"
"rw_160","160","Agricultural (Biological) Assets","asset_current","False"
"rw_161","161","Bearer Plants","asset_current","False"
"rw_162","162","Animals","asset_current","False"
"rw_163","163","Other Agricultural Assets","asset_current","False"
"rw_170","170","Inventory","asset_current","False"
"rw_171","171","Merchandise","asset_current","False"
"rw_172","172","Raw Material, Parts And Supplies","asset_current","False"
"rw_173","173","Work In Process","asset_current","False"
"rw_174","174","Finished Goods","asset_current","False"
"rw_175","175","Other Inventory","asset_current","False"
"rw_176","176","Income tax assets","asset_current","False"
"rw_180","180","Accruals And Additional Assets","asset_receivable","True"
"rw_181","181","Prepaid Expense","asset_current","False"
"rw_182","182","Accrued Income","asset_current","False"
"rw_183","183","Additional Assets","asset_receivable","True"
"rw_184","184","Investments and financial receivables","asset_receivable","True"
"rw_190","190","Receivables And Contracts","asset_receivable","True"
"rw_191","191","Accounts, Notes And Loans Receivable","asset_receivable","True"
"rw_192","192","Contracts","asset_current","False"
"rw_193","193","Nontrade And Other Receivables","asset_receivable","True"
"rw_210","210","Owners Equity (Attributable To Owners Of Parent)","equity","False"
"rw_211","211","Equity At Par (Issued Capital)","equity","False"
"rw_212","212","Retained Earnings","equity","False"
"rw_213","213","Additional Paid-In Capital","equity","False"
"rw_220","220","Treasury Stock","equity","False"
"rw_221","221","Treasury Stock Common","equity","False"
"rw_222","222","Treasury Stock Preferred","equity","False"
"rw_230","230","Accumulated OCI","equity","False"
"rw_231","231","Exchange Differences On Translation","equity","False"
"rw_232","232","Remeasurements Cash Flow Hedges","equity","False"
"rw_233","233","Remeasurements Available-For-Sale Financial Assets","equity","False"
"rw_234","234","Remeasurement Of Defined Benefit Plans","equity","False"
"rw_235","235","Revaluation Surplus (IFRS Only)","equity","False"
"rw_236","236","Remeasurements Investments In Equity Instruments (IFRS only)","equity","False"
"rw_240","240","Other Equity Items","equity","False"
"rw_241","241","ESOP Related Items","equity","False"
"rw_242","242","Subscribed Stock Receivables","equity","False"
"rw_250","250","Miscellaneous Equity","equity","False"
"rw_260","260","Non-controlling (Minority) Interest","equity","False"
"rw_270","270","Share capital","equity","False"
"rw_307","307","Tax received","liability_current","False"
"rw_308","308","Tax Payable","liability_current","False"
"rw_311","311","Trade Payables","liability_payable","True"
"rw_312","312","Dividends Payable","liability_payable","True"
"rw_313","313","Interest Payable","liability_payable","True"
"rw_314","314","Other Payables","liability_payable","True"
"rw_320","320","Provisions (Contingencies)","liability_current","False"
"rw_321","321","Customer Related Provisions","liability_current","False"
"rw_322","322","Ligation And Regulatory Provisions","liability_current","False"
"rw_323","323","Additional Provisions","liability_current","False"
"rw_330","330","Financial Liabilities","liability_current","False"
"rw_331","331","Notes Payable","liability_payable","True"
"rw_332","332","Loans Payable","liability_payable","True"
"rw_333","333","Bonds (Debentures)","liability_current","False"
"rw_334","334","Other Debts And Borrowings","liability_current","False"
"rw_335","335","Lease Obligations","liability_current","False"
"rw_336","336","Derivative Financial Liabilities","liability_current","False"
"rw_340","340","Accruals And Other Liabilities","liability_current","False"
"rw_341","341","Accrued Expenses","liability_current","False"
"rw_342","342","Deferred Income (Unearned Revenue)","liability_current","False"
"rw_343","343","Accrued Taxes (Other Than Payroll)","liability_current","False"
"rw_344","344","Other Liabilities","liability_current","False"
"rw_350","350","Banks overdrafts and short-term borrowings","liability_current","False"
"rw_360","360","Interest-bearing loans and short term borrowings","liability_current","False"
"rw_370","370","Income tax liabilities","liability_current","False"
"rw_380","380","Interest-bearing loans and short term borrowings","liability_non_current","False"
"rw_390","390","Employee benefits liabilities","liability_non_current","False"
"rw_395","395","Provisions","liability_non_current","False"
"rw_396","396","Deferred tax liabilities","liability_non_current","False"
"rw_400","400","Revenue","income","False"
"rw_410","410","Recognized Point Of Time","income","False"
"rw_411","411","Goods","income","False"
"rw_412","412","Services","income","False"
"rw_420","420","Recognized Over Time","income","False"
"rw_421","421","Products","income","False"
"rw_422","422","Services","income","False"
"rw_430","430","Adjustments","income","False"
"rw_431","431","Variable Consideration","income","False"
"rw_432","432","Consideration Paid (Payable) To Customers","income","False"
"rw_433","433","Other Adjustments","income","False"
"rw_510","510","Expenses Classified By Nature","expense","False"
"rw_511","511","Material And Merchandise","expense","False"
"rw_512","512","Employee Benefits","expense","False"
"rw_513","513","Services","expense","False"
"rw_514","514","Rent, Depreciation, Amortization And Depletion","expense","False"
"rw_515","515","Increase (Decrease) In Inventories Of Finished Goods And Work In Progress (IFRS only)","expense","False"
"rw_516","516","Other Work Performed By Entity And Capitalized (IFRS only)","expense","False"
"rw_520","520","Expenses Classified By Function","expense","False"
"rw_521","521","Cost Of Sales","expense","False"
"rw_522","522","Selling, General And Administrative","expense","False"
"rw_611","611","Other Revenue","income","False"
"rw_612","612","Other Expenses","expense","False"
"rw_613","613","Change in inventories","income","False"
"rw_614","614","Change in fair value of investment property","income","False"
"rw_615","615","Depreciation, amortisation and impairment of non-financial assets","income","False"
"rw_616","616","Impairment losses of financial assets","expense","False"
"rw_6211","6211","Foreign Currency Transaction Gain","income","False"
"rw_6212","6212","Foreign Currency Transaction Loss","expense","False"
"rw_6221","6221","Gain On Investments","income","False"
"rw_6222","6222","Loss On Investments","expense","False"
"rw_6231","6231","Gain On Derivatives","income","False"
"rw_6232","6232","Loss On Derivatives","expense","False"
"rw_6241","6241","Gain On Disposal Of Assets","income","False"
"rw_6242","6242","Loss On Disposal Of Assets","expense","False"
"rw_6251","6251","Debt Related Gain","income","False"
"rw_6252","6252","Debt Related Loss","expense","False"
"rw_626","626","Impairment Loss","expense","False"
"rw_627","627","Impairment Loss (Reversal) Financial Assets (IFRS Only)","expense","False"
"rw_6281","6281","Other Gains","income","False"
"rw_6282","6282","Other Losses","expense","False"
"rw_630","630","Taxes (Other Than Income And Payroll) And Fees","expense","False"
"rw_631","631","Real Estate Taxes And Insurance","expense","False"
"rw_632","632","Highway (Road) Taxes And Tolls","expense","False"
"rw_633","633","Direct Tax And License Fees","expense","False"
"rw_634","634","Excise And Sales Taxes","expense","False"
"rw_635","635","Customs Fees And Duties (Not Classified As Sales Or Excise)","expense","False"
"rw_636","636","Non-Deductible VAT (GST)","expense","False"
"rw_637","637","General Insurance Expense","expense","False"
"rw_638","638","Administrative Fees (Revenue Stamps)","expense","False"
"rw_639","639","Fines And Penalties","expense","False"
"rw_640","640","Income Tax Expense (Benefit)","expense","False"
"rw_650","650","Miscellaneous Taxes","expense","False"
"rw_660","660","Other Taxes And Fees","expense","False"
"rw_671","671","Foreign Exchange Gain","income","False"
"rw_672","672","Foreign Exchange Loss","expense","False"
"rw_680","680","Share of profit from equity accounted investments","income","False"
"rw_681","681","Finance costs","expense","False"
"rw_682","682","Finance income","income","False"
"rw_683","683","Other financial items","expense","False"
"rw_690","690","Loss for the year from discontinued operations","expense","False"

```

## File: data\template\account.fiscal.position-rw.csv

```csv
"id","name","sequence","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_rw_national","National B2B","2","1","1","base.rw","","",""
"fiscal_position_rw_international","International","3","1","","","","VAT_S_IN_RW_18","VAT_S_EXPORT"
"","","","","","","","VAT_P_IN_RW_18","VAT_P_IMPORT"

```

## File: data\template\account.tax-rw.csv

```csv
"id","sequence","description","invoice_label","name","price_include","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id"
"VAT_S_IN_RW_18","1","Local 18% sales","Local 18% sales","18%","False","18.0","percent","sale","tax_group_vat_18","100","base","invoice","+5.base",""
"","","","","","","","","","","100","tax","invoice","+35.tax","rw_307"
"","","","","","","","","","","100","base","refund","+5.base",""
"","","","","","","","","","","100","tax","refund","+35.tax","rw_307"
"VAT_S_exempt_O","2","Exempt","Exempt","0% EXEMPT","False","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+10.base",""
"","","","","","","","","","","100","tax","invoice","",""
"","","","","","","","","","","100","base","refund","+10.base",""
"","","","","","","","","","","100","tax","refund","",""
"VAT_S_IN_O","3","Local 0% sales","Local 0% sales","0%","False","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+15.base",""
"","","","","","","","","","","100","tax","invoice","",""
"","","","","","","","","","","100","base","refund","+15.base",""
"","","","","","","","","","","100","tax","refund","",""
"VAT_S_EXPORT","4","Export 0%","Export 0%","0% EX","False","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+20.base",""
"","","","","","","","","","","100","tax","invoice","",""
"","","","","","","","","","","100","base","refund","+20.base",""
"","","","","","","","","","","100","tax","refund","",""
"VAT_S_reverse_O","5","Reverse charge 0%","Reverse charge 0%","0% R C","False","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+40.base",""
"","","","","","","","","","","100","tax","invoice","",""
"","","","","","","","","","","100","base","refund","+40.base",""
"","","","","","","","","","","100","tax","refund","",""
"VAT_P_IMPORT","6","Import 18%","Import 18%","18% EX","False","18.0","percent","purchase","tax_group_vat_0","100","base","invoice","",""
"","","","","","","","","","","100","tax","invoice","+50.tax",""
"","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","100","tax","refund","+50.tax",""
"VAT_P_IN_RW_0","7","Local 0% purchases","Local 0% purchases","0%","False","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","",""
"","","","","","","","","","","100","tax","invoice","",""
"","","","","","","",,"","","100","base","refund","",""
"","","","","","","","","","","100","tax","refund","",""
"VAT_P_IN_RW_18","8","Local 0% purchases","Local 0% purchases","18%","False","18.0","percent","purchase","tax_group_vat_18","100","base","invoice","",""
"","","","","","","",,"","","100","tax","invoice","+55.tax","rw_107"
"","","","","","","",,"","","100","base","refund","",""
"","","","","","","","","","","100","tax","refund","+55.tax","rw_107"
"VAT_P_reverse_18","9","Reverse charge 18%","Reverse charge 0%","18% R C","False","18.0","percent","purchase","tax_group_vat_0","100","base","invoice","",""
"","","","","","","","","","","100","tax","invoice","+65.tax","rw_107"
"","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","100","tax","refund","+65.tax","rw_107"

```

## File: data\template\account.tax.group-rw.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"tax_group_vat_0","VAT 0%","base.rw","rw_108","rw_308"
"tax_group_vat_18","VAT 18%","base.rw","rw_108","rw_308"

```

## File: models\template_rw.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('rw')
    def _get_rw_template_data(self):
        return {
            'code_digits': '4',
            'property_account_receivable_id': 'rw_190',
            'property_account_payable_id': 'rw_311',
            'property_account_expense_categ_id': 'rw_510',
            'property_account_income_categ_id': 'rw_400',
        }

    @template('rw', 'res.company')
    def _get_rw_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.rw',
                'cash_account_code_prefix': '101',
                'bank_account_code_prefix': '103',
                'transfer_account_code_prefix': '105',
                'account_default_pos_receivable_account_id': 'rw_155',
                'income_currency_exchange_account_id': 'rw_671',
                'expense_currency_exchange_account_id': 'rw_672',
                'deferred_revenue_account_id': 'rw_181',
                'deferred_expense_account_id': 'rw_342',
                'account_sale_tax_id': 'VAT_S_IN_RW_18',
                'account_purchase_tax_id': 'VAT_P_IN_RW_18',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_rw

```

