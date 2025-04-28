# Odoo Module: l10n_nz

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
    'name': 'New Zealand - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['nz'],
    'version': '1.2',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
New Zealand Accounting Module
=============================

New Zealand accounting basic charts and localizations.

Also:
    - activates a number of regional currencies.
    - sets up New Zealand taxes.
    """,
    'author': 'Odoo S.A., Richard deMeester - Willow IT',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'depends': [
        'account',
    ],
    'data': [
        'data/account_tax_report_data.xml',
        'data/res_currency_data.xml',
        'views/report_invoice.xml',
        'views/res_company_views.xml',
        'views/res_partner_views.xml',
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
        <field name="name">GST Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.nz"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_sale_and_income" model="account.report.line">
                <field name="name">Sales and Income</field>
                <field name="aggregation_formula">NZBOX5.balance + NZBOX6.balance + NZBOX9.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_box5" model="account.report.line">
                        <field name="name">[BOX 5] Total sales and income for the period(including GST and zero-rated Supplies)</field>
                        <field name="code">NZBOX5</field>
                        <field name="expression_ids">
                            <record id="tax_report_box5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BOX 5</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_box6" model="account.report.line">
                        <field name="name">[BOX 6] Zero-rated supplies in Box 5</field>
                        <field name="code">NZBOX6</field>
                        <field name="expression_ids">
                            <record id="tax_report_box6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BOX 6</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_box7" model="account.report.line">
                        <field name="name">[BOX 7] The amount in Box 6 is subracted from Box 5</field>
                        <field name="code">NZBOX7</field>
                        <field name="aggregation_formula">NZBOX5.balance - NZBOX6.balance</field>
                    </record>
                    <record id="tax_report_box8" model="account.report.line">
                        <field name="name">[BOX 8] Multiply the amount in Box 7 by 3 and then divide by 23</field>
                        <field name="code">NZBOX8</field>
                        <field name="aggregation_formula">(NZBOX7.balance * 3)/23</field>
                    </record>
                    <record id="tax_report_box9" model="account.report.line">
                        <field name="name">[BOX 9] Enter any adjustments from your calculation sheet</field>
                        <field name="code">NZBOX9</field>
                        <field name="expression_ids">
                            <record id="tax_report_box9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BOX 9</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_box10" model="account.report.line">
                        <field name="name">[BOX 10] Total GST collected on sales and income</field>
                        <field name="code">NZBOX10</field>
                        <field name="aggregation_formula">NZBOX8.balance + NZBOX9.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_purchases_and_expenses" model="account.report.line">
                <field name="name">Purchases and Expenses</field>
                <field name="aggregation_formula">NZBOX11.balance + NZBOX13.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_box11" model="account.report.line">
                        <field name="name">[BOX 11] Total purchases and expenses(including GST) for which tax invoicing requirements have been met</field>
                        <field name="code">NZBOX11</field>
                        <field name="expression_ids">
                            <record id="tax_report_box11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">BOX 11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_box12" model="account.report.line">
                        <field name="name">[BOX 12] Multiply BOX11 by 3 and then divide by 23</field>
                        <field name="code">NZBOX12</field>
                        <field name="aggregation_formula">(NZBOX11.balance * 3)/23</field>
                    </record>
                    <record id="tax_report_box13" model="account.report.line">
                        <field name="name">[BOX 13] Credit adjustments from your calculation sheet</field>
                        <field name="code">NZBOX13</field>
                        <field name="expression_ids">
                            <record id="tax_report_box13_aggregation" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">NZBOX13_tags.balance + NZBOX13_manual.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_box13_tags" model="account.report.line">
                                <field name="name">[BOX 13] Amount computed from tax tags</field>
                                <field name="code">NZBOX13_tags</field>
                                <field name="expression_ids">
                                    <record id="tax_report_box13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">BOX 13</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_box13_manual" model="account.report.line">
                                <field name="name">[BOX 13] Manual adjustments</field>
                                <field name="code">NZBOX13_manual</field>
                                <field name="expression_ids">
                                    <record id="tax_report_box13_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_box14" model="account.report.line">
                        <field name="name">[BOX 14] Total GST credit for purchases and expenses</field>
                        <field name="code">NZBOX14</field>
                        <field name="aggregation_formula">NZBOX12.balance + NZBOX13.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_box15" model="account.report.line">
                <field name="name">[BOX 15] Difference between BOX10 and BOX14</field>
                <field name="aggregation_formula">NZBOX10.balance - NZBOX14.balance</field>
                <field name="hierarchy_level">0</field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\res_currency_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.NZD" model="res.currency">
            <field name="active" eval="True"/>
        </record>

        <!-- Common trading partner currencies, and especially for AU -->

        <record id="base.AUD" model="res.currency">
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

## File: data\template\account.account-nz.csv

```csv
"id","code","name","account_type","reconcile"
"nz_11110","11110","Bank","asset_cash","False"
"nz_11130","11130","Petty Cash","asset_cash","False"
"nz_11140","11140","Cash Drawer","asset_cash","False"
"nz_11180","11180","Undeposited Funds","asset_current","True"
"nz_11190","11190","Electronic Clearing","asset_current","True"
"nz_11200","11200","Trade Debtors","asset_receivable","True"
"nz_11210","11210","Less Prov'n for Doubtful Debts","asset_current","False"
"nz_11220","11220","Trade Debtors (PoS)","asset_receivable","True"
"nz_11310","11310","Raw Materials","asset_current","False"
"nz_11320","11320","Finished Goods","asset_current","False"
"nz_11330","11330","Trading Stock on Hand","asset_current","False"
"nz_11340","11340","Goods Shipped not Invoiced","asset_current","False"
"nz_11350","11350","Work In Process (Inventory)","asset_current","False"
"nz_12100","12100","Deposits Paid","asset_prepayments","False"
"nz_12200","12200","Prepaid Insurance","asset_current","False"
"nz_13110","13110","Manufacturing Plant at Cost","asset_fixed","False"
"nz_13120","13120","Manufac. Plant Accum Dep","asset_fixed","False"
"nz_13130","13130","Manufacturing Equipment Cost","asset_fixed","False"
"nz_13140","13140","Manufac. Equip Accum Dep","asset_fixed","False"
"nz_13210","13210","Furniture & Fixtures at Cost","asset_fixed","False"
"nz_13220","13220","Furniture & Fixtures Accum Dep","asset_fixed","False"
"nz_13310","13310","Office Equip at Cost","asset_fixed","False"
"nz_13320","13320","Office Equip Accum Dep","asset_fixed","False"
"nz_13410","13410","Motor Vehicles at Cost","asset_fixed","False"
"nz_13420","13420","Motor Vehicles Accum Dep","asset_fixed","False"
"nz_21110","21110","Credit Card","liability_current","False"
"nz_21200","21200","Trade Creditors","liability_payable","True"
"nz_21210","21210","Goods Received not Billed","liability_current","False"
"nz_21310","21310","GST Collected","liability_current","False"
"nz_21320","21320","GST Payments","liability_current","False"
"nz_21330","21330","GST Paid","asset_current","False"
"nz_21350","21350","Fuel Tax Credits Accrued","liability_current","False"
"nz_21360","21360","Import Duty Payable","liability_current","False"
"nz_21370","21370","Voluntary Withholdings Payable","liability_current","False"
"nz_21380","21380","Tax Withholdings Payable","liability_current","False"
"nz_21410","21410","Payroll Accruals Payable","liability_current","False"
"nz_21420","21420","PAYG Withholding Payable","liability_current","False"
"nz_21600","21600","Customer Deposits","liability_current","False"
"nz_21700","21700","Other Current Liabilities","liability_current","False"
"nz_22100","22100","Mortgages Payable","liability_non_current","False"
"nz_22200","22200","Notes Payable","liability_non_current","False"
"nz_22300","22300","Other Long Term Liabilities","liability_non_current","False"
"nz_31100","31100","Capital Investment","equity","False"
"nz_31200","31200","Capital Drawings","equity","False"
"nz_38000","38000","Retained Earnings","equity","False"
"nz_39000","39000","Current Year Earnings","equity_unaffected","False"
"nz_39999","39999","Historical Balancing","equity","False"
"nz_41110","41110","Sales Product #1","income","False"
"nz_41120","41120","Sales Product #2","income","False"
"nz_41130","41130","Sales Product #3","income","False"
"nz_42000","42000","Wholesale Sales","income","False"
"nz_43000","43000","Consignment Sales","income","False"
"nz_44000","44000","Freight Income","income","False"
"nz_45000","45000","Late Fees Collected","income_other","False"
"nz_46000","46000","Miscellaneous Income","income_other","False"
"nz_47000","47000","Fuel Tax Credits","income_other","False"
"nz_51110","51110","Cost of Goods Sold #1","expense_direct_cost","False"
"nz_51120","51120","Cost of Goods Sold # 2","expense_direct_cost","False"
"nz_51130","51130","Cost of Goods Sold # 3","expense_direct_cost","False"
"nz_52000","52000","Wholesale Cost of Sales","expense_direct_cost","False"
"nz_53000","53000","Consignment Cost of Sales","expense_direct_cost","False"
"nz_54000","54000","Wages for Production Labour","expense_direct_cost","False"
"nz_55000","55000","Materials & Supplies","expense_direct_cost","False"
"nz_56000","56000","Freight","expense_direct_cost","False"
"nz_57000","57000","Other Costs","expense_direct_cost","False"
"nz_61000","61000","Advertising","expense","False"
"nz_61200","61200","Car & Truck Expenses","expense","False"
"nz_61300","61300","Commissions Paid","expense","False"
"nz_61500","61500","Depreciation Expense","expense_depreciation","False"
"nz_61610","61610","Discounts Given","expense","False"
"nz_61620","61620","Discounts Taken","expense","False"
"nz_61630","61630","Exchange Rate Losses (Gains)","expense","False"
"nz_61700","61700","Freight Paid","expense","False"
"nz_61800","61800","Insurance","expense","False"
"nz_61910","61910","Overdraft Interest","expense","False"
"nz_61920","61920","Mortgage Interest","expense","False"
"nz_61930","61930","Other Interest","expense","False"
"nz_62000","62000","Late Fees Paid","expense","False"
"nz_62110","62110","Machinery & Equipment","expense","False"
"nz_62120","62120","Other Business Property","expense","False"
"nz_62200","62200","Legal & Professional Services","expense","False"
"nz_62300","62300","Office Expenses","expense","False"
"nz_62410","62410","Staff Amenities","expense","False"
"nz_62420","62420","Superannuation","expense","False"
"nz_62430","62430","Wages & Salaries","expense","False"
"nz_62440","62440","Workers' Compensation","expense","False"
"nz_62450","62450","Other Employer Expenses","expense","False"
"nz_62500","62500","Repairs","expense","False"
"nz_62550","62550","Shrinkage/Spoilage","expense","False"
"nz_62600","62600","Supplies","expense","False"
"nz_62700","62700","Taxes","expense","False"
"nz_62800","62800","Telephone","expense","False"
"nz_62910","62910","Gas","expense","False"
"nz_62920","62920","Electricity","expense","False"
"nz_62930","62930","Water","expense","False"
"nz_63110","63110","Travel","expense","False"
"nz_63120","63120","Meals & Entertainment","expense","False"
"nz_81000","81000","Interest Income","income_other","False"
"nz_91000","91000","Interest Expense","expense","False"
"nz_92000","92000","Income Tax Expense","expense","False"

```

## File: data\template\account.fiscal.position-nz.csv

```csv
"id","name","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_os_partner","OS Partner","nz_tax_sale_15","nz_tax_sale_0"
"","","nz_tax_sale_inc_15","nz_tax_sale_0"
"","","nz_tax_purchase_15","nz_tax_purchase_0"
"","","nz_tax_purchase_inc_15","nz_tax_purchase_0"

```

## File: data\template\account.tax-nz.csv

```csv
"id","name","sequence","description","invoice_label","type_tax_use","amount_type","amount","price_include","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id"
"nz_tax_sale_15","15%","1","Sale (15%)","GST Sales (15%)","sale","percent","15.0","False","tax_group_gst_15","base","invoice","+BOX 5",""
"","","","","","","","","","","tax","invoice","+BOX 5","nz_21310"
"","","","","","","","","","","base","refund","-BOX 5",""
"","","","","","","","","","","tax","refund","-BOX 5","nz_21310"
"nz_tax_sale_inc_15","15% INC","2","GST Inc Sale (15%)","GST Incl. Sales (15%)","sale","percent","15.0","True","tax_group_gst_15","base","invoice","+BOX 5",""
"","","","","","","","","","","tax","invoice","+BOX 5","nz_21310"
"","","","","","","","","","","base","refund","-BOX 5",""
"","","","","","","","","","","tax","refund","-BOX 5","nz_21310"
"nz_tax_sale_0","0% EX","3","Zero/Export (0%) Sale","Zero Rated (Export) Sales","sale","percent","0.0","False","tax_group_0","base","invoice","+BOX 5||+BOX 6",""
"","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","base","refund","-BOX 5||-BOX 6",""
"","","","","","","","","","","tax","refund","",""
"nz_tax_purchase_15","15%","1","Purch (15%)","GST Purchases (15%)","purchase","percent","15.0","False","tax_group_gst_15","base","invoice","+BOX 11",""
"","","","","","","","","","","tax","invoice","+BOX 11","nz_21330"
"","","","","","","","","","","base","refund","-BOX 11",""
"","","","","","","","","","","tax","refund","-BOX 11","nz_21330"
"nz_tax_purchase_inc_15","15% INC","2","GST Inc Purch (15%)","GST Incl. Purchases (15%)","purchase","percent","15.0","True","tax_group_gst_15","base","invoice","+BOX 11",""
"","","","","","","","","","","tax","invoice","+BOX 11","nz_21330"
"","","","","","","","","","","base","refund","-BOX 11",""
"","","","","","","","","","","tax","refund","-BOX 11","nz_21330"
"nz_tax_purchase_0","0% F","3","Zero/Import (0%) Purch","GST Free Purchases","purchase","percent","0.0","False","tax_group_0","base","invoice","",""
"","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","base","refund","",""
"","","","","","","","","","","tax","refund","",""
"nz_tax_purchase_taxable_import","0% TPS","4","Purch (Imports Taxable)","Purchase (Taxable Imports) - Tax Paid Separately","purchase","percent","0.0","False","tax_group_0","base","invoice","",""
"","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","base","refund","",""
"","","","","","","","","","","tax","refund","",""
"nz_tax_purchase_gst_only","100% ONLY","5","GST Only - Imports","GST Only on Imports","purchase","division","100.0","True","tax_group_100","base","invoice","",""
"","","","","","","","","","","tax","invoice","+BOX 13","nz_21330"
"","","","","","","","","","","base","refund","",""
"","","","","","","","","","","tax","refund","-BOX 13","nz_21330"

```

## File: data\template\account.tax.group-nz.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","GST 0%","base.nz","nz_21320","nz_21320"
"tax_group_gst_15","GST 15%","base.nz","nz_21320","nz_21320"
"tax_group_100","GST 100%","base.nz","nz_21320","nz_21320"

```

## File: migrations\1.2\post-migrate.py

```python
def migrate(cr, version):
    cr.execute("""
        UPDATE account_report_expression e
           SET label = 'balance'
          FROM ir_model_data d
         WHERE e.id = d.res_id
           AND d.module = 'l10n_nz'
           AND d.name = 'tax_report_box13_formula'
           AND e.label = '_upg_1.2balance'
    """)

```

## File: migrations\1.2\pre-migrate.py

```python
def migrate(cr, version):
    cr.execute("""
        UPDATE account_report_expression e
           SET label = '_upg_1.2balance'
          FROM ir_model_data d
         WHERE e.id = d.res_id
           AND d.module = 'l10n_nz'
           AND d.name = 'tax_report_box13_formula'
           AND e.label = 'balance'
    """)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_name_invoice_report(self):
        # Safety mechanism to avoid issues if the module has not yet been updated.
        template = self.env.ref('l10n_nz.report_invoice_document', raise_if_not_found=False)
        if template and self.company_id.account_fiscal_country_id.code == 'NZ':
            return 'l10n_nz.report_invoice_document'
        return super()._get_name_invoice_report()

```

## File: models\template_nz.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('nz')
    def _get_nz_template_data(self):
        return {
            'code_digits': '5',
            'property_account_receivable_id': 'nz_11200',
            'property_account_payable_id': 'nz_21200',
            'property_account_expense_categ_id': 'nz_51110',
            'property_account_income_categ_id': 'nz_41110',
            'property_stock_account_input_categ_id': 'nz_21210',
            'property_stock_account_output_categ_id': 'nz_11340',
            'property_stock_valuation_account_id': 'nz_11330',
            'property_stock_account_production_cost_id': 'nz_11350',
        }

    @template('nz', 'res.company')
    def _get_nz_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.nz',
                'bank_account_code_prefix': '1111',
                'cash_account_code_prefix': '1113',
                'transfer_account_code_prefix': '11170',
                'account_default_pos_receivable_account_id': 'nz_11220',
                'income_currency_exchange_account_id': 'nz_61630',
                'expense_currency_exchange_account_id': 'nz_61630',
                'account_journal_early_pay_discount_loss_account_id': 'nz_61610',
                'account_journal_early_pay_discount_gain_account_id': 'nz_61620',
                'account_sale_tax_id': 'nz_tax_sale_15',
                'account_purchase_tax_id': 'nz_tax_purchase_15',
                'fiscalyear_last_month': '3',
                'fiscalyear_last_day': 31,
                # Changing the opening date to the first day of the fiscal year.
                # This way the opening entries will be set to the 31st of March.
                'account_opening_date': fields.Date.context_today(self).replace(month=4, day=1),
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move
from . import template_nz

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//div[hasclass('page')]/h2" position="replace">
            <h2>
                <span t-if="not proforma"/>
                <span t-else="">PROFORMA</span>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'">Tax Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'draft'">Draft Tax Invoice</span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'cancel'">Cancelled Tax Invoice</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'posted'">Tax Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'draft'">Draft Tax Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'cancel'">Cancelled Tax Credit Note</span>
                <span t-elif="o.move_type == 'in_refund'">Tax Vendor Credit Note</span>
                <span t-elif="o.move_type == 'in_invoice'">Tax Vendor Bill</span>
                <span t-if="o.name != '/'" t-field="o.name">INV/2024/0001</span>
            </h2>
        </xpath>
    </template>

    <!-- Workaround for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_nz.report_invoice_document'"
               t-call="l10n_nz.report_invoice_document"
               t-lang="lang"/>
        </xpath>
    </template>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form_inherit_l10n_nz" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_nz</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="attributes">
                <attribute name="invisible" add="country_code == 'NZ'" separator=" or "/> 
            </xpath>
            <xpath expr="//field[@name='vat']" position="after">
                <field name="vat" string="GST" invisible="country_code != 'NZ'"/>
            </xpath>
            <xpath expr="//field[@name='company_registry']" position="attributes">
                <attribute name="invisible" add="country_code == 'NZ'" separator=" or "/> 
            </xpath>
            <xpath expr="//field[@name='company_registry']" position="after">
                <field name="company_registry" string="NZBN" invisible="country_code != 'NZ'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_nz" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_nz</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='company_registry']" position="attributes">
                <attribute name="nolabel">1</attribute>
            </xpath>
            <field name="company_registry" position="before">
                <label for="company_registry" invisible="country_code == 'NZ'" />
                <label for="company_registry" string="NZBN" invisible="country_code != 'NZ'" />
            </field>
        </field>
    </record>
</odoo>

```

