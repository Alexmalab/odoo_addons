# Odoo Module: l10n_th

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

def _preserve_tag_on_taxes(env):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(env, 'l10n_th')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Thailand - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['th'],
    'version': '2.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Chart of Accounts for Thailand.
===============================

Thai accounting chart and localization.
    """,
    'author': 'Almacom (http://almacom.co.th/)',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/thailand.html',
    'depends': [
        'account_qr_code_emv',
    ],
    'data': [
        'data/account_tax_report_data.xml',
        'views/report_invoice.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_preserve_tag_on_taxes',
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
        <field name="country_id" ref="base.th"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_out_tax_title" model="account.report.line">
                <field name="name">Output Tax</field>
                <field name="aggregation_formula">OUTPUTTAX_SALEAMOUNT.balance + OUTPUTTAX_SALE_ZERO.balance + OUTPUTTAX_EXEMPTED.balance + OUTPUTTAX_TAX.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_out_tax_sale" model="account.report.line">
                        <field name="name">1. Sales amount</field>
                        <field name="code">OUTPUTTAX_SALEAMOUNT</field>
                        <field name="expression_ids">
                            <record id="tax_report_out_tax_sale_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1. Sales amount</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_out_tax_less_sales_0_rate" model="account.report.line">
                        <field name="name">2. Less sales subject to 0% tax rate </field>
                        <field name="code">OUTPUTTAX_SALE_ZERO</field>
                        <field name="expression_ids">
                            <record id="tax_report_out_tax_less_sales_0_rate_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2. Less sales subject to 0% tax rate </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_out_tax_less_exempted_sales" model="account.report.line">
                        <field name="name">3. Less exempted sales</field>
                        <field name="code">OUTPUTTAX_EXEMPTED</field>
                        <field name="expression_ids">
                            <record id="tax_report_out_tax_less_exempted_sales_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3. Less exempted sales</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_out_tax_taxable_sales_amount" model="account.report.line">
                        <field name="name">4. Taxable sales amount(1. -2. -3.)</field>
                        <field name="aggregation_formula">OUTPUTTAX_SALEAMOUNT.balance-(OUTPUTTAX_SALE_ZERO.balance+OUTPUTTAX_EXEMPTED.balance)</field>
                    </record>
                    <record id="tax_report_out_tax" model="account.report.line">
                        <field name="name">5. Output tax</field>
                        <field name="code">OUTPUTTAX_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_out_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5. Output tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_input_tax_title" model="account.report.line">
                <field name="name">Input Tax</field>
                <field name="code">INPUTTAX</field>
                <field name="aggregation_formula">6_PURCHASE_AMOUNT_THAT_IS_ENTITLED_TO_DEDUCTION_OF_INPUT_TAX_FROM_OUTPUT_TAX_IN_TAX_COMPUTATION.balance + INPUTTAX_TAX.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_input_tax_purchase_from_out_tax" model="account.report.line">
                        <field name="name">6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation</field>
                        <field name="code">6_PURCHASE_AMOUNT_THAT_IS_ENTITLED_TO_DEDUCTION_OF_INPUT_TAX_FROM_OUTPUT_TAX_IN_TAX_COMPUTATION</field>
                        <field name="expression_ids">
                            <record id="tax_report_input_tax_purchase_from_out_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_input_tax" model="account.report.line">
                        <field name="name">7. Input tax (according to invoice of purchase amount in 6.)</field>
                        <field name="code">INPUTTAX_TAX</field>
                        <field name="expression_ids">
                            <record id="tax_report_input_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7. Input tax (according to invoice of purchase amount in 6.)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_vat" model="account.report.line">
                <field name="name">Value Added Tax</field>
                <field name="aggregation_formula">10_EXCESS_TAX_PAYMENT_CARRIED_FORWARD_FROM_LAST_PERIOD.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_vat_payable" model="account.report.line">
                        <field name="name">8. Tax payable (5. minus 7. (if 5. is greater than 7.))</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_payable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">OUTPUTTAX_TAX.balance - INPUTTAX_TAX.balance</field>
                                <field name="subformula">if_above(THB(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_excess" model="account.report.line">
                        <field name="name">9. Excess tax payable (7. minus 5. (if 5. is less than 7.))</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_excess_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">INPUTTAX_TAX.balance - OUTPUTTAX_TAX.balance</field>
                                <field name="subformula">if_above(THB(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_payment_last_period" model="account.report.line">
                        <field name="name">10. Excess tax payment carried forward from last period</field>
                        <field name="code">10_EXCESS_TAX_PAYMENT_CARRIED_FORWARD_FROM_LAST_PERIOD</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_payment_last_period_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_net_vat" model="account.report.line">
                <field name="name">Net Tax</field>
                <field name="aggregation_formula">11_NET_TAX_PAYABLE_IF_8_IS_GREATER_THAN_10.balance + 12_NET_EXCESS_TAX_PAYABLE_IF_10_IS_GREATER_THAN_8_OR_9_PLUS_10.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_net_vat_payable" model="account.report.line">
                        <field name="name">11. Net tax payable (if 8. is greater than 10.)</field>
                        <field name="code">11_NET_TAX_PAYABLE_IF_8_IS_GREATER_THAN_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_net_vat_payable_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11. Net tax payable (if 8. is greater than 10.)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_net_vat_excess" model="account.report.line">
                        <field name="name">12. Net excess tax payable ((if 10. is greater than 8.) or (9. plus 10.))</field>
                        <field name="code">12_NET_EXCESS_TAX_PAYABLE_IF_10_IS_GREATER_THAN_8_OR_9_PLUS_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_net_vat_excess_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12. Net excess tax payable ((if 10. is greater than 8.) or (9. plus 10.))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>

    <record id="tax_report_pnd53" model="account.report">
        <field name="name">PND53</field>
        <field name="root_report_id" ref="account.generic_tax_report" />
        <field name="country_id" ref="base.th" />
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_pnd53_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_total_income_pnd53_line" model="account.report.line">
                <field name="name">Total Income</field>
                <field name="code">INCOME_PND53</field>
                <field name="expression_ids">
                    <record id="tax_report_total_income_pnd53" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Income PND53</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_total_remittance_pnd53_line" model="account.report.line">
                <field name="name">Total Remittance</field>
                <field name="code">P53</field>
                <field name="expression_ids">
                    <record id="tax_report_total_remittance_pnd53" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">PND53</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_surcharge_pnd53_line" model="account.report.line">
                <field name="name">Surcharge</field>
                <field name="code">S53</field>
                <field name="expression_ids">
                    <record id="tax_report_surcharge_pnd53" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">SUR53</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_total_pnd53_line" model="account.report.line">
                <field name="name">Total</field>
                <field name="expression_ids">
                    <record id="tax_report_total_pnd53" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">P53.balance + S53.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>

    <record id="tax_report_pnd3" model="account.report">
        <field name="name">PND3</field>
        <field name="root_report_id" ref="account.generic_tax_report" />
        <field name="country_id" ref="base.th" />
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_pnd3_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_total_income_pnd3_line" model="account.report.line">
                <field name="name">Total Income</field>
                <field name="code">INCOME_PND3</field>
                <field name="expression_ids">
                    <record id="tax_report_total_income_pnd3" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Income PND3</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_total_remittance_pnd3_line" model="account.report.line">
                <field name="name">Total Remittance</field>
                <field name="code">P3</field>
                <field name="expression_ids">
                    <record id="tax_report_total_remittance_pnd3" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">PND3</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_surcharge_pnd3_line" model="account.report.line">
                <field name="name">Surcharge</field>
                <field name="code">S3</field>
                <field name="expression_ids">
                    <record id="tax_report_surcharge_pnd3" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">SUR3</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_total_pnd3_line" model="account.report.line">
                <field name="name">Total</field>
                <field name="expression_ids">
                    <record id="tax_report_total_pnd3" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">P3.balance + S3.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>

</odoo>

```

## File: data\template\account.account-th.csv

```csv
"id","name","code","account_type","reconcile"
"a_recv","Account Receivable","1200","asset_receivable","True"
"a_recv_pos","Account Receivable (PoS)","1210","asset_receivable","True"
"a_cheque","Outstanding Cheques","1201","liability_current","True"
"a_invent","Inventory","1300","asset_current","False"
"a_building_depr","Accumulated Depreciation - Building","1411","expense_depreciation","False"
"a_equipment_dept","Accumulated Depreciation - Equipment","1421","expense_depreciation","False"
"a_input_vat","Input VAT","1510","asset_current","False"
"a_wht_income","Withholding Income Tax","1520","asset_current","False"
"a_pay","Account Payable","2101","liability_payable","True"
"a_accr_exp","Accrued Expenses","2102","liability_current","True"
"a_uninvoiced_receipts","Uninvoiced Receipts","2103","liability_current","True"
"a_loan","Loans","2200","liability_current","True"
"a_output_vat","Output VAT","2310","liability_current","False"
"a_wht","Withholding Tax","2320","liability_current","False"
"a_common_stock","Capital Stock","3100","equity","False"
"a_retained_earnings","Retained Earnings","3200","equity","False"
"a_dividends","Dividends","3300","equity","False"
"a_income_summary","Income Summary","3400","equity","False"
"a_sales","Income","4100","income","False"
"a_income_gain","Gain Account","4200","income_other","False"
"a_exp_cogs","Cost of Revenue","5100","expense_direct_cost","False"
"a_exp_salary","Salary","5201","expense","False"
"a_exp_rent","Rent","5202","expense","False"
"a_exp_office","Office Expenses","5203","expense","False"
"a_exp_interest","Interest expenses","5300","expense","False"
"a_exp_income_tax","Income tax expenses","5400","expense","False"
"a_exp_loss","Loss Account","5500","expense","False"

```

## File: data\template\account.tax-th.csv

```csv
"id","name","description","invoice_label","amount_type","amount","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id"
"tax_input_vat","7%","Input VAT 7%","","percent","7.0","purchase","tax_group_vat_7","base","invoice","+6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation",""
"","","","","","","","","tax","invoice","+7. Input tax (according to invoice of purchase amount in 6.)","a_input_vat"
"","","","","","","","","base","refund","-6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation",""
"","","","","","","","","tax","refund","-7. Input tax (according to invoice of purchase amount in 6.)","a_input_vat"
"tax_output_vat","7%","Output VAT 7%","","percent","7.0","sale","tax_group_vat_7","base","invoice","+1. Sales amount",""
"","","","","","","","","tax","invoice","+5. Output tax","a_output_vat"
"","","","","","","","","base","refund","-1. Sales amount",""
"","","","","","","","","tax","refund","-5. Output tax","a_output_vat"
"tax_input_vat_0","0%","Input VAT 0%","","percent","0.0","purchase","","base","invoice","+6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation",""
"","","","","","","","","tax","invoice","+7. Input tax (according to invoice of purchase amount in 6.)","a_input_vat"
"","","","","","","","","base","refund","-6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation",""
"","","","","","","","","tax","refund","+7. Input tax (according to invoice of purchase amount in 6.)","a_input_vat"
"tax_output_vat_0","0%","Output VAT 0%","","percent","0.0","sale","","base","invoice","+1. Sales amount||+2. Less sales subject to 0% tax rate",""
"","","","","","","","","tax","invoice","+5. Output tax","a_output_vat"
"","","","","","","","","base","refund","-1. Sales amount||-2. Less sales subject to 0% tax rate",""
"","","","","","","","","tax","refund","-5. Output tax","a_output_vat"
"tax_input_vat_exempted","0% EXEMPT","Input VAT Exempted","","percent","0.0","purchase","","base","invoice","+6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation",""
"","","","","","","","","tax","invoice","+7. Input tax (according to invoice of purchase amount in 6.)","a_input_vat"
"","","","","","","","","base","refund","-6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation",""
"","","","","","","","","tax","refund","-7. Input tax (according to invoice of purchase amount in 6.)","a_input_vat"
"tax_output_vat_exempted","0% EXEMPT","Output VAT Exempted","","percent","0.0","sale","","base","invoice","+1. Sales amount||+3. Less exempted sales",""
"","","","","","","","","tax","invoice","+5. Output tax","a_output_vat"
"","","","","","","","","base","refund","-1. Sales amount||-3. Less exempted sales",""
"","","","","","","","","tax","refund","-5. Output tax","a_output_vat"
"tax_wht_co_1","1% WH C T","Company Withholding Tax 1% (Transportation)","","percent","-1.0","purchase","tax_group_1","base","invoice","+Income PND53",""
"","","","","","","","","tax","invoice","+PND53","a_wht"
"","","","","","","","","base","refund","-Income PND53",""
"","","","","","","","","tax","refund","-PND53","a_wht"
"tax_wht_co_2","2% WH C A","Company Withholding Tax 2% (Advertising)","","percent","-2.0","purchase","tax_group_2","base","invoice","+Income PND53",""
"","","","","","","","","tax","invoice","+PND53","a_wht"
"","","","","","","","","base","refund","-Income PND53",""
"","","","","","","","","tax","refund","-PND53","a_wht"
"tax_wht_co_3","3% WH C S","Company Withholding Tax 3% (Service)","","percent","-3.0","purchase","tax_group_3","base","invoice","+Income PND53",""
"","","","","","","","","tax","invoice","+PND53","a_wht"
"","","","","","","","","base","refund","-Income PND53",""
"","","","","","","","","tax","refund","-PND53","a_wht"
"tax_wht_co_5","5% WH C R","Company Withholding Tax 5% (Rental)","","percent","-5.0","purchase","tax_group_5","base","invoice","+Income PND53",""
"","","","","","","","","tax","invoice","+PND53","a_wht"
"","","","","","","","","base","refund","-Income PND53",""
"","","","","","","","","tax","refund","-PND53","a_wht"
"tax_wht_pers_1","1% WH P T","Personal Withholding Tax 1% (Transportation)","","percent","-1.0","purchase","tax_group_1","base","invoice","+Income PND3",""
"","","","","","","","","tax","invoice","+PND3","a_wht"
"","","","","","","","","base","refund","-Income PND3",""
"","","","","","","","","tax","refund","-PND3","a_wht"
"tax_wht_pers_2","2% WH P A","Personal Withholding Tax 2% (Advertising)","","percent","-2.0","purchase","tax_group_2","base","invoice","+Income PND3",""
"","","","","","","","","tax","invoice","+PND3","a_wht"
"","","","","","","","","base","refund","-Income PND3",""
"","","","","","","","","tax","refund","-PND3","a_wht"
"tax_wht_pers_3","3% WH P S","Personal Withholding Tax 3% (Service)","","percent","-3.0","purchase","tax_group_3","base","invoice","+Income PND3",""
"","","","","","","","","tax","invoice","+PND3","a_wht"
"","","","","","","","","base","refund","-Income PND3",""
"","","","","","","","","tax","refund","-PND3","a_wht"
"tax_wht_pers_5","5% WH P R","Personal Withholding Tax 5% (Rental)","","percent","-5.0","purchase","tax_group_5","base","invoice","+Income PND3",""
"","","","","","","","","tax","invoice","+PND3","a_wht"
"","","","","","","","","base","refund","-Income PND3",""
"","","","","","","","","tax","refund","-PND3","a_wht"
"tax_wht_income_1","1% WH T","Withholding Income Tax 1% (Transportation)","","percent","-1.0","sale","tax_group_1","base","invoice","",""
"","","","","","","","","tax","invoice","","a_wht_income"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","","a_wht_income"
"tax_wht_income_2","2% WH A","Withholding Income Tax 2% (Advertising)","","percent","-2.0","sale","tax_group_2","base","invoice","",""
"","","","","","","","","tax","invoice","","a_wht_income"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","","a_wht_income"
"tax_wht_income_3","3% WH S","Withholding Income Tax 3% (Service)","","percent","-3.0","sale","tax_group_3","base","invoice","",""
"","","","","","","","","tax","invoice","","a_wht_income"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","","a_wht_income"
"tax_wht_income_5","5% WH R","Withholding Income Tax 5% (Rental)","","percent","-5.0","sale","tax_group_5","base","invoice","",""
"","","","","","","","","tax","invoice","","a_wht_income"
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","","a_wht_income"

```

## File: data\template\account.tax.group-th.csv

```csv
"id","name","country_id"
"tax_group_1","TAX 1%","base.th"
"tax_group_2","TAX 2%","base.th"
"tax_group_3","TAX 3%","base.th"
"tax_group_5","TAX 5%","base.th"
"tax_group_vat_7","VAT 7%","base.th"

```

## File: models\account_move.py

```python
from odoo import models

class AccountMove(models.Model):
    _inherit = "account.move"

    def _get_name_invoice_report(self):
        self.ensure_one()
        if self.company_id.account_fiscal_country_id.code == 'TH':
            return 'l10n_th.report_invoice_document'
        return super()._get_name_invoice_report()

```

## File: models\ir_actions_report.py

```python
from odoo import _, models
from odoo.exceptions import UserError


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _render_qweb_pdf(self, report_ref, res_ids=None, data=None):
        # Check for reports only available for invoices.
        if self._get_report(report_ref).report_name == 'l10n_th.report_commercial_invoice':
            invoices = self.env['account.move'].browse(res_ids)
            if any(not x.is_invoice(include_receipts=True) for x in invoices):
                raise UserError(_("Only invoices could be printed."))

        return super()._render_qweb_pdf(report_ref, res_ids=res_ids, data=data)

```

## File: models\res_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    proxy_type = fields.Selection(selection_add=[('ewallet_id', 'Ewallet ID'),
                                                 ('merchant_tax_id', 'Merchant Tax ID'),
                                                 ('mobile', "Mobile Number")],
                                  ondelete={'ewallet_id': 'set default', 'merchant_tax_id': 'set default', 'mobile': 'set default'})

    @api.constrains('proxy_type', 'proxy_value', 'partner_id')
    def _check_th_proxy(self):
        tax_id_re = re.compile(r'^[0-9]{13}$')
        mobile_re = re.compile(r'^[0-9]{10}$')
        for bank in self.filtered(lambda b: b.country_code == 'TH'):
            if bank.proxy_type not in ['ewallet_id', 'merchant_tax_id', 'mobile', 'none', False]:
                raise ValidationError(_("The QR Code Type must be either Ewallet ID, Merchant Tax ID or Mobile Number to generate a Thailand Bank QR code for account number %s.", bank.acc_number))
            if bank.proxy_type == 'merchant_tax_id' and (not bank.proxy_value or not tax_id_re.match(bank.proxy_value)):
                raise ValidationError(_("The Merchant Tax ID must be in the format 1234567890123 for account number %s.", bank.acc_number))
            if bank.proxy_type == 'mobile' and (not bank.proxy_value or not mobile_re.match(bank.proxy_value)):
                raise ValidationError(_("The Mobile Number must be in the format 0812345678 for account number %s.", bank.acc_number))

    @api.depends('country_code')
    def _compute_display_qr_setting(self):
        bank_th = self.filtered(lambda b: b.country_code == 'TH')
        bank_th.display_qr_setting = self.env.company.qr_code
        super(ResPartnerBank, self - bank_th)._compute_display_qr_setting()

    def _get_merchant_account_info(self):
        if self.country_code == 'TH':
            proxy_type_mapping = {
                'mobile': 1,
                'merchant_tax_id': 2,
                'ewallet_id': 3,
            }
            proxy_value = re.sub(r"^0", "66", self.proxy_value).zfill(13) if self.proxy_type == 'mobile' else self.proxy_value
            vals = [
                (0, 'A000000677010111'),
                (proxy_type_mapping[self.proxy_type], proxy_value),
            ]
            return (29, ''.join([self._serialize(*val) for val in vals]))
        return super()._get_merchant_account_info()

    def _get_error_messages_for_qr(self, qr_method, debtor_partner, currency):
        if qr_method == 'emv_qr' and self.country_code == 'TH':
            if currency.name not in ['THB']:
                return _("Can't generate a PayNow QR code with a currency other than THB.")
            return None

        return super()._get_error_messages_for_qr(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'emv_qr' and self.country_code == 'TH' and self.proxy_type not in ['ewallet_id', 'merchant_tax_id', 'mobile']:
            return _("The PayNow Type must be either Ewallet ID, Merchant Tax ID or Mobile Number to generate a Thailand Bank QR code")

        return super()._check_for_qr_code_errors(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields

class ResPartner(models.Model):
    _inherit = "res.partner"

    l10n_th_branch_name = fields.Char(compute="_l10n_th_get_branch_name")

    def _l10n_th_get_branch_name(self):
        for partner in self:
            if not partner.is_company or partner.country_code != 'TH':
                partner.l10n_th_branch_name = ""
            else:
                code = partner.company_registry
                partner.l10n_th_branch_name = f"Branch {code}" if code else "Headquarter"

```

## File: models\template_th.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('th')
    def _get_th_template_data(self):
        return {
            'property_account_receivable_id': 'a_recv',
            'property_account_payable_id': 'a_pay',
            'property_account_expense_categ_id': 'a_exp_cogs',
            'property_account_income_categ_id': 'a_sales',
        }

    @template('th', 'res.company')
    def _get_th_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.th',
                'bank_account_code_prefix': '1110',
                'cash_account_code_prefix': '1100',
                'transfer_account_code_prefix': '16',
                'account_default_pos_receivable_account_id': 'a_recv_pos',
                'income_currency_exchange_account_id': 'a_income_gain',
                'expense_currency_exchange_account_id': 'a_exp_loss',
                'account_sale_tax_id': 'tax_output_vat',
                'account_purchase_tax_id': 'tax_input_vat',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_th
from . import res_partner
from . import account_move
from . import ir_actions_report
from . import res_bank

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//div[@name='address_not_same_as_shipping']//span[@t-field='o.partner_id.vat']" position="after">
            <span t-esc="o.partner_id.l10n_th_branch_name"/>
        </xpath>

        <xpath expr="//div[@name='address_same_as_shipping']//span[@t-field='o.partner_id.vat']" position="after">
            <span t-esc="o.partner_id.l10n_th_branch_name"/>
        </xpath>
        <xpath expr="//div[@name='no_shipping']//span[@t-field='o.partner_id.vat']" position="after">
            <span t-esc="o.partner_id.l10n_th_branch_name"/>
        </xpath>
        <xpath expr="//h2/span[contains(@t-if, 'posted')]" position="replace">
            <t t-if="o.move_type == 'out_invoice' and o.state == 'posted'">
                <span t-if="o.company_id.account_fiscal_country_id.code == 'TH' and not commercial_invoice">Tax Invoice</span>
                <span t-else="">Invoice</span>
            </t>
        </xpath>
    </template>

    <template id="report_commercial_invoice">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-set="lang" t-value="o.partner_id.lang"/>
                <t t-call="account.report_invoice_document" t-lang="lang"/>
            </t>
        </t>
    </template>

    <record id="action_report_commercial_invoice" model="ir.actions.report">
        <field name="name">Commercial Invoice</field>
        <field name="model">account.move</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">l10n_th.report_commercial_invoice</field>
        <field name="report_file">l10n_th.report_commercial_invoice</field>
        <field name="binding_model_id" ref="account.model_account_move"/>
        <field name="binding_type">report</field>
    </record>

    <!-- Workaround for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_th.report_invoice_document'"
               t-call="l10n_th.report_invoice_document"
               t-lang="lang"/>
        </xpath>
    </template>
</odoo>

```

