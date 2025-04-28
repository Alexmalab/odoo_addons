# Odoo Module: l10n_il

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
    'name': 'Israel - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['il'],
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest basic Israelian localisation necessary to run Odoo in Israel:
================================================================================

This module consists of:
 - Generic Israel Chart of Accounts
 - Taxes and tax report
 - Multiple Fiscal positions
 """,
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'depends': [
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_account_tag.xml',
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_tag_retention_tax_vendor_account" model="account.account.tag">
        <field name="name">Withholding Vendor Tax Account</field>
    </record>

	<record id="account_tag_dividend_account" model="account.account.tag">
        <field name="name">Dividend Account</field>
    </record>

    <record id="account_tag_retention_tax_dividend_account" model="account.account.tag">
        <field name="name">Withholding Dividend Tax Account</field>
    </record>

    <record id="account_tag_retention_tax_employees_account" model="account.account.tag">
        <field name="name">Withholding Employees Tax Account</field>
    </record>

    <record id="account_tag_retention_tax_customers_account" model="account.account.tag">
        <field name="name">Withholding Customers Tax Account</field>
    </record>

</odoo>
```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="vat_report" model="account.report">
        <field name="name">VAT Report (PCN874)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.il"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="vat_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_out_base_title" model="account.report.line">
                <field name="name">VAT SALES (BASE)</field>
                <field name="aggregation_formula">ILTAX_OUT_BASE.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_out_base" model="account.report.line">
                        <field name="name">VAT SALES (BASE)</field>
                        <field name="code">ILTAX_OUT_BASE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT SALES (BASE)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_vat_sales_tax" model="account.report.line">
                <field name="name">VAT SALES (TAX)</field>
                <field name="code">ILTAX_OUT_BALANCE</field>
                <field name="aggregation_formula">ILTAX_OUT_BALANCE_00.balance+ILTAX_OUT_BALANCE_PA.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_out_balance" model="account.report.line">
                        <field name="name">VAT Sales</field>
                        <field name="code">ILTAX_OUT_BALANCE_00</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_balance_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Sales</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_out_balance_pa" model="account.report.line">
                        <field name="name">VAT PA Sales</field>
                        <field name="code">ILTAX_OUT_BALANCE_PA</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_balance_pa_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT PA Sales</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_out_base_exempt_title" model="account.report.line">
                <field name="name">VAT Exempt Sales (BASE)</field>
                <field name="aggregation_formula">ILTAX_OUT_BASE_exempt.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_out_base_exempt" model="account.report.line">
                        <field name="name">VAT Exempt Sales (BASE)</field>
                        <field name="code">ILTAX_OUT_BASE_exempt</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_base_exempt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Exempt Sales (BASE)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_in_balance" model="account.report.line">
                <field name="name">VAT INPUTS (TAX)</field>
                <field name="code">ILTAX_IN_BALANCE</field>
                <field name="aggregation_formula">ILTAX_IN_BALANCE_17.balance+ILTAX_IN_BALANCE_18.balance+ILTAX_IN_BALANCE_2_3.balance+ILTAX_IN_BALANCE_1_4.balance+ILTAX_IN_BALANCE_PA.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_in_balance_17" model="account.report.line">
                        <field name="name">VAT Inputs 17%</field>
                        <field name="code">ILTAX_IN_BALANCE_17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs 17%</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_in_balance_18" model="account.report.line">
                        <field name="name">VAT Inputs 18%</field>
                        <field name="code">ILTAX_IN_BALANCE_18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs 18%</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_in_balance_pa_16" model="account.report.line">
                        <field name="name">VAT Inputs PA 16%</field>
                        <field name="code">ILTAX_IN_BALANCE_PA</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_pa_16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs PA 16%</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_in_balance_2_3" model="account.report.line">
                        <field name="name">VAT Inputs 2/3</field>
                        <field name="code">ILTAX_IN_BALANCE_2_3</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_2_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs 2/3</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_in_balance_1_4" model="account.report.line">
                        <field name="name">VAT Inputs 1/4</field>
                        <field name="code">ILTAX_IN_BALANCE_1_4</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_1_4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs 1/4</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_vat_in_fa_title" model="account.report.line">
                <field name="name">VAT INPUTS (fixed assets)</field>
                <field name="aggregation_formula">ILTAX_VAT_IN_FA.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_vat_in_fa" model="account.report.line">
                        <field name="name">VAT INPUTS (fixed assets)</field>
                        <field name="code">ILTAX_VAT_IN_FA</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_vat_in_fa_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT INPUTS (fixed assets)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_vat_due" model="account.report.line">
                <field name="name">VAT DUE</field>
                <field name="code">ILTAX_VAT_DUE</field>
                <field name="aggregation_formula">(ILTAX_OUT_BALANCE_00.balance + ILTAX_OUT_BALANCE_PA.balance) - (ILTAX_IN_BALANCE_17.balance + ILTAX_IN_BALANCE_18.balance + ILTAX_IN_BALANCE_2_3.balance + ILTAX_IN_BALANCE_1_4.balance + ILTAX_IN_BALANCE_PA.balance) - ILTAX_VAT_IN_FA.balance</field>
                <field name="hierarchy_level">0</field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-il.csv

```csv
"id","code","name","account_type","reconcile","tag_ids","name@he_IL"
"il_account_100100","100100","Land and buildings","asset_fixed","False","","קרקע ומבנים"
"il_account_100200","100200","Machinery and equipment","asset_fixed","False","","מכונות"
"il_account_100300","100300","Vehicles","asset_fixed","False","","כלי רכב"
"il_account_100400","100400","Other property","asset_fixed","False","","נכסים אחרים"
"il_account_101110","101110","Stock Valuation","asset_current","False","",""
"il_account_101120","101120","Stock Interim Account (Received)","asset_current","False","","חשבון זמני - מלאי שהתקבל"
"il_account_101130","101130","Stock Interim Account (Delivered)","asset_current","False","","חשבון זמני - מלאי שנשלח"
"il_account_101140","101140","Inventory - Raw materials","asset_current","False","","מלאי- חומרי גלם"
"il_account_101150","101150","Inventory - Work in progress","asset_current","False","","מלאי- עבודה בתהליך"
"il_account_101160","101160","Inventory - Finished goods","asset_current","False","","מלאי- מוצרים מוגמרים"
"il_account_101200","101200","Account Receivable","asset_receivable","True","","חשבון לקוחות"
"il_account_101201","101201","Account Receivable(POS)","asset_receivable","True","","חשבון לקוחות (קופה)"
"il_account_101250","101250","Allowance for credit losses","asset_current","True","","הפרשה לחובות מסופקים"
"il_account_101410","101410","Bank Deposit","asset_cash","False","","פקדונות"
"il_account_101420","101420","Cheques","asset_cash","False","","קופת שיקים"
"il_account_101430","101430","Financial Assets","asset_cash","False","","נכסים פיננסיים"
"il_account_101440","101440","Petty Cash","asset_cash","False","","קופה קטנה"
"il_account_101310","101310","VAT - Inputs","asset_current","False","","מע""מ תשומות"
"il_account_101320","101320","VAT - Fixed Assets","asset_current","False","","מע""מ תשומות (רכוש קבוע)"
"il_account_101330","101330","VAT - Import Transactions","asset_current","False","","מע""מ - ייבוא"
"il_account_101340","101340","VAT - PA Import Transactions","asset_current","False","","מע""מ - ייבוא מפלסטין"
"il_account_101840","101840","VAT - Inputs import line","asset_current","False","","מע""מ - תשומות ייבוא"
"il_account_100110","100110","Land and buildings depreciation","asset_non_current","False","","הוצאות פחת קרקע ומבנים"
"il_account_100210","100210","Machinery and equipment depreciation","asset_non_current","False","","הוצאות פחת מכונות"
"il_account_100310","100310","Vehicles depreciation","asset_non_current","False","","הוצאות פחת מכוניות"
"il_account_100410","100410","Other property depreciation","asset_non_current","False","","הוצאות פחת נכסים אחרים"
"il_account_100420","100420","Intangible Assets","asset_non_current","False","","נכסים בלתי מוחשיים"
"il_account_101350","101350","Prepayments","asset_current","False","","הוצאות מראש"
"il_account_111000","111000","Current Liabilities","liability_current","False","","התחייבויות שוטפות"
"il_account_111100","111100","Account Payable","liability_payable","True","","חשבון ספקים"
"il_account_111110","111110","VAT Sales","liability_current","False","","מע""מ עסקאות"
"il_account_111120","111120","VAT PA Sales","liability_current","False","","מע""מ עסקאות- פלסטין"
"il_account_111200","111200","Income tax withheld - vendors","liability_current","False","l10n_il.account_tag_retention_tax_vendor_account","ניכוי מס במקור- ספקים"
"il_account_111210","111210","Income tax withheld - employees","liability_current","False","l10n_il.account_tag_retention_tax_employees_account","ניכוי מס במקור- עובדים"
"il_account_111220","111220","Income tax withheld - dividends","liability_current","False","l10n_il.account_tag_retention_tax_dividend_account","ניכוי מס במקור- דיבידנד"
"il_account_111230","111230","Income tax withheld - customers","liability_current","False","l10n_il.account_tag_retention_tax_customers_account","ניכוי מס במקור- לקוחות"
"il_account_300300","300300","Reserve and Profit/Loss","equity","False","",""
"il_account_111400","111400","Credit cards","liability_current","False","","כרטיסי אשראי"
"il_account_111450","111450","Short term loans","liability_current","False","","הלוואות לזמן קצר"
"il_account_111460","111460","Interest Payable","liability_current","True","","ריבית לשלם"
"il_account_111500","111500","Employees wages","liability_current","True","","משכורות עובדים"
"il_account_111550","111550","Employees Benefits","liability_current","True","","הטבות לעובדים"
"il_account_112210","112210","Deferred Revenue","liability_non_current","False","","הכנסות נדחות"
"il_account_111650","111650","Advances","liability_current","True","","מקדמות מלקוחות"
"il_account_111660","111660","Other Payable","liability_current","True","","הוצאות לשלם"
"il_account_111700","111700","Income Tax","liability_current","True","","מס הכנסה"
"il_account_111710","111710","National Insurance","liability_current","True","","ביטוח לאומי"
"il_account_111720","111720","VAT due","liability_current","True","","מע""מ לתשלום"
"il_account_111730","111730","Deferred Taxes","liability_current","False","","מיסים נדחים"
"il_account_112000","112000","Non-current Liabilities","liability_non_current","False","","התחייבויות לא שוטפות"
"il_account_112100","112100","Long Term loans","liability_non_current","False","","הלוואות לטווח ארוך"
"il_account_112150","112150","Provisions","liability_non_current","False","","הפרשות"
"il_account_200000","200000","Product Sales","income","False","account.account_tag_operating","מכירות"
"il_account_200100","200100","Services Sales","income","False","account.account_tag_operating","מכירות שירות"
"il_account_200200","200200","Other Income","income_other","False","account.account_tag_operating","הכנסות אחרות"
"il_account_200300","200300","Interest Income","income_other","False","account.account_tag_operating","הכנסות מריבית"
"il_account_202000","202000","Other Expenses","expense","False","account.account_tag_operating","הכנסות אחרות"
"il_account_202100","202100","Foreign Exchange Gain&Loss","expense","False","account.account_tag_financing","הפסד ממטבע חוץ"
"il_account_202200","202200","Interest Expenses","expense","False","account.account_tag_financing","הוצאות ריבית"
"il_account_202300","202300","Donations","expense","False","","תרומות"
"il_account_202400","202400","Fines","expense","False","","קנסות"
"il_account_201000","201000","Cost of Goods Sold","expense_direct_cost","False","account.account_tag_operating","רווח ממטבע חוץ"
"il_account_211000","211000","Cost of Services Sold","expense_direct_cost","False","account.account_tag_operating","עלות המכר - שרותים שנמכרו"
"il_account_212000","212000","Customer discounts","expense","False","account.account_tag_operating","הנחות לקוח"
"il_account_212100","212100","Salary Expenses","expense","False","account.account_tag_operating","הוצאות משכורת"
"il_account_212200","212200","Purchase of Equipment","expense","False","account.account_tag_investing",""
"il_account_212300","212300","Bank Fees","expense","False","account.account_tag_financing","עמלות בנקים"
"il_account_213000","213000","Customer returns","expense_direct_cost","False","account.account_tag_operating","החזרות מלקוחות"
"il_account_214000","214000","Vendor returns","expense_direct_cost","False","account.account_tag_operating","החזרות לספקים"
"il_account_215000","215000","Vendor discounts","expense_direct_cost","False","account.account_tag_operating","הנחות מספקים"
"il_account_216000","216000","Good's Shrinkage","expense_direct_cost","False","account.account_tag_operating","התכווצות סחורה"
"il_account_217000","217000","Lost goods","expense_direct_cost","False","account.account_tag_operating","איבוד סחורה"
"il_account_220000","220000","G&A Expenses","expense","False","account.account_tag_operating","הוצאות הנהלה וכלליות"
"il_account_220100","220100","Rent Expenses","expense","False","account.account_tag_operating","הוצאות שכירות"
"il_account_220200","220200","Communication expenses","expense","False","account.account_tag_operating","הוצאות תקשורת"
"il_account_220300","220300","Transportation expenses","expense","False","account.account_tag_operating","הוצאות תחבורה"
"il_account_220400","220400","Depretiation expenses","expense","False","account.account_tag_operating","הוצאות פחת"
"il_account_220500","220500","Credit provision expenses","expense","False","account.account_tag_operating","הוצאות הפרשות אשראי"
"il_account_220600","220600","Sales and marketing expenses","expense","False","account.account_tag_operating","הוצאות מכירות ושיווק"
"il_account_220700","220700","Other non-operating expenses","expense","False","account.account_tag_operating","הוצאות אחרות שאינן תפעוליות"
"il_account_220800","220800","Bad debts expense","expense","False","account.account_tag_operating","חובות מסופקים"
"il_account_300100","300100","Capital","equity","False","","הון"
"il_account_300110","300110","Ordinary Shares","equity","False","","מניות רגילות"
"il_account_300120","300120","Preferred Shares","equity","False","","מניות מועדפות"
"il_account_300130","300130","Shares Premium","equity","False","","מניות פרימיום"
"il_account_300270","300270","current year earnings","equity_unaffected","False","","עודפים מהשנה הנוכחית"

```

## File: data\template\account.fiscal.position-il.csv

```csv
"id","name","country_id","auto_apply","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@he_IL"
"account_fiscal_position_israel","Israel","base.il","1","","","ישראל"
"account_fiscal_position_palestinian_authority","Palestinian Authority (PA)","base.ps","1","il_vat_sales_18","il_vat_pa_sales_18","הרשות הפלסטינית"
"","","","","il_vat_inputs_18","il_vat_pa_purchase_16",""
"account_fiscal_position_import_export","Import / Export","","1","il_vat_sales_18","il_vat_sales_exempt","ייבוא \ ייצוא"
"","","","","il_vat_inputs_18","",""
"account_fiscal_position_eilat","Eilat","","","il_vat_sales_18","il_vat_sales_exempt","אילת"
"","","","","il_vat_inputs_18","il_vat_purchase_exempt",""
"account_fiscal_position_vat_zero","Vat Zero","","","il_vat_sales_18","il_vat_sales_zero","מע""מ אפס"
"","","","","il_vat_inputs_18","il_vat_purchase_zero",""
"account_fiscal_position_self_invoice","Self Invoice","","","il_vat_inputs_18","il_vat_self_inv_purchase_18","חשבונית עצמית"

```

## File: data\template\account.group-il.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@he_IL"
"il_group_100100","100100","100499","Fixed Assets","רכוש קבוע"
"il_group_101110","101110","101400","Current Assets","רכוש שוטף"
"il_group_101401","101401","101799","Bank And Cash","בנק ומזומנים"
"il_group_111000","111000","111999","Current Liabilities","התחייבויות שוטפות"
"il_group_112000","112000","112210","Non-current Liabilities","התחייבויות לא שוטפות"
"il_group_200000","200000","200199","Sales Income","הכנסות ממכירות"
"il_group_200200","200200","200300","Other Income","הכנסות אחרות"
"il_group_201000","201000","201299","Cost of Goods","עלות המכר"
"il_group_202000","202000","220900","Expenses","הוצאות"
"il_group_300000","300000","399999","Capital And Shares","הון ומניות"

```

## File: data\template\account.tax-il.csv

```csv
"id","sequence","description","invoice_label","name","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@he_IL","price_include_override"
"il_vat_sales_18","1","VAT Sales","18%","18%","18.0","percent","sale","tax_group_vat_18","base","invoice","+VAT SALES (BASE)","","","מע""מ עסקאות",""
"","","","","","","","","","tax","invoice","+VAT Sales","il_account_111110","","",""
"","","","","","","","","","base","refund","-VAT SALES (BASE)","","","",""
"","","","","","","","","","tax","refund","-VAT Sales","il_account_111110","","",""
"il_vat_pa_sales_18","8","VAT PA Sales","18%","18% PA","18.0","percent","sale","tax_group_vat_18","base","invoice","+VAT SALES (BASE)","","","מע""מ עסקאות- פלסטין",""
"","","","","","","","","","tax","invoice","+VAT PA Sales","il_account_111120","","",""
"","","","","","","","","","base","refund","-VAT SALES (BASE)","","","",""
"","","","","","","","","","tax","refund","-VAT PA Sales","il_account_111120","","",""
"il_vat_sales_exempt","9","VAT exempt sales","0%","0% EXEMPT","0.0","percent","sale","tax_group_vat_exempt","base","invoice","+VAT Exempt Sales (BASE)","","","מע""מ עסקאות פטורות","tax_included"
"","","","","","","","","","tax","invoice","","il_account_111110","","",""
"","","","","","","","","","base","refund","-VAT Exempt Sales (BASE)","","","",""
"","","","","","","","","","tax","refund","","il_account_111110","","",""
"il_vat_self_inv_purchase_18","10","Self Invoice","18%","18% Self","18.0","percent","purchase","tax_group_vat_18","base","invoice","+VAT SALES (BASE)","","100","חשבונית עצמית",""
"","","","","","","","","","tax","invoice","+VAT Inputs 18%","il_account_101310","100","",""
"","","","","","","","","","tax","invoice","-VAT Sales","il_account_111110","-100","",""
"","","","","","","","","","base","refund","-VAT SALES (BASE)","","","",""
"","","","","","","","","","tax","refund","-VAT Inputs 18%","il_account_101310","100","",""
"","","","","","","","","","tax","refund","+VAT Sales","il_account_111110","-100","",""
"il_vat_inputs_18","2","VAT inputs","18%","18%","18.0","percent","purchase","tax_group_vat_18","base","invoice","","","","מע""מ תשומות",""
"","","","","","","","","","tax","invoice","+VAT Inputs 18%","il_account_101310","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-VAT Inputs 18%","il_account_101310","","",""
"il_vat_pa_purchase_16","3","VAT 16% (PA)","16%","16% PA","16.0","percent","purchase","tax_group_vat_16","base","invoice","","","","מע""מ תשומות 16%",""
"","","","","","","","","","tax","invoice","+VAT Inputs PA 16%","il_account_101340","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-VAT Inputs PA 16%","il_account_101340","","",""
"il_vat_inputs_2_3_18","4","VAT Inputs 2/3","18%","18% 2/3","18.0","percent","purchase","tax_group_vat_18","base","invoice","","","","מע""מ תשומות 2/3",""
"","","","","","","","","","tax","invoice","+VAT Inputs 2/3","il_account_101310","66.67","",""
"","","","","","","","","","tax","invoice","","","33.33","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-VAT Inputs 2/3","il_account_101310","66.67","",""
"","","","","","","","","","tax","refund","","","33.33","",""
"il_vat_inputs_1_4_18","5","VAT Inputs 1/4","18%","18% 1/4","18.0","percent","purchase","tax_group_vat_18","base","invoice","","","","מע""מ תשומות 1/4",""
"","","","","","","","","","tax","invoice","+VAT Inputs 1/4","il_account_101310","75","",""
"","","","","","","","","","tax","invoice","","","25","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-VAT Inputs 1/4","il_account_101310","75","",""
"","","","","","","","","","tax","refund","","","25","",""
"il_vat_inputs_fa_18","6","VAT inputs for fixed assets","18%","18% F A","18.0","percent","purchase","tax_group_vat_18","base","invoice","","","","מע""מ תשומות לרכוש קבוע",""
"","","","","","","","","","tax","invoice","+VAT INPUTS (fixed assets)","il_account_101320","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-VAT INPUTS (fixed assets)","il_account_101320","","",""
"il_vat_purchase_exempt","7","VAT exempt purchase","0%","0% EXEMPT","0.0","percent","purchase","tax_group_vat_exempt_purchase","base","invoice","","","","מע""מ תשומות פטורות",""
"","","","","","","","","","tax","invoice","","il_account_101310","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","il_account_101310","","",""
"il_vat_only_purchase_18","18","VAT Import Line","VAT Import Line","100% EX (18%)","100.0","division","purchase","tax_group_vat_18","base","invoice","","","","שורת מע""מ ייבוא",""
"","","","","","","","","","tax","invoice","+VAT Inputs 18%","il_account_101840","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","-VAT Inputs 18%","il_account_101840","","",""
"il_vat_purchase_zero","7","VAT Zero","0%","0%","0.0","percent","purchase","tax_group_vat_exempt_purchase","base","invoice","","","","מע""מ אפס",""
"","","","","","","","","","tax","invoice","","il_account_101310","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","il_account_101310","","",""
"il_vat_sales_zero","9","VAT Zero","0%","0%","0.0","percent","sale","tax_group_vat_exempt","base","invoice","+VAT Exempt Sales (BASE)","","","מע""מ אפס","tax_included"
"","","","","","","","","","tax","invoice","","il_account_111110","","",""
"","","","","","","","","","base","refund","-VAT Exempt Sales (BASE)","","","",""
"","","","","","","","","","tax","refund","","il_account_111110","","",""

```

## File: data\template\account.tax.group-il.csv

```csv
"id","name","country_id","name@he_IL"
"tax_group_vat_16","VAT 16%","base.il","מע""מ 16%"
"tax_group_vat_18","VAT 18%","base.il","מע""מ 18%"
"tax_group_vat_exempt","VAT exempt sale","base.il",""
"tax_group_vat_exempt_purchase","VAT exempt purchase","base.il","מע""מ תשומות פטורות"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'il')]):
        env['account.chart.template'].try_loading('il', company)

```

## File: models\template_il.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('il')
    def _get_il_template_data(self):
        return {
            'property_account_receivable_id': 'il_account_101200',
            'property_account_payable_id': 'il_account_111100',
            'property_account_expense_categ_id': 'il_account_212200',
            'property_account_income_categ_id': 'il_account_200000',
            'property_stock_account_input_categ_id': 'il_account_101120',
            'property_stock_account_output_categ_id': 'il_account_101130',
            'property_stock_valuation_account_id': 'il_account_101110',
            'code_digits': '6',
        }

    @template('il', 'res.company')
    def _get_il_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.il',
                'bank_account_code_prefix': '1014',
                'cash_account_code_prefix': '1015',
                'transfer_account_code_prefix': '1017',
                'account_default_pos_receivable_account_id': 'il_account_101201',
                'income_currency_exchange_account_id': 'il_account_201000',
                'expense_currency_exchange_account_id': 'il_account_202100',
                'account_sale_tax_id': 'il_vat_sales_18',
                'account_purchase_tax_id': 'il_vat_inputs_18',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_il

```

