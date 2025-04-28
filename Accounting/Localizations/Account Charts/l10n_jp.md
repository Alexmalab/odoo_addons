# Odoo Module: l10n_jp

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
    'name': 'Japan - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['jp'],
    'version': '2.3',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """

Overview:
---------

* Chart of Accounts and Taxes template for companies in Japan.
* This probably does not cover all the necessary accounts for a company. You are expected to add/delete/modify accounts based on this template.

Note:
-----

* Fiscal positions '内税' and '外税' have been added to handle special requirements which might arise from POS implementation. [1]  Under normal circumstances, you might not need to use those at all.

[1] See https://github.com/odoo/odoo/pull/6470 for detail.

    """,
    'author': 'Quartile Limited (https://www.quartile.co/)',
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
        <field name="country_id" ref="base.jp"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_jp_tax_report_to_pay" model="account.report.line">
                <field name="name">Total Tax Amount</field>
                <field name="aggregation_formula">GST_SALE_AMOUNT.balance + GST_PURCHASE_AMOUNT.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_jp_tax_report_to_pay_temp_tx" model="account.report.line">
                        <field name="name">GST Sale Amount</field>
                        <field name="code">GST_SALE_AMOUNT</field>
                        <field name="aggregation_formula">GST_SALE_8.balance + GST_SALE_10.balance + TAX_EXEMPT.balance + ZERO_RATED_TAX_SALE.balance</field>
                        <field name="children_ids">
                            <record id="l10n_jp_tax_report_to_pay_temp_tx_output_8" model="account.report.line">
                                <field name="name">GST Sale 8%</field>
                                <field name="code">GST_SALE_8</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_tx_output_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">GST Sale 8%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_to_pay_temp_tx_output_10" model="account.report.line">
                                <field name="name">GST Sale 10%</field>
                                <field name="code">GST_SALE_10</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_tx_output_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">GST Sale 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_to_pay_temp_tx_duty_free" model="account.report.line">
                                <field name="name">Tax Exempt</field>
                                <field name="code">TAX_EXEMPT</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_tx_duty_free_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Tax Exempt</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_to_pay_temp_tx_tax_free" model="account.report.line">
                                <field name="name">Zero-rated Tax</field>
                                <field name="code">ZERO_RATED_TAX_SALE</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_tx_tax_free_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Zero-rated Tax (GST Sale Amount)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_jp_tax_report_to_pay_temp_pmt" model="account.report.line">
                        <field name="name">GST Purchase Amount</field>
                        <field name="code">GST_PURCHASE_AMOUNT</field>
                        <field name="aggregation_formula">GST_PURCHASE_8.balance + GST_PURCHASE_10.balance + IMPORTED.balance + ZERO_RATED_TAX_PURCHASE.balance</field>
                        <field name="children_ids">
                            <record id="l10n_jp_tax_report_to_pay_temp_pmt_susp_cons_8" model="account.report.line">
                                <field name="name">GST Purchase 8%</field>
                                <field name="code">GST_PURCHASE_8</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_pmt_susp_cons_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">GST Purchase 8%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_to_pay_temp_pmt_susp_cons_10" model="account.report.line">
                                <field name="name">GST Purchase 10%</field>
                                <field name="code">GST_PURCHASE_10</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_pmt_susp_cons_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">GST Purchase 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_to_pay_temp_pmt_import_8" model="account.report.line">
                                <field name="name">Imported</field>
                                <field name="code">IMPORTED</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_pmt_import_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Imported</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_to_pay_temp_pmt_tax_free" model="account.report.line">
                                <field name="name">Zero-rated Tax</field>
                                <field name="code">ZERO_RATED_TAX_PURCHASE</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_to_pay_temp_pmt_tax_free_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Zero-rated Tax (GST Purchase Amount)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_jp_tax_report_comp_basis" model="account.report.line">
                <field name="name">Taxable Amount</field>
                <field name="aggregation_formula">SALE_AMOUNT.balance + PURCHASE_AMOUNT.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_jp_tax_report_comp_basis_sales" model="account.report.line">
                        <field name="name">Sale Amount</field>
                        <field name="code">SALE_AMOUNT</field>
                        <field name="aggregation_formula">TAXABLE_SALE_AMOUNT_8.balance + TAXABLE_SALE_AMOUNT_10.balance + TAX_EXEMPTED_SALE_AMOUNT.balance + ZERO_RATED_TAX_AMOUNT.balance</field>
                        <field name="children_ids">
                            <record id="l10n_jp_tax_report_comp_basis_sales_taxable_8" model="account.report.line">
                                <field name="name">Taxable Sale Amount (8%)</field>
                                <field name="code">TAXABLE_SALE_AMOUNT_8</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_sales_taxable_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Taxable Sale Amount (8%)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_comp_basis_sales_taxable_10" model="account.report.line">
                                <field name="name">Taxable Sale Amount (10%)</field>
                                <field name="code">TAXABLE_SALE_AMOUNT_10</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_sales_taxable_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Taxable Sale Amount (10%)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_comp_basis_sales_duty_free" model="account.report.line">
                                <field name="name">Tax Exempted Sale Amount</field>
                                <field name="code">TAX_EXEMPTED_SALE_AMOUNT</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_sales_duty_free_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Tax Exempted Sale Amount</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_comp_basis_sales_tax_free" model="account.report.line">
                                <field name="name">Zero-rated Tax Amount</field>
                                <field name="code">ZERO_RATED_TAX_AMOUNT</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_sales_tax_free_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Zero-rated Tax Amount</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_jp_tax_report_comp_basis_purchases" model="account.report.line">
                        <field name="name">Purchase Amount</field>
                        <field name="code">PURCHASE_AMOUNT</field>
                        <field name="aggregation_formula">TAXABLE_PURCHASE_AMOUNT_8.balance + TAXABLE_PURCHASE_AMOUNT_10.balance + IMPORTED_PURCHASE.balance + ZERO_RATED_PURCHASE.balance</field>
                        <field name="children_ids">
                            <record id="l10n_jp_tax_report_comp_basis_purchases_taxable_8" model="account.report.line">
                                <field name="name">Taxable Purchase Amount (8%)</field>
                                <field name="code">TAXABLE_PURCHASE_AMOUNT_8</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_purchases_taxable_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Taxable Purchase Amount (8%)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_comp_basis_purchases_taxable_10" model="account.report.line">
                                <field name="name">Taxable Purchase Amount (10%)</field>
                                <field name="code">TAXABLE_PURCHASE_AMOUNT_10</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_purchases_taxable_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Taxable Purchase Amount (10%)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_comp_basis_purchases_import" model="account.report.line">
                                <field name="name">Imported Purchase</field>
                                <field name="code">IMPORTED_PURCHASE</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_purchases_import_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Imported Purchase</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_jp_tax_report_comp_basis_purchases_tax_free" model="account.report.line">
                                <field name="name">Zero-rated Purchase</field>
                                <field name="code">ZERO_RATED_PURCHASE</field>
                                <field name="expression_ids">
                                    <record id="l10n_jp_tax_report_comp_basis_purchases_tax_free_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Zero-rated Purchase</field>
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

