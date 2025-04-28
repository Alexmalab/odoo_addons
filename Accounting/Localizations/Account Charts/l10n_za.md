# Odoo Module: l10n_za

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
    'name': 'South Africa - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['za'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest basic South African localisation necessary to run Odoo in ZA:
================================================================================
    - a generic chart of accounts
    - SARS VAT Ready Structure""",
    'author': 'Paradigm Digital (https://www.paradigmdigital.co.za)',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'depends': [
        'account',
        'base_vat',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account.account.tag.csv',
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
id,name,applicability,country_id:id
tag_ST1,ST1,taxes,base.za
tag_ST1A,ST1A,taxes,base.za
tag_ST2,ST2,taxes,base.za
tag_ST2A,ST2A,taxes,base.za
tag_ST3,ST3,taxes,base.za
tag_ST5,ST5,taxes,base.za
tag_ST7,ST7,taxes,base.za
tag_ST10,ST10,taxes,base.za
tag_ST12,ST12,taxes,base.za
tag_PT14,PT14,taxes,base.za
tag_PT14A,PT14A,taxes,base.za
tag_PT15,PT15,taxes,base.za
tag_PT15A,PT15A,taxes,base.za
tag_PT16,PT16,taxes,base.za
tag_PT17,PT17,taxes,base.za
tag_PT18,PT18,taxes,base.za

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.za"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="total_vat_payable" model="account.report.line">
                <field name="name">[20] VAT PAYABLE/REFUNDABLE (Total A - Total B)</field>
                <field name="aggregation_formula">TotalA.balance - TotalB.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="total_output_tax" model="account.report.line">
                        <field name="name">[13] Total A: TOTAL OUTPUT TAX (4 + 4A + 9 + 11 + 12)</field>
                        <field name="code">TotalA</field>
                        <field name="aggregation_formula">VAT4.balance + VAT4A.balance + (SEC6.balance * 0.6 + SEC7.balance) + VAT11.balance + VAT12.balance</field>
                        <field name="children_ids">
                            <record id="standard_rate_exclude_capital_goods_service" model="account.report.line">
                                <field name="name">[1] Standard Rate (Excluding Capital goods and/or services and accomodation)</field>
                                <field name="code">VAT1</field>
                                <field name="expression_ids">
                                    <record id="standard_rate_exclude_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[1] Standard Rate (Excluding Capital goods and/or services and accomodation)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_on_standard_rate_exclude_capital_goods_service" model="account.report.line">
                                <field name="name">[4] x 15/ (100 + 15)</field>
                                <field name="code">VAT4</field>
                                <field name="expression_ids">
                                    <record id="vat_on_standard_rate_exclude_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[4] x 15/ (100 + 15)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="standard_rate_only_capital_goods_service" model="account.report.line">
                                <field name="name">[1A] Standard Rate (Only Capital goods and/or services)</field>
                                <field name="code">VAT1A</field>
                                <field name="expression_ids">
                                    <record id="standard_rate_only_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[1A] Standard Rate (Only Capital goods and/or services)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_on_standard_rate_only_capital_goods_service" model="account.report.line">
                                <field name="name">[4A] x 15/ (100 + 15)</field>
                                <field name="code">VAT4A</field>
                                <field name="expression_ids">
                                    <record id="vat_on_standard_rate_only_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[4A] x 15/ (100 + 15)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="zero_rate_exclude_goods_exported" model="account.report.line">
                                <field name="name">[2] Zero Rate (excluding goods exported)</field>
                                <field name="code">VAT2</field>
                                <field name="expression_ids">
                                    <record id="zero_rate_exclude_goods_exported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[2] Zero Rate (excluding goods exported)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="zero_rate_only_goods_exported" model="account.report.line">
                                <field name="name">[2A] Zero Rate (Only goods exported)</field>
                                <field name="code">VAT2A</field>
                                <field name="expression_ids">
                                    <record id="zero_rate_only_goods_exported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[2A] Zero Rate (Only goods exported)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="exempt_and_non_supplies" model="account.report.line">
                                <field name="name">[3] Exempt and Non supplies</field>
                                <field name="code">VAT3</field>
                                <field name="expression_ids">
                                    <record id="exempt_and_non_supplies_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[3] Exempt and Non supplies</field>
                                    </record>
                                </field>
                            </record>
                            <record id="accomodation_exceeding_28_days" model="account.report.line">
                                <field name="name">[5] Accomodation exceeding 28 days</field>
                                <field name="code">VAT5</field>
                                <field name="expression_ids">
                                    <record id="accomodation_exceeding_28_days_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[5] Accomodation exceeding 28 days</field>
                                    </record>
                                </field>
                            </record>
                            <record id="accomodation_exceeding_28_days_60_percent" model="account.report.line">
                                <field name="name">[6] x 60%</field>
                                <field name="aggregation_formula">VAT5.balance * 0.6</field>
                            </record>
                            <record id="accomodation_under_28_days" model="account.report.line">
                                <field name="name">[7] Accomodation under 28 days</field>
                                <field name="code">VAT7</field>
                                <field name="expression_ids">
                                    <record id="accomodation_under_28_days_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[7] Accomodation under 28 days</field>
                                    </record>
                                </field>
                            </record>
                            <record id="accomodation_28_days" model="account.report.line">
                                <field name="name">[8] Total (6 + 7)</field>
                                <field name="aggregation_formula">(VAT5.balance * 0.6) + VAT7.balance</field>
                            </record>
                            <record id="vat_on_accomodation_28_days" model="account.report.line">
                                <field name="name">[9] x 15 / (100 ??) </field>
                                <field name="aggregation_formula">SEC6.balance * 0.6 + SEC7.balance </field>
                                <field name="children_ids">
                                    <record id="vat_on_accomodation_exceeding_28_days" model="account.report.line">
                                        <field name="name">VAT on Accomodation exceeding 28 days</field>
                                        <field name="code">SEC6</field>
                                        <field name="expression_ids">
                                            <record id="vat_on_accomodation_exceeding_28_days_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">VAT on Accomodation exceeding 28 days</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="vat_on_accomodation_under_28_days" model="account.report.line">
                                        <field name="name">VAT on Accomodation under 28 days</field>
                                        <field name="code">SEC7</field>
                                        <field name="expression_ids">
                                            <record id="vat_on_accomodation_under_28_days_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">VAT on Accomodation under 28 days</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_plus_base_of_second_hand_goods_export" model="account.report.line">
                                <field name="name">[10] Change in use and export of second-hand goods</field>
                                <field name="aggregation_formula">VAT10a.balance + VAT11.balance</field>
                                <field name="children_ids">
                                    <record id="second_hand_goods_export" model="account.report.line">
                                        <field name="name">[10] Base Amount: Change in use and export of second-hand goods</field>
                                        <field name="code">VAT10a</field>
                                        <field name="expression_ids">
                                            <record id="second_hand_goods_export_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">[10] Change in use and export of second-hand goods</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_on_second_hand_goods_export" model="account.report.line">
                                <field name="name">[11] x 15 / (100 + 15)</field>
                                <field name="code">VAT11</field>
                                <field name="expression_ids">
                                    <record id="vat_on_second_hand_goods_export_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[11] x 15 / (100 + 15)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="other_imported_services" model="account.report.line">
                                <field name="name">[12] Other and imported services</field>
                                <field name="code">VAT12</field>
                                <field name="expression_ids">
                                    <record id="other_imported_services_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[12] Other and imported services</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="total_input_tax" model="account.report.line">
                        <field name="name">[19] Total B: TOTAL INPUT TAX (14 + 14A + 15 + 15A + 16 + 17 + 18)</field>
                        <field name="code">TotalB</field>
                        <field name="aggregation_formula">VAT14.balance + VAT14A.balance + VAT15.balance + VAT15A.balance + VAT16.balance + VAT17.balance + VAT18.balance</field>
                        <field name="children_ids">
                            <record id="capital_goods_services_supplied" model="account.report.line">
                                <field name="name">[14] Capital Goods and/or services supplied to you</field>
                                <field name="code">VAT14</field>
                                <field name="expression_ids">
                                    <record id="capital_goods_services_supplied_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[14] Capital Goods and/or services supplied to you</field>
                                    </record>
                                </field>
                            </record>
                            <record id="capital_goods_imported" model="account.report.line">
                                <field name="name">[14A] Capital Goods imported by you</field>
                                <field name="code">VAT14A</field>
                                <field name="expression_ids">
                                    <record id="capital_goods_imported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[14A] Capital Goods imported by you</field>
                                    </record>
                                </field>
                            </record>
                            <record id="other_goods_services_supplied" model="account.report.line">
                                <field name="name">[15] Other goods and/or services supplied to you (not Capital Goods)</field>
                                <field name="code">VAT15</field>
                                <field name="expression_ids">
                                    <record id="other_goods_services_supplied_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[15] Other goods and/or services supplied to you (not Capital Goods)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="other_goods_imported" model="account.report.line">
                                <field name="name">[15A] Other goods imported by you (not Capital Goods)</field>
                                <field name="code">VAT15A</field>
                                <field name="expression_ids">
                                    <record id="other_goods_imported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[15A] Other goods imported by you (not Capital Goods)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="change_in_use" model="account.report.line">
                                <field name="name">[16] Change in Use</field>
                                <field name="code">VAT16</field>
                                <field name="expression_ids">
                                    <record id="change_in_use_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[16] Change in Use</field>
                                    </record>
                                </field>
                            </record>
                            <record id="bad_debts" model="account.report.line">
                                <field name="name">[17] Bad Debts</field>
                                <field name="code">VAT17</field>
                                <field name="expression_ids">
                                    <record id="bad_debts_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[17] Bad Debts</field>
                                    </record>
                                </field>
                            </record>
                            <record id="others" model="account.report.line">
                                <field name="name">[18] Other</field>
                                <field name="code">VAT18</field>
                                <field name="expression_ids">
                                    <record id="others_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[18] Other</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-za.csv

```csv
"id","code","name","account_type","reconcile"
"100020","100020","Stock Valuation","asset_current","True"
"100030","100030","Stock Work In Progress","asset_current","False"
"100040","100040","Stock Finished Goods","asset_current","False"
"100050","100050","Stock Delivered Control","asset_current","True"
"100060","100060","Purchase Tax Control","asset_current","False"
"100070","100070","Other Current Assets","asset_current","False"
"100800","100800","Cash Control Account","asset_current","True"
"110010","110010","Debtors Control","asset_receivable","True"
"110020","110020","Sundry Debtors","asset_receivable","True"
"110030","110030","Debtors Control Account (PoS)","asset_receivable","True"
"124010","124010","Credit Card Merchant","asset_cash","False"
"126010","126010","Cash In Hand","asset_cash","False"
"130010","130010","Prepayments","asset_prepayments","False"
"140010","140010","Software","asset_fixed","False"
"140020","140020","Patents & Trademarks","asset_fixed","False"
"140030","140030","Fixtures & Fittings","asset_fixed","False"
"140040","140040","Land & Buildings","asset_fixed","False"
"140050","140050","Motor Vehicles","asset_fixed","False"
"140060","140060","Office Equipment (incl computer equipment)","asset_fixed","False"
"140070","140070","Plant & Machinery","asset_fixed","False"
"150010","150010","Non-current assets","asset_non_current","False"
"200010","200010","Stock Received Control","liability_current","True"
"200020","200020","Sundry Creditors","liability_current","False"
"200030","200030","Other Creditors","liability_current","False"
"200040","200040","Accruals","liability_current","False"
"200050","200050","Bad debt provision","liability_current","False"
"200060","200060","Sales Tax Control","liability_current","False"
"200070","200070","Manual Adjustments & VAT","liability_current","False"
"200080","200080","Loans","liability_current","False"
"200090","200090","Hire Purchase","liability_current","False"
"200100","200100","Mortgages","liability_current","False"
"210010","210010","Company Credit Card","liability_credit_card","False"
"220010","220010","Creditors Control","liability_payable","True"
"220020","220020","SARS - VAT","liability_payable","True"
"220030","220030","P.A.Y.E. & UIF","liability_payable","True"
"220040","220040","Net Wages","liability_payable","True"
"220050","220050","Pension Fund","liability_payable","True"
"220060","220060","Corporation Tax","liability_payable","True"
"300010","300010","Called up share capital","equity","False"
"300020","300020","Share premium","equity","False"
"300030","300030","Revaluation reserve","equity","False"
"300040","300040","Other reserves","equity","False"
"300050","300050","Capital","equity","False"
"300060","300060","Dividends","equity","False"
"300070","300070","Drawings","equity","False"
"300080","300080","Directors Loans","equity","False"
"400010","400010","Undistributed Profits/Losses","equity_unaffected","False"
"500010","500010","Sales category 1","income","False"
"500020","500020","Sales category 2","income","False"
"500030","500030","Sales category 3","income","False"
"500040","500040","Sales category 4","income","False"
"500050","500050","Bank Interest received","income","False"
"500060","500060","Investment Interest received","income","False"
"500070","500070","Profits/Losses on disposals of assets","income","False"
"500080","500080","Rental Income","income","False"
"500090","500090","Discount Received","income","False"
"500100","500100","Foreign Exchange Gains","income","False"
"500110","500110","Cash Difference Gains","income","False"
"510010","510010","Other Income","income_other","False"
"600010","600010","Cost of sales 1","expense_direct_cost","False"
"600020","600020","Cost of sales 2","expense_direct_cost","False"
"600030","600030","Cost of sales 3","expense_direct_cost","False"
"600040","600040","Cost of sales 4","expense_direct_cost","False"
"610010","610010","Marketing","expense","False"
"610020","610020","Exhibitions and events","expense","False"
"610030","610030","Public Relations","expense","False"
"610040","610040","Distribution vehicles","expense","False"
"610050","610050","Distribution salaries and wages","expense","False"
"610060","610060","Shipping","expense","False"
"610070","610070","Directors pension","expense","False"
"610080","610080","Directors remuneration","expense","False"
"610090","610090","Gross Salaries","expense","False"
"610100","610100","Employers SDL & UIF","expense","False"
"610110","610110","Subcontractors payments","expense","False"
"610120","610120","Rent and rates","expense","False"
"610130","610130","Water, electricity and other utilities","expense","False"
"610140","610140","Repairs and maintenance","expense","False"
"610150","610150","Car hire","expense","False"
"610160","610160","Car fuel","expense","False"
"610170","610170","Car maintenance","expense","False"
"610180","610180","Telephone","expense","False"
"610190","610190","Internet & hosting","expense","False"
"610200","610200","Mobiles","expense","False"
"610210","610210","Stationery","expense","False"
"610220","610220","Office consumables","expense","False"
"610230","610230","Postage and Carriage","expense","False"
"610240","610240","Books","expense","False"
"610250","610250","Network costs","expense","False"
"610260","610260","Software expenses","expense","False"
"610270","610270","Other computer costs","expense","False"
"610280","610280","Recruitment fees","expense","False"
"610290","610290","Other admin expenses","expense","False"
"610300","610300","Accounting","expense","False"
"610310","610310","Auditing","expense","False"
"610320","610320","Consultancy","expense","False"
"610330","610330","Legal and professional charges","expense","False"
"610340","610340","Foreign Exchange Losses","expense","False"
"610350","610350","Other sundry expenses","expense","False"
"610360","610360","Bad debts","expense","False"
"610370","610370","Interest paid","expense","False"
"610380","610380","Bank Charges","expense","False"
"610390","610390","Donations","expense","False"
"610400","610400","Entertaining","expense","False"
"610410","610410","Insurance","expense","False"
"610420","610420","Travel and subsistence","expense","False"
"610430","610430","Corporation tax expense","expense","False"
"610450","610450","Price Differences Control","expense","False"
"610460","610460","Cash Difference Loss","expense","False"
"610470","610470","Discount Given","expense","False"
"620010","620010","Software Depreciation","expense_depreciation","False"
"620020","620020","Patents & Trademarks Depreciation","expense_depreciation","False"
"620030","620030","Fixtures and fittings Depreciation","expense_depreciation","False"
"620040","620040","Land and buildings Depreciation","expense_depreciation","False"
"620050","620050","Motor vehicles Depreciation","expense_depreciation","False"
"620060","620060","Office equipment (inc computer equipment) Depreciation","expense_depreciation","False"
"620070","620070","Plant and machinery Depreciation","expense_depreciation","False"
"620080","620080","Depreciation","expense_depreciation","False"

```

## File: data\template\account.tax-za.csv

```csv
"id","description","invoice_label","type_tax_use","name","amount_type","amount","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id"
"ST1","Standard Rate","15%","sale","15%","percent","15.0","tax_group_1","base","invoice","+[1] Standard Rate (Excluding Capital goods and/or services and accomodation)",""
"","","","","","","","","tax","invoice","+[1] Standard Rate (Excluding Capital goods and/or services and accomodation)||+[4] x 15/ (100 + 15)","200060"
"","","","","","","","","base","refund","-[1] Standard Rate (Excluding Capital goods and/or services and accomodation)",""
"","","","","","","","","tax","refund","-[1] Standard Rate (Excluding Capital goods and/or services and accomodation)||-[4] x 15/ (100 + 15)","200060"
"ST1A","Standard Rate (Capital Goods)","15%","sale","15% G","percent","15.0","tax_group_1","base","invoice","+[1A] Standard Rate (Only Capital goods and/or services)",""
"","","","","","","","","tax","invoice","+[1A] Standard Rate (Only Capital goods and/or services)||+[4A] x 15/ (100 + 15)","200060"
"","","","","","","","","base","refund","-[1A] Standard Rate (Only Capital goods and/or services)",""
"","","","","","","","","tax","refund","-[1A] Standard Rate (Only Capital goods and/or services)||-[4A] x 15/ (100 + 15)","200060"
"ST2","Zero Rate","0%","sale","0%","percent","0.0","tax_group_0","base","invoice","+[2] Zero Rate (excluding goods exported)",""
"","","","","","","","","tax","invoice","",""
"","","","","","","","","base","refund","-[2] Zero Rate (excluding goods exported)",""
"","","","","","","","","tax","refund","",""
"ST2A","Zero Rate Exports","0%","sale","0% EX","percent","0.0","tax_group_0","base","invoice","+[2A] Zero Rate (Only goods exported)",""
"","","","","","","","","tax","invoice","",""
"","","","","","","","","base","refund","-[2A] Zero Rate (Only goods exported)",""
"","","","","","","","","tax","refund","",""
"ST3","Exempt and Non-Supplies","0%","sale","0% EXEMPT","percent","0.0","tax_group_0","base","invoice","+[3] Exempt and Non supplies",""
"","","","","","","","","tax","invoice","",""
"","","","","","","","","base","refund","-[3] Exempt and Non supplies",""
"","","","","","","","","tax","refund","",""
"ST5","Accommodation (28+ days)","15%","sale","15% A 28+","percent","15.0","tax_group_1","base","invoice","+[5] Accomodation exceeding 28 days",""
"","","","","","","","","tax","invoice","+VAT on Accomodation exceeding 28 days","200060"
"","","","","","","","","base","refund","-[5] Accomodation exceeding 28 days",""
"","","","","","","","","tax","refund","-VAT on Accomodation exceeding 28 days","200060"
"ST7","Accommodation (Under 28 days)","15%","sale","15% A 28-","percent","15.0","tax_group_1","base","invoice","+[7] Accomodation under 28 days",""
"","","","","","","","","tax","invoice","+VAT on Accomodation under 28 days","200060"
"","","","","","","","","base","refund","-[7] Accomodation under 28 days",""
"","","","","","","","","tax","refund","-VAT on Accomodation under 28 days","200060"
"ST10","Export of Second-hand Goods/ Change in Use","15%","sale","15% EX","percent","15.0","tax_group_1","base","invoice","+[10] Change in use and export of second-hand goods",""
"","","","","","","","","tax","invoice","+[11] x 15 / (100 + 15)","200060"
"","","","","","","","","base","refund","-[10] Change in use and export of second-hand goods",""
"","","","","","","","","tax","refund","-[11] x 15 / (100 + 15)","200060"
"ST12","VAT Adjustments and Manual VAT","15%","sale","15% Adj","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[12] Other and imported services","200060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[12] Other and imported services","200060"
"PT15","Standard Rate","15%","purchase","15%","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[15] Other goods and/or services supplied to you (not Capital Goods)","100060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[15] Other goods and/or services supplied to you (not Capital Goods)","100060"
"PT14","Standard Rate (Capital Goods)","15%","purchase","15% G","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[14] Capital Goods and/or services supplied to you","100060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[14] Capital Goods and/or services supplied to you","100060"
"PT14A","Capital Goods Imported","15%","purchase","15% EX G","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[14A] Capital Goods imported by you","100060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[14A] Capital Goods imported by you","100060"
"PT15A","Goods and Services Imported","15%","purchase","15% EX","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[15A] Other goods imported by you (not Capital Goods)","100060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[15A] Other goods imported by you (not Capital Goods)","100060"
"PT16","Change in Use","15%","purchase","15% C","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[16] Change in Use","100060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[16] Change in Use","100060"
"PT17","Bad Debts","15%","purchase","15% B D","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[17] Bad Debts","100060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[17] Bad Debts","100060"
"PT18","Other Adjustments","15%","purchase","15% O A","percent","15.0","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","+[18] Other","100060"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","-[18] Other","100060"

```

## File: data\template\account.tax.group-za.csv

```csv
"id","name","country_id"
"tax_group_0","VAT 0%","base.za"
"tax_group_1","VAT 15%","base.za"

```

## File: models\template_za.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('za')
    def _get_za_template_data(self):
        return {
            'property_account_receivable_id': '110010',
            'property_account_payable_id': '220010',
            'property_account_expense_categ_id': '600010',
            'property_account_income_categ_id': '500010',
            'property_stock_account_input_categ_id': '200010',
            'property_stock_account_output_categ_id': '100050',
            'property_stock_valuation_account_id': '100020',
            'code_digits': '6',
        }

    @template('za', 'res.company')
    def _get_za_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.za',
                'bank_account_code_prefix': '1200',
                'cash_account_code_prefix': '1250',
                'transfer_account_code_prefix': '1010',
                'account_default_pos_receivable_account_id': '110030',
                'income_currency_exchange_account_id': '500100',
                'expense_currency_exchange_account_id': '610340',
                'default_cash_difference_income_account_id': '500110',
                'default_cash_difference_expense_account_id': '610460',
                'account_sale_tax_id': 'ST1',
                'account_purchase_tax_id': 'PT15',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_za

```

