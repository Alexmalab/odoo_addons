# Odoo Module: l10n_jo

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
    'name': 'Jordan - Accounting',
    'countries': ['jo'],
    'description': """
This is the base module to manage the accounting chart for Jordan in Odoo.
==============================================================================

Jordan accounting basic charts and localization.

Activates:

- Chart of accounts

- Taxes

- Tax report

- Fiscal positions
    """,
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.0',
    'depends': [
        'account',
    ],
    'data': [
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/demo_partner.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_vat_return" model="account.report">
        <field name="name">Tax Report Jordan</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.jo"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_vat_return_base" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">balance</field>
            </record>
            <record id="tax_report_vat_return_tax" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">taxbalance</field>
            </record>
        </field>
        <field name="line_ids">
            <!-- Purchase -->
            <record id="tax_report_vat_purchase_base" model="account.report.line">
                <field name="name">Purchase Tax</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_purchase_base_aggregation" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">JO_STD_PURCHASE_2A.balance + JO_STD_PURCHASE_2B.balance +
                    JO_STD_PURCHASE_2C.balance + JO_STD_PURCHASE_2D.balance + JO_STD_PURCHASE_3A.balance +
                    JO_STD_PURCHASE_3B.balance + JO_STD_PURCHASE_3C.balance + JO_STD_PURCHASE_3D.balance +
                    JO_STD_PURCHASE_4.balance + JO_STD_PURCHASE_5.balance</field>
                    </record>
                    <record id="tax_report_vat_purchase_tax_aggregation" model="account.report.expression">
                        <field name="label">taxbalance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">JO_STD_PURCHASE_2A.taxbalance + JO_STD_PURCHASE_2B.taxbalance +
                    JO_STD_PURCHASE_2C.taxbalance + JO_STD_PURCHASE_2D.taxbalance + JO_STD_PURCHASE_3A.taxbalance +
                    JO_STD_PURCHASE_3B.taxbalance + JO_STD_PURCHASE_3C.taxbalance + JO_STD_PURCHASE_3D.taxbalance +
                    JO_STD_PURCHASE_4.taxbalance + JO_STD_PURCHASE_5.taxbalance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_vat_purchase_sixteen" model="account.report.line">
                        <field name="name">2a. Purchase Tax 16%</field>
                        <field name="code">JO_STD_PURCHASE_2A</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_base_sixteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2a (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_tax_sixteen_tag"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2a (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_ten" model="account.report.line">
                        <field name="name">2b. Purchase Tax 10%</field>
                        <field name="code">JO_STD_PURCHASE_2B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_base_ten_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2b (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_tax_ten_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2b (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_four" model="account.report.line">
                        <field name="name">2c. Purchase Tax 4%</field>
                        <field name="code">JO_STD_PURCHASE_2C</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_base_four_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2c (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_tax_four_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2c (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_other" model="account.report.line">
                        <field name="name">2d. Purchase Other Tax</field>
                        <field name="code">JO_STD_PURCHASE_2D</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_base_other_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2d (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_tax_other_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2d (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_import_sixteen" model="account.report.line">
                        <field name="name">3a. Import Tax 16%</field>
                        <field name="code">JO_STD_PURCHASE_3A</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_import_sixteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3a (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_import_tax_sixteen_tag"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3a (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_import_ten" model="account.report.line">
                        <field name="name">3b. Import Tax 10%</field>
                        <field name="code">JO_STD_PURCHASE_3B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_import_ten_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3b (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_import_tax_ten_tag"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3b (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_import_four" model="account.report.line">
                        <field name="name">3c. Import Tax 4%</field>
                        <field name="code">JO_STD_PURCHASE_3C</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_import_four_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3c (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_import_tax_four_tag"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3c (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_import_other" model="account.report.line">
                        <field name="name">3d. Import Other Tax</field>
                        <field name="code">JO_STD_PURCHASE_3D</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_import_other_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3d (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_import_tax_other_tag"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3d (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_deferred_supply" model="account.report.line">
                        <field name="name">4. Imports Deferred Supply Tax</field>
                        <field name="code">JO_STD_PURCHASE_4</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_deferred_supply_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4 (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_deferred_supply_tax_tag"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4 (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_purchase_exempt" model="account.report.line">
                        <field name="name">5. Exempted Purchases and Imports</field>
                        <field name="code">JO_STD_PURCHASE_5</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_purchase_exempt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5 (Base)</field>
                            </record>
                            <record id="tax_report_vat_purchase_exempt_tax_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5 (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <!-- Sales -->
            <record id="tax_report_vat_sales" model="account.report.line">
                <field name="name">Sales Tax</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_sales_base_aggregation" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">JO_STD_SALES_6A.balance + JO_STD_SALES_6B.balance +
                    JO_STD_SALES_6C.balance + JO_STD_SALES_6D.balance + JO_STD_SALES_7.balance +
                    JO_STD_SALES_8.balance + JO_STD_SALES_9.balance + JO_STD_SALES_10.balance +
                    JO_STD_SALES_11.balance</field>
                    </record>
                    <record id="tax_report_vat_sales_tax_aggregation" model="account.report.expression">
                        <field name="label">taxbalance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">JO_STD_SALES_6A.taxbalance + JO_STD_SALES_6B.taxbalance +
                    JO_STD_SALES_6C.taxbalance + JO_STD_SALES_6D.taxbalance + JO_STD_SALES_7.taxbalance +
                    JO_STD_SALES_8.taxbalance + JO_STD_SALES_9.taxbalance + JO_STD_SALES_10.taxbalance +
                    JO_STD_SALES_11.taxbalance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_vat_sales_sixteen" model="account.report.line">
                        <field name="name">6a. Sales Tax 16%</field>
                        <field name="code">JO_STD_SALES_6A</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sales_base_sixteen_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6a (Base)</field>
                            </record>
                            <record id="tax_report_vat_sales_tax_sixteen_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6a (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_ten" model="account.report.line">
                        <field name="name">6b. Sales Tax 10%</field>
                        <field name="code">JO_STD_SALES_6B</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sales_base_ten_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6b (Base)</field>
                            </record>
                             <record id="tax_report_vat_sales_tax_ten_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6b (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_four" model="account.report.line">
                        <field name="name">6c. Sales Tax 4%</field>
                        <field name="code">JO_STD_SALES_6C</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sales_base_four_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6c (Base)</field>
                            </record>
                            <record id="tax_report_vat_sales_tax_four_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6c (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_other" model="account.report.line">
                        <field name="name">6d. Sales Other Tax</field>
                        <field name="code">JO_STD_SALES_6D</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sale_base_other_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6d (Base)</field>
                            </record>
                            <record id="tax_report_vat_sale_tax_other_tag" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6d (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_zero" model="account.report.line">
                        <field name="name">7. Sales Tax 0%</field>
                        <field name="code">JO_STD_SALES_7</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sale_zero_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7 (Base)</field>
                            </record>
                            <record id="tax_report_vat_sale_zero_tag_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7 (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_export_zero" model="account.report.line">
                        <field name="name">8. Exported Sales</field>
                        <field name="code">JO_STD_SALES_8</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sale_export_zero_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8 (Base)</field>
                            </record>
                            <record id="tax_report_vat_sale_export_zero_tag_tax" model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8 (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_no_tax_zero" model="account.report.line">
                        <field name="name">9. Sales that are not subject to tax</field>
                        <field name="code">JO_STD_SALES_9</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sale_export_no_tax_zero_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9 (Base)</field>
                            </record>
                            <record id="tax_report_vat_sale_export_no_tax_zero_tag_tax"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9 (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_exempt_local_zero" model="account.report.line">
                        <field name="name">10. Exempted Local Sales</field>
                        <field name="code">JO_STD_SALES_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sale_export_exempt_local_zero_tag"
                                    model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 (Base)</field>
                            </record>
                            <record id="tax_report_vat_sale_export_exempt_local_zero_tag_tax"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_sales_no_deductible_zero" model="account.report.line">
                        <field name="name">11. Non-deductible tax</field>
                        <field name="code">JO_STD_SALES_11</field>
                        <field name="expression_ids">
                            <record id="tax_report_vat_sale_export_no_deductible_zero_tag"
                                    model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 (Base)</field>
                            </record>
                            <record id="tax_report_vat_sale_export_no_deductible_zero_tag_tax"
                                    model="account.report.expression">
                                <field name="label">taxbalance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>

            <!-- Totals -->
            <record id="tax_report_vat_return_net" model="account.report.line">
                <field name="name">Net Tax Due (Total)</field>
                <field name="children_ids">
                    <record id="tax_report_vat_payable" model="account.report.line">
                        <field name="name">Total Tax Received - To be Forwarded To Authority</field>
                        <field name="code">JO_TOTAL_PAYABLE</field>
                        <field name="aggregation_formula">JO_STD_SALES_6A.taxbalance + JO_STD_SALES_6B.taxbalance + JO_STD_SALES_6C.taxbalance
                        </field>
                    </record>
                    <record id="tax_report_vat_recoverable" model="account.report.line">
                        <field name="name">Total Tax Paid - Recoverable</field>
                        <field name="code">JO_TOTAL_RECOVERABLE</field>
                        <field name="aggregation_formula">JO_STD_PURCHASE_2A.taxbalance + JO_STD_PURCHASE_2B.taxbalance + JO_STD_PURCHASE_2C.taxbalance +
                        JO_STD_PURCHASE_3A.taxbalance + JO_STD_PURCHASE_3B.taxbalance + JO_STD_PURCHASE_3C.taxbalance
                        </field>
                    </record>
                    <record id="tax_report_vat_net_due" model="account.report.line">
                        <field name="name">Net Tax Due</field>
                        <field name="aggregation_formula">(JO_STD_SALES_6A.taxbalance + JO_STD_SALES_6B.taxbalance + JO_STD_SALES_6C.taxbalance) -
                        (JO_STD_PURCHASE_2A.taxbalance + JO_STD_PURCHASE_2B.taxbalance + JO_STD_PURCHASE_2C.taxbalance + 
                        JO_STD_PURCHASE_3A.taxbalance + JO_STD_PURCHASE_3B.taxbalance + JO_STD_PURCHASE_3C.taxbalance)
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-jo_standard.csv

```csv
"id","name","code","account_type","reconcile","name@ar_001"
"jo_account_100102","Bank Suspense Account","100102","asset_current","False","حساب التعليق البنكي"
"jo_account_100103","Outstanding Receipts","100103","asset_current","False","الإيصالات المستحقة"
"jo_account_100104","Outstanding Payments","100104","asset_current","False","المدفوعات المستحقة"
"jo_account_100106","Credit cards","100106","asset_current","False","البطاقات الائتمانية"
"jo_account_100107","Post Dated Cheques Received","100107","asset_current","False","الشيكات مؤجلة الصرف المستلمة"
"jo_account_100201","Accounts Receivable","100201","asset_receivable","True","الحسابات المدينة"
"jo_account_100202","Accounts Receivable (PoS)","100202","asset_receivable","True","الحسابات المدينة (نقطة البيع)"
"jo_account_100203","Other Receivable","100203","asset_current","False","المستحقات الأخرى"
"jo_account_100301","Deposit - Office Rent","100301","asset_current","False","إيداع - إيجار المكتب"
"jo_account_100302","Deposits - Customs","100302","asset_current","False","الإيداعات - الجمارك"
"jo_account_100303","Deposit to Immigration (Visa)","100303","asset_current","False","إيداع للهجرة (فيزا)"
"jo_account_100304","Deposit Others","100304","asset_current","False","إيداع آخر"
"jo_account_100401","Prepaid Medical Insurance","100401","asset_current","False","التأمين الصحي مسبق الدفع"
"jo_account_100402","Prepaid Life Insurance","100402","asset_current","False","التأمين على الحياة مسبق الدفع"
"jo_account_100403","Prepaid Office Rent","100403","asset_current","False","إيجار المكتب مسبق الدفع"
"jo_account_100404","Prepaid Other Insurance","100404","asset_current","False","التأمينات الأخرى مسبقة الدفع"
"jo_account_100405","Prepaid License Fees","100405","asset_current","False","رسوم الرخصة مسبقة الدفع"
"jo_account_100406","Prepaid Maintenance","100406","asset_current","False","الصيانة مسبقة الدفع"
"jo_account_100407","Prepaid Employees Housing","100407","asset_current","False","سكن الموظفين مسبق الدفع"
"jo_account_100408","Prepaid Schooling Fees","100408","asset_current","False","الرسوم الدراسية مسبقة الدفع"
"jo_account_100409","Prepaid Consultancy Fees","100409","asset_current","False","الرسوم الاستشارية مسبقة الدفع"
"jo_account_100410","Prepaid Legal Fees","100410","asset_current","False","الرسوم القانونية مسبقة الدفع"
"jo_account_100411","Prepaid Sponsorship Fees","100411","asset_current","False","رسوم الكفالة مسبقة الدفع"
"jo_account_100412","Prepaid Advertisement Expenses","100412","asset_current","False","نفقات الإعلان مسبقة الدفع"
"jo_account_100413","Prepaid Bank Guarantee","100413","asset_current","False","ضمان البنك مسبق الدفع"
"jo_account_100414","Prepaid Finance charge for Loans","100414","asset_current","False","رسوم التمويل مسبقة الدفع للقروض"
"jo_account_100415","Other Prepayments","100415","asset_current","False","المدفوعات المسبقة الأخرى"
"jo_account_100416","Prepaid Expenses","100416","asset_current","False","المصروفات المدفوعة مقدما"
"jo_account_100501","Handling Difference in Inventory","100501","asset_current","False","التعامل مع الفرق في المخزون"
"jo_account_100502","Inventory Valuation","100502","asset_current","False","تقييم المخزون"
"jo_account_100503","Stock Incoming","100503","asset_current","False","الأسهم الواردة"
"jo_account_100504","Stock Outgoing","100504","asset_current","False","الأسهم الصادرة"
"jo_account_100505","Work in Progress (Inventory)","100505","asset_current","False","العمل قيد التنفيذ (المخزون)"
"jo_account_100601","Accumulated Depreciation of Motor Vehicles","100601","asset_fixed","False","حساب الإهلاك للمركبات"
"jo_account_100602","Amortisation on Leasehold Improvement","100602","asset_fixed","False","الاستهلاك عند تحسين العقارات المستأجرة"
"jo_account_100603","Leasehold Improvement","100603","asset_fixed","False","تحسين العقارات المستأجرة"
"jo_account_100604","Furniture and Equipment","100604","asset_fixed","False","الأثاث والمعدات"
"jo_account_100605","Computer Hardware & Software","100605","asset_fixed","False","أجهزة وبرامج الحاسوب"
"jo_account_100606","Accumulated Depreciation of Furniture & Office Equipment","100606","asset_fixed","False","حساب الإهلاك للأثاث والأدوات المكتبية"
"jo_account_100607","Accumulated Depreciation of Computer Hardware & Software","100607","asset_fixed","False","حساب الإهلاك لبرامج وأجهزة الحاسوب"
"jo_account_100701","Registration of Trademarks","100701","asset_current","False","تسجيل العلامات التجارية"
"jo_account_100801","Right of use Asset (IFRS 16)","100801","asset_fixed","False","حق استخدام الأصل (IFRS 16)"
"jo_account_100802","Accumulated Depreciation Right of use Asset (IFRS 16)","100802","asset_fixed","False","حق استخدام الأصل للإهلاك المتراكم (IFRS 16)"
"jo_account_200101","Payables","200101","liability_payable","True","المبالغ مستحقة الدفع"
"jo_account_200102","Trade Payables","200102","liability_payable","True","الذمم التجارية الدائنة"
"jo_account_200103","Employees Payables","200103","liability_payable","True","المبالغ المستحقة للموظفين"
"jo_account_200104","Credit Notes to Customers","200104","liability_current","False","الإشعارات الدائنة للعملاء"
"jo_account_200201","Accrued - Salaries","200201","liability_current","False","مستحق - المرتبات"
"jo_account_200202","Accrued - Commissions","200202","liability_current","False","مستحق - العمولات"
"jo_account_200203","Accrued - Staff Bonus","200203","liability_current","False","مكافآت الموظفين المستحقة"
"jo_account_200204","Accrued Other Personnel Cost","200204","liability_current","False","تكاليف الموظفين الآخرين المستحقة"
"jo_account_200205","Accrued - Sponsorship","200205","liability_current","False","مستحق - الكفالة"
"jo_account_200301","Accrued - Utilities","200301","liability_current","False","مستحق - المرافق"
"jo_account_200302","Accrued - Telephone","200302","liability_current","False","مستحق - الهاتف"
"jo_account_200303","Accrued - Audit Fees","200303","liability_current","False","مستحق - رسوم التدقيق"
"jo_account_200304","Accrued - Office Rent","200304","liability_current","False","مستحق - إيجار المكتب"
"jo_account_200305","Accrued Others","200305","liability_current","False","المستحقات الأخرى"
"jo_account_200306","Accrued Jordan Customs","200306","liability_current","False","جمارك الأردن المستحقة"
"jo_account_200401","Deferred income","200401","liability_current","False","الدخل المؤجل"
"jo_account_200501","Leave Tickets Provision","200501","liability_non_current","False","حكم تذاكر الطيران"
"jo_account_200502","Leave Days Provision","200502","liability_non_current","False","حكم أيام الإجازة"
"jo_account_200503","End of Service Provision","200503","liability_non_current","False","حكم نهاية الخدمة"
"jo_account_200504","Income Tax Provision","200504","liability_non_current","False","حكم ضريبة الدخل"
"jo_account_200901","VAT Input","200901","asset_current","False","مدخلات ضريبة القيمة المضافة"
"jo_account_200902","VAT Output","200902","liability_current","False","مخرجات ضريبة القيمة المضافة"
"jo_account_200903","VAT Receivable","200903","asset_non_current","False","ضريبة القيمة المضافة مستحقة الدفع"
"jo_account_200904","VAT Payable","200904","liability_non_current","False","ضريبة القيمة المضافة المستحقة"
"jo_account_200905","Tax Payable","200905","liability_current","False","الضريبة المستحقة"
"jo_account_200906","Tax Receivable","200906","asset_current","False","ضريبة مستحقة القبض"
"jo_account_300100","Retained Earnings","300100","equity","False","الأرباح المستبقاة"
"jo_account_300101","Undistributed Profits/Losses","300101","equity_unaffected","False","الأرباح/الخسائر غير الموزعة"
"jo_account_400100","Income Clearing Account","400100","income","False","حساب مقاصة الدخل"
"jo_account_400101","Sales Account","400101","income","False","حساب المبيعات"
"jo_account_400102","Sales of I/C","400102","income","False","المبيعات بين الشركات التابعة"
"jo_account_400103","Sales from Other Region","400103","income","False","المبيعات من منطقة أخرى"
"jo_account_400104","Management Consultancy Fees","400104","income","False","رسوم الاستشارة الإدارية"
"jo_account_400105","Advertising Income","400105","income","False","دخل الإعلان"
"jo_account_400201","Other Income","400201","income_other","False","دخل آخر"
"jo_account_400301","Gain on Difference on Exchange","400301","income_other","False","أرباح فرق صرف العملة"
"jo_account_400302","Cash Difference Gain","400302","income_other","False","أرباح فرق النقد"
"jo_account_400303","Excess In Till","400303","income_other","False","الفائض في صندوق النقود"
"jo_account_400304","Cash Discount Gain","400304","income_other","False","مكاسب الخصم النقدي"
"jo_account_500101","Cost of Goods Sold in Trading","500101","expense_direct_cost","False","تكاليف البضائع المباعة في التجارة"
"jo_account_500102","Cost Of Goods Sold I/C Sales","500102","expense_direct_cost","False","تكاليف البضائع المباعة - المبيعات بين الشركات التابعة"
"jo_account_500200","Expense Clearing Account","500200","expense","False","حساب مقاصة النفقات"
"jo_account_500201","Medical Insurance","500201","expense","False","التأمين الصحي"
"jo_account_500202","End of Service Indemnity","500202","expense","False","تعويض نهاية الخدمة"
"jo_account_500203","Sponsorship Fees","500203","expense","False","رسوم الكفالة"
"jo_account_500301","Basic Salary","500301","expense","False","الراتب الأساسي"
"jo_account_500302","Housing Allowance","500302","expense","False","بدل السكن"
"jo_account_500303","Transportation Allowance","500303","expense","False","بدل المواصلات"
"jo_account_500304","Leave Ticket","500304","expense","False","تذكرة الطيران"
"jo_account_500305","Leave Salary","500305","expense","False","راتب الإجازة"
"jo_account_500306","Sales Commission","500306","expense","False","عمولة المبيعات"
"jo_account_500307","Visa Expenses","500307","expense","False","نفقات الفيزا"
"jo_account_500308","Staff Other Allowances","500308","expense","False","نفقات الموظفين الأخرى"
"jo_account_500309","Air tickets","500309","expense","False","تذاكر الطيران"
"jo_account_500401","Office Rent","500401","expense","False","إيجار المكتب"
"jo_account_500402","Warehouse Rent","500402","expense","False","إيجار المستودع"
"jo_account_500403","Water & Electricity","500403","expense","False","الماء والكهرباء"
"jo_account_500404","Other Utility Charges","500404","expense","False","رسوم المرافق الأخرى"
"jo_account_500501","Audit Fees","500501","expense","False","رسوم التدقيق"
"jo_account_500502","Legal fees","500502","expense","False","الرسوم القانونية"
"jo_account_500503","Trade License Fees","500503","expense","False","رسوم الرخصة التجارية"
"jo_account_500504","Others - Professional Fees","500504","expense","False","غير ذلك - الرسوم المهنية"
"jo_account_500505","Insurance","500505","expense","False","التأمين"
"jo_account_500506","Previous Year Adjustments Account","500506","expense","False","حساب تعديلات العام الماضي"
"jo_account_500601","Credit Card Charges","500601","expense","False","رسوم البطاقة الائتمانية"
"jo_account_500602","Other Bank Charges","500602","expense","False","الرسوم البنكية الأخرى"
"jo_account_500603","Bank Finance & Loan Charges","500603","expense","False","رسوم القروض والتمويل البنكي"
"jo_account_500651","Income Tax Expense","500651","expense","False","نفقات ضريبة الدخل"
"jo_account_500701","Other - Advertising Expenses","500701","expense","False","غير ذلك - نفقات الإعلان"
"jo_account_500702","Training","500702","expense","False","التدريب"
"jo_account_500703","Consultancy Fees","500703","expense","False","الرسوم الاستشارية"
"jo_account_500801","Amortisation on Leasehold Improvement","500801","expense_depreciation","False","الاستهلاك عند تحسين العقارات المستأجرة"
"jo_account_500802","Vehicle Expenses","500802","expense_depreciation","False","نفقات المركبات"
"jo_account_500803","Depreciation of Motor Vehicles","500803","expense_depreciation","False","إهلاك المركبات"
"jo_account_500804","Depreciation of Furniture & Office Equipment","500804","expense_depreciation","False","إهلاك الأثاث والمعدات المكتبية"
"jo_account_500805","Depreciation of Computer Hard & Soft","500805","expense_depreciation","False","إهلاك أجهزة وبرامج الحاسوب"
"jo_account_500851","Depreciation on Right of use Asset (IFRS 16)","500851","expense_depreciation","False","إهلاك حق استخدام الأصل (IFRS 16)"
"jo_account_500901","Loss on Fixed Assets Disposal","500901","expense","False","خسائر التصرف في الأصول الثابتة"
"jo_account_500902","Cash Shortage","500902","expense","False","القصور النقدي"
"jo_account_500903","Loss on Difference on Exchange","500903","expense","False","حسائر فرق صرف العملة"
"jo_account_500904","Write Off Receivables & Payables","500904","expense","False","شطب الحسابات المدينة والدائنة"
"jo_account_500905","Write Off Inventory","500905","expense","False","شطب المخزون"
"jo_account_500906","Others - Provision & Write Off","500906","expense","False","غير ذلك - المَحافظ والتعديلات"
"jo_account_500907","Others","500907","expense","False","غير ذلك"
"jo_account_500908","Other Non-Operating Expenses","500908","expense","False","النفقات الأخرى غير التشغيلية"
"jo_account_500909","Cash Difference Loss","500909","expense","False","خسائر فريق النقد"
"jo_account_501101","Telephone","501101","expense","False","الهاتف"
"jo_account_501102","Others - Communication","501102","expense","False","غير ذلك - التواصل"
"jo_account_501103","Maintenance","501103","expense","False","الصيانة"
"jo_account_501104","Security & Guard","501104","expense","False","الأمن والحراسة"
"jo_account_501105","Cleaning","501105","expense","False","التنظيف"
"jo_account_501106","Others - Office Various Expenses","501106","expense","False","غير ذلك - نفقات المكتب المختلفة"
"jo_account_501107","Cash Discount Loss","501107","expense","False","خسارة الخصم النقدي"

```

## File: data\template\account.fiscal.position-jo_standard.csv

```csv
"id","name","sequence","auto_apply","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@ar_001"
"account_fiscal_position_jordan","Jordan","19","1","base.jo","","","الأردن"
"account_fiscal_position_non_jordan","Non-Jordan","20","1","","jo_standard_sale_16","jo_zero_sale_export","غير الأردن"
"","","","","","jo_standard_purchase_16","jo_standard_purchase_import_other",""

```

## File: data\template\account.group-jo_standard.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@ar_001"
"jo_group_01","1000","1000","Liquidity","السيولة"
"jo_group_02","1001","1001","Liquidity","السيولة"
"jo_group_03","1009","1009","Liquidity","السيولة"
"jo_group_04","1002","1002","Receivables","الحسابات المدينة"
"jo_group_05","1003","1003","Deposits","الإيداعات"
"jo_group_06","1004","1004","Prepaid Expenses","النفقات مسبقة الدفع"
"jo_group_07","1005","1005","Inventory","المخزون"
"jo_group_08","1006","1006","Company Assets & Depreciation","أصول الشركة والإهلاك"
"jo_group_09","1007","1007","Licensing and Copyrights","الترخيص وحقوق النشر"
"jo_group_10","1008","1008","IFRS16 Assets & Depreciation","أصول IFRS16 والإهلاك"
"jo_group_11","2001","2001","Payables","الحسابات الدائنة"
"jo_group_12","2002","2002","Accrued Employee Expenses","نفقات الموظفين المستحقة"
"jo_group_13","2003","2003","Accrued Expenses","النفقات المستحقة"
"jo_group_14","2004","2004","Deferrals","التأجيلات"
"jo_group_15","2005","2005","Provisions","الأحكام"
"jo_group_16","2009","2009"," VAT","ضريبة القيمة المضافة"
"jo_group_17","4001","4001","Operating Income","الإيرادات التشغيلية"
"jo_group_18","4002","4002","Non-Operating Income","الإيرادات غير التشغيلية"
"jo_group_19","4003","4003","Other gains & losses - Other Income","المكاسب والخسائر الأخرى - إيرادات أخرى"
"jo_group_20","5001","5001","Cost of Sales","تكلفة المبيعات"
"jo_group_21","5002","5002","Employees Expenses","مصروفات الموظفين"
"jo_group_22","5003","5003","Payroll Expenses","مصروفات الرواتب"
"jo_group_23","5004","5004","Office and Location Expenses","نفقات المكتب والموقع"
"jo_group_24","5005","5005","Company Expenses","مصروفات الشركة"
"jo_group_25","500600","500649","Finance Expenses","المصروفات المالية"
"jo_group_26","500650","500699","Income Tax","ضريبة الدخل"
"jo_group_27","5007","5007","Misc. Company Expenses","متفرقات. مصروفات الشركة"
"jo_group_28","500800","500849","Assets Depreciation Expenses","مصروفات استهلاك الأصول"
"jo_group_29","500851","500899","IFRS16 Depreciation","IFRS16 الإهلاك"
"jo_group_30","5009","5009","Other gains & losses - Expenses","الأرباح والخسائر الأخرى - المصروفات"
"jo_group_31","5011","5011","Misc. Office Expenses","متفرقات. نفقات مكتبية"

```

## File: data\template\account.tax-jo_standard.csv

```csv
"id","name","type_tax_use","amount","amount_type","description","invoice_label","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@ar_001"
"jo_standard_purchase_16","16%","purchase","16.0","percent","","16%","jo_tax_group_16","base","invoice","+2a (Base)","",""
"","","","","","","","","tax","invoice","+2a (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-2a (Base)","",""
"","","","","","","","","tax","refund","-2a (Tax)","jo_account_200901",""
"jo_standard_purchase_10","10%","purchase","10.0","percent","","10%","jo_tax_group_10","base","invoice","+2b (Base)","",""
"","","","","","","","","tax","invoice","+2b (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-2b (Base)","",""
"","","","","","","","","tax","refund","-2b (Tax)","jo_account_200901",""
"jo_standard_purchase_4","4%","purchase","4.0","percent","","4%","jo_tax_group_4","base","invoice","+2c (Base)","",""
"","","","","","","","","tax","invoice","+2c (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-2c (Base)","",""
"","","","","","","","","tax","refund","-2c (Tax)","jo_account_200901",""
"jo_standard_purchase_other","0% O","purchase","0.0","percent","Other","0% Other","jo_tax_group_0","base","invoice","+2d (Base)","","أخرى"
"","","","","","","","","tax","invoice","+2d (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-2d (Base)","",""
"","","","","","","","","tax","refund","-2d (Tax)","jo_account_200901",""
"jo_standard_purchase_import_16","16% EX","purchase","16.0","percent","Import","16% Import","jo_tax_group_16","base","invoice","+3a (Base)","","إستيراد"
"","","","","","","","","tax","invoice","+3a (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-3a (Base)","",""
"","","","","","","","","tax","refund","-3a (Tax)","jo_account_200901",""
"jo_standard_purchase_import_10","10% EX","purchase","10.0","percent","Import","10% Import","jo_tax_group_10","base","invoice","+3b (Base)","","إستيراد"
"","","","","","","","","tax","invoice","+3b (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-3b (Base)","",""
"","","","","","","","","tax","refund","-3b (Tax)","jo_account_200901",""
"jo_standard_purchase_import_4","4%  EX","purchase","4.0","percent","Import","4% Import","jo_tax_group_4","base","invoice","+3c (Base)","","إستيراد"
"","","","","","","","","tax","invoice","+3c (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-3c (Base)","",""
"","","","","","","","","tax","refund","-3c (Tax)","jo_account_200901",""
"jo_standard_purchase_import_other","0% EX O","purchase","0.0","percent","Import Other","0% Import Other","jo_tax_group_0","base","invoice","+3d (Base)","","إستيراد أخرى"
"","","","","","","","","tax","invoice","+3d (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-3d (Base)","",""
"","","","","","","","","tax","refund","-3d (Tax)","jo_account_200901",""
"jo_standard_purchase_import_deferred","0% EX DS","purchase","0.0","percent","Imports Deferred Supply","0% Imports Deferred Supply","jo_tax_group_0","base","invoice","+4 (Base)","","الواردات المؤجلة"
"","","","","","","","","tax","invoice","+4 (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-4 (Base)","",""
"","","","","","","","","tax","refund","-4 (Tax)","jo_account_200901",""
"eg_zero_purchase_exempted","0% EXT","purchase","0.0","percent","Exempt","Exempt","jo_tax_group_exempt","base","invoice","+5 (Base)","","معفاة"
"","","","","","","","","tax","invoice","+5 (Tax)","jo_account_200901",""
"","","","","","","","","base","refund","-5 (Base)","",""
"","","","","","","","","tax","refund","-5 (Tax)","jo_account_200901",""
"jo_standard_sale_16","16%","sale","16.0","percent","","16%","jo_tax_group_16","base","invoice","+6a (Base)","",""
"","","","","","","","","tax","invoice","+6a (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-6a (Base)","",""
"","","","","","","","","tax","refund","-6a (Tax)","jo_account_200902",""
"jo_standard_sale_10","10%","sale","10.0","percent","","10%","jo_tax_group_10","base","invoice","+6b (Base)","",""
"","","","","","","","","tax","invoice","+6b (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-6b (Base)","",""
"","","","","","","","","tax","refund","-6b (Tax)","jo_account_200902",""
"jo_standard_sale_4","4%","sale","4.0","percent","","4%","jo_tax_group_4","base","invoice","+6c (Base)","",""
"","","","","","","","","tax","invoice","+6c (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-6c (Base)","",""
"","","","","","","","","tax","refund","-6c (Tax)","jo_account_200902",""
"jo_standard_sale_other","0% O","sale","0.0","percent","Other","0% Other","jo_tax_group_0","base","invoice","+6d (Base)","","أخرى"
"","","","","","","","","tax","invoice","+6d (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-6d (Base)","",""
"","","","","","","","","tax","refund","-6d (Tax)","jo_account_200902",""
"jo_zero_sale_0","0%","sale","0.0","percent","","0%","jo_tax_group_0","base","invoice","+7 (Base)","",""
"","","","","","","","","tax","invoice","+7 (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-7 (Base)","",""
"","","","","","","","","tax","refund","-7 (Tax)","jo_account_200902",""
"jo_zero_sale_export","0% EX","sale","0.0","percent","Export","0% Export","jo_tax_group_0","base","invoice","+8 (Base)","","تصدير"
"","","","","","","","","tax","invoice","+8 (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-8 (Base)","",""
"","","","","","","","","tax","refund","-8 (Tax)","jo_account_200902",""
"jo_zero_sale_no_tax","0% SNT","sale","0.0","percent","Not Subject To Tax","0% Not Subject To Tax","jo_tax_group_0","base","invoice","+9 (Base)","","غير خاضع للضريبة"
"","","","","","","","","tax","invoice","+9 (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-9 (Base)","",""
"","","","","","","","","tax","refund","-9 (Tax)","jo_account_200902",""
"jo_zero_sale_exempted","0% EXT LS","sale","0.0","percent","Exempt Local","Exempt Local","jo_tax_group_exempt","base","invoice","+10 (Base)","","المبيعات المحلية المعفاة من الضريبة"
"","","","","","","","","tax","invoice","+10 (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-10 (Base)","",""
"","","","","","","","","tax","refund","-10 (Tax)","jo_account_200902",""
"jo_zero_sale_non_deductible","0% NDT","sale","0.0","percent","Non-Deductible Tax (For Exempted)","Non-Deductible Tax","jo_tax_group_0","base","invoice","+11 (Base)","","الضريبة غير القابلة للخصم (للمبيعات المعفاة)"
"","","","","","","","","tax","invoice","+11 (Tax)","jo_account_200902",""
"","","","","","","","","base","refund","-11 (Base)","",""
"","","","","","","","","tax","refund","-11 (Tax)","jo_account_200902",""

```

## File: data\template\account.tax.group-jo_standard.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@ar_001"
"jo_tax_group_16","Tax 16%","base.jo","jo_account_200906","jo_account_200905","10% ضريبة"
"jo_tax_group_10","Tax 10%","base.jo","jo_account_200906","jo_account_200905","10% ضريبة"
"jo_tax_group_4","Tax 4%","base.jo","jo_account_200906","jo_account_200905","4% ضريبة"
"jo_tax_group_0","Tax 0%","base.jo","jo_account_200906","jo_account_200905","0% ضريبة"
"jo_tax_group_exempt","Tax Exempt","base.jo","jo_account_200906","jo_account_200905","إعفاء ضريبي"

```

## File: i18n_extra\l10n_jo.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_jo
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 16.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2023-03-28 07:33+0000\n"
"PO-Revision-Date: 2023-03-28 07:33+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_exempt_local_zero
msgid "10. Exempted Local Sales (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_exempt_local_zero_tax
msgid "10. Exempted Local Sales (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_no_deductible_zero
msgid "11. Non-deductible tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_no_deductible_zero_tax
msgid "11. Non-deductible tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_sixteen
msgid "2a. Purchase Tax 16% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_sixteen_tax
msgid "2a. Purchase Tax 16% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_ten
msgid "2b. Purchase Tax 10% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_ten_tax
msgid "2b. Purchase Tax 10% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_four
msgid "2c. Purchase Tax 4% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_four_tax
msgid "2c. Purchase Tax 4% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_other
msgid "2d. Purchase Other Tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_other_tax
msgid "2d. Purchase Other Tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_sixteen
msgid "3a. Import Tax 16% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_sixteen_tax
msgid "3a. Import Tax 16% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_ten
msgid "3b. Import Tax 10% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_ten_tax
msgid "3b. Import Tax 10% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_four
msgid "3c. Import Tax 4% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_four_tax
msgid "3c. Import Tax 4% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_other
msgid "3d. Import Other Tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_import_other_tax
msgid "3d. Import Other Tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_deferred_supply
msgid "4. Imports Deferred Supply Tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_deferred_supply_tax
msgid "4. Imports Deferred Supply Tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_exempt
msgid "5. Exempted Purchases and Imports (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_exempt_tax
msgid "5. Exempted Purchases and Imports (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_sixteen
msgid "6a. Sales Tax 16% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_sixteen_tax
msgid "6a. Sales Tax 16% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_ten
msgid "6b. Sales Tax 10% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_ten_tax
msgid "6b. Sales Tax 10% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_four
msgid "6c. Sales Tax 4% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_four_tax
msgid "6c. Sales Tax 4% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_other
msgid "6d. Sales Other Tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_other_tax
msgid "6d. Sales Other Tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_zero
msgid "7. Sales Tax 0% (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_zero_tax
msgid "7. Sales Tax 0% (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_export_zero
msgid "8. Exported Sales (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_export_zero_tax
msgid "8. Exported Sales (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_no_tax_zero
msgid "9. Sales that are not subject to tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_no_tax_zero_tax
msgid "9. Sales that are not subject to tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:ir.model,name:l10n_jo.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_jo
#: model:account.report.column,name:l10n_jo.tax_report_vat_return_balance
msgid "Balance"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_net_due
msgid "Net Tax Due"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_return_net
msgid "Net Tax Due (Total)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_base
msgid "Purchase Tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_purchase_tax
msgid "Purchase Tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_base
msgid "Sales Tax (Base)"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_sales_tax
msgid "Sales Tax (Tax)"
msgstr ""

#. module: l10n_jo
#: model:account.report,name:l10n_jo.tax_report_vat_return
msgid "Tax Report Jordan"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_recoverable
msgid "Total Tax Paid - Recoverable"
msgstr ""

#. module: l10n_jo
#: model:account.report.line,name:l10n_jo.tax_report_vat_payable
msgid "Total Tax Received - To be Forwarded To Authority"
msgstr ""

```

## File: models\template_jo_standard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('jo_standard')
    def _get_jo_standard_template_data(self):
        return {
            'property_account_receivable_id': 'jo_account_100201',
            'property_account_payable_id': 'jo_account_200101',
            'property_account_expense_categ_id': 'jo_account_500101',
            'property_account_income_categ_id': 'jo_account_400101',
            'property_account_expense_id': 'jo_account_500101',
            'property_account_income_id': 'jo_account_400101',
            'property_stock_valuation_account_id': 'jo_account_100502',
            'property_stock_account_input_categ_id': 'jo_account_100503',
            'property_stock_account_output_categ_id': 'jo_account_100504',
            'property_stock_account_production_cost_id': 'jo_account_100505',
            'code_digits': '6',
        }

    @template('jo_standard', 'res.company')
    def _get_jo_standard_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.jo',
                'bank_account_code_prefix': '1000',
                'cash_account_code_prefix': '1009',
                'transfer_account_code_prefix': '1001',
                'account_default_pos_receivable_account_id': 'jo_account_100202',
                'income_currency_exchange_account_id': 'jo_account_400301',
                'expense_currency_exchange_account_id': 'jo_account_500903',
                'account_journal_suspense_account_id': 'jo_account_100102',
                'account_journal_early_pay_discount_loss_account_id': 'jo_account_501107',
                'account_journal_early_pay_discount_gain_account_id': 'jo_account_400304',
                'account_journal_payment_debit_account_id': 'jo_account_100103',
                'account_journal_payment_credit_account_id': 'jo_account_100104',
                'default_cash_difference_income_account_id': 'jo_account_400302',
                'default_cash_difference_expense_account_id': 'jo_account_500909',
                'deferred_expense_account_id': 'jo_account_100416',
                'deferred_revenue_account_id': 'jo_account_200401',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_jo_standard

```