## File: data\template\account.fiscal.position-jp.csv

```csv
"id","name","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@ja_JP"
"l10n_jp_fiscal_position_tax_exempt","Oversea Customer","l10n_jp_tax_sale_exc_10","l10n_jp_tax_sale_exempt","海外取引先"
"","","l10n_jp_tax_purchase_exc_10","l10n_jp_tax_purchase_imp",""
"","","l10n_jp_tax_sale_exc_8","l10n_jp_tax_sale_exempt",""
"","","l10n_jp_tax_purchase_exc_8","l10n_jp_tax_purchase_imp",""
"l10n_jp_fiscal_position_tax_reduction","Reduced Tax Rate","l10n_jp_tax_sale_exc_10","l10n_jp_tax_sale_exc_8","軽減税率"
"","","l10n_jp_tax_purchase_exc_10","l10n_jp_tax_purchase_exc_8",""

```

## File: data\template\account.tax-jp.csv

```csv
"id","sequence","name","description","invoice_label","amount_type","amount","type_tax_use","active","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@ja_JP","invoice_label@ja_JP"
"l10n_jp_tax_sale_exc_8","1","8%","Sale (8%)","GST Sale 8%","percent","8.0","sale","True","l10n_jp_tax_group_8","base","invoice","+Taxable Sale Amount (8%)","","課税売上8%","課税売上8%"
"","","","","","","","","","","tax","invoice","+GST Sale 8%","l10n_jp_330800","",""
"","","","","","","","","","","base","refund","-Taxable Sale Amount (8%)","","",""
"","","","","","","","","","","tax","refund","-GST Sale 8%","l10n_jp_330800","",""
"l10n_jp_tax_sale_exc_10","1","10%","Sale (10%)","GST Excluded Sale 10%","percent","10.0","sale","True","l10n_jp_tax_group_10","base","invoice","+Taxable Sale Amount (10%)","","課税売上10%","課税売上10%"
"","","","","","","","","","","tax","invoice","+GST Sale 10%","l10n_jp_330800","",""
"","","","","","","","","","","base","refund","-Taxable Sale Amount (10%)","","",""
"","","","","","","","","","","tax","refund","-GST Sale 10%","l10n_jp_330800","",""
"l10n_jp_tax_sale_exempt","1","0% Exempt","GST Tax Exempt Sale","GST Tax Exempt Sale","percent","0.0","sale","True","l10n_jp_tax_group_exempt","base","invoice","-Tax Exempted Sale Amount","","輸出免税","輸出免税"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","+Tax Exempted Sale Amount","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_jp_tax_sale_non_vat","1","0%","Non-VAT","Non-VAT","percent","0.0","sale","True","l10n_jp_tax_group_exempt","base","invoice","-Zero-rated Tax Amount","","対象外売上","対象外売上"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","+Zero-rated Tax Amount","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_jp_tax_purchase_exc_8","1","8%","Purchase (8%)","GST Purchase 8%","percent","8.0","purchase","True","l10n_jp_tax_group_8","base","invoice","+Taxable Purchase Amount (8%)","","課税仕入8%","課税仕入8%"
"","","","","","","","","","","tax","invoice","+GST Purchase 8%","l10n_jp_123200","",""
"","","","","","","","","","","base","refund","-Taxable Purchase Amount (8%)","","",""
"","","","","","","","","","","tax","refund","-GST Purchase 8%","l10n_jp_123200","",""
"l10n_jp_tax_purchase_exc_10","1","10%","Purchase (10%)","GST Purchase 10%","percent","10.0","purchase","True","l10n_jp_tax_group_10","base","invoice","+Taxable Purchase Amount (10%)","","課税仕入10%","課税仕入10%"
"","","","","","","","","","","tax","invoice","+GST Purchase 10%","l10n_jp_123200","",""
"","","","","","","","","","","base","refund","-Taxable Purchase Amount (10%)","","",""
"","","","","","","","","","","tax","refund","-GST Purchase 10%","l10n_jp_123200","",""
"l10n_jp_tax_purchase_imp","1","0% EX","Oversea Purchase","Oversea Purchase","percent","0.0","purchase","True","l10n_jp_tax_group_exempt","base","invoice","+Imported","","輸入仕入","輸入仕入"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-Imported","","",""
"","","","","","","","","","","tax","refund","","","",""
"l10n_jp_tax_purchase_exempt","1","0% Exempt","GST Tax Exempt Purchase","GST Tax Exempt Purchase","percent","0.0","purchase","True","l10n_jp_tax_group_exempt","base","invoice","+Zero-rated Purchase","","非課税仕入","非課税仕入"
"","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","base","refund","-Zero-rated Purchase","","",""
"","","","","","","","","","","tax","refund","","","",""

```

## File: data\template\account.tax.group-jp.csv

```csv
"id","name","country_id","name@ja_JP"
"l10n_jp_tax_group_exempt","Tax Exempt","base.jp","免税"
"l10n_jp_tax_group_8","GST 8%","base.jp","8% 対象"
"l10n_jp_tax_group_10","GST 10%","base.jp","10% 対象"

```

## File: migrations\2.3\pre-migrate.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    if not env.ref('l10n_jp.l10n_jp_tax_group_exempt', raise_if_not_found=False):
        cr.execute("""
            UPDATE ir_model_data
              SET name = 'l10n_jp_tax_group_exempt'
            WHERE name = 'tax_group_0'
              AND module = 'l10n_jp'
        """)
    if not env.ref('l10n_jp.l10n_jp_tax_group_8', raise_if_not_found=False):
        cr.execute("""
            UPDATE ir_model_data
            SET name = 'l10n_jp_tax_group_8'
            WHERE name = 'tax_group_8'
            AND module = 'l10n_jp'
        """)
    if not env.ref('l10n_jp.l10n_jp_tax_group_10', raise_if_not_found=False):
        cr.execute("""
            UPDATE ir_model_data
            SET name = 'l10n_jp_tax_group_10'
            WHERE name = 'tax_group_10'
            AND module = 'l10n_jp'
        """)

```

## File: models\template_jp.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('jp')
    def _get_jp_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'l10n_jp_126000',
            'property_account_payable_id': 'l10n_jp_220000',
            'property_account_expense_categ_id': 'l10n_jp_510000',
            'property_account_income_categ_id': 'l10n_jp_410000',
            'property_stock_valuation_account_id': 'l10n_jp_121100',
            'property_stock_account_input_categ_id': 'l10n_jp_121200',
            'property_stock_account_output_categ_id': 'l10n_jp_121300',
        }

    @template('jp', 'res.company')
    def _get_jp_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.jp',
                'bank_account_code_prefix': '1202',
                'cash_account_code_prefix': '1201',
                'transfer_account_code_prefix': '1236',
                'transfer_account_id': 'l10n_jp_123600',
                'account_default_pos_receivable_account_id': 'l10n_jp_126200',
                'income_currency_exchange_account_id': 'l10n_jp_425700',
                'expense_currency_exchange_account_id': 'l10n_jp_513500',
                'account_journal_suspense_account_id': 'l10n_jp_123900',
                'default_cash_difference_expense_account_id': 'l10n_jp_510100',
                'default_cash_difference_income_account_id': 'l10n_jp_999002',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_jp_510200',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_jp_425000',
                'account_sale_tax_id': 'l10n_jp_tax_sale_exc_10',
                'account_purchase_tax_id': 'l10n_jp_tax_purchase_exc_10',
                'tax_calculation_rounding_method': 'round_globally',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_jp

```

