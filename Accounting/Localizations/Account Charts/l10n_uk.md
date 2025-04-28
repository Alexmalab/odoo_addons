# Odoo Module: l10n_uk

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
    'name': 'United Kingdom - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['gb'],
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest UK Odoo localisation necessary to run Odoo accounting for UK SME's with:
=================================================================================================
    - a CT600-ready chart of accounts
    - VAT100-ready tax structure
    - InfoLogic UK counties listing
    - a few other adaptations""",
    'author': 'SmartMode LTD',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/united_kingdom.html',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'auto_install': ['account'],
    'data': [
        'data/l10n_uk_chart_data.xml',
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/l10n_uk_demo.xml',
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
        <field name="country_id" ref="base.uk"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_vat_cal" model="account.report.line">
                <field name="name">VAT Calculations</field>
                <field name="aggregation_formula">UKTAX_1.balance + UKTAX_2.balance - UKTAX_4.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_vat_box1" model="account.report.line">
                        <field name="name">[BOX 1] VAT due on sales and other outputs</field>
                        <field name="code">UKTAX_1</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_vat_box1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_vat_box2" model="account.report.line">
                        <field name="name">[BOX 2] VAT due on acquisitions of goods made in Northern Ireland from EU member states</field>
                        <field name="code">UKTAX_2</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_vat_box2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_vat_box3" model="account.report.line">
                        <field name="name">[BOX 3] Total VAT due</field>
                        <field name="aggregation_formula">UKTAX_1.balance + UKTAX_2.balance</field>
                    </record>
                    <record id="account_tax_report_line_vat_box4" model="account.report.line">
                        <field name="name">[BOX 4] VAT reclaimed on purchases and other inputs (including acquisitions from the EU)</field>
                        <field name="code">UKTAX_4</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_vat_box4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_vat_box5" model="account.report.line">
                        <field name="name">[BOX 5] VAT to pay/reclaim</field>
                        <field name="aggregation_formula">UKTAX_1.balance + UKTAX_2.balance - UKTAX_4.balance</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_exd_vat" model="account.report.line">
                <field name="name">Sales and Purchases Excluding VAT</field>
                <field name="aggregation_formula">UKTAX_6.balance + UKTAX_7.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_exd_vat_box6" model="account.report.line">
                        <field name="name">[BOX 6] Total value of sales and all other outputs excluding any VAT</field>
                        <field name="code">UKTAX_6</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_exd_vat_box6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_exd_vat_box7" model="account.report.line">
                        <field name="name">[BOX 7] Total value of purchases and all other inputs excluding VAT</field>
                        <field name="code">UKTAX_7</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_exd_vat_box7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_ec_exd_vat" model="account.report.line">
                <field name="name">EU Sales and Purchases Excluding VAT</field>
                <field name="aggregation_formula">UKTAX_8.balance + UKTAX_9.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_exd_vat_box8" model="account.report.line">
                        <field name="name">[BOX 8] Total value of all supplies of goods and related costs, excluding any VAT, to EU member states</field>
                        <field name="code">UKTAX_8</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_exd_vat_box8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_exd_vat_box9" model="account.report.line">
                        <field name="name">[BOX 9] Total value of all acquisitions of goods and related costs, excluding any VAT, from EU member states</field>
                        <field name="code">UKTAX_9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_exd_vat_box9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">9</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_uk_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_uk_statements_menu" name="United Kingdom" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>

        <!-- Chart template -->
        </odoo>

```

## File: data\template\account.account-uk.csv

```csv
"id","code","name","account_type","reconcile"
"0010","0010","Software","asset_fixed","False"
"0011","0011","Software Depreciation","asset_fixed","False"
"0020","0020","Patents & Trademarks","asset_fixed","False"
"0021","0021","Patents & Trademarks Depreciation","asset_fixed","False"
"0030","0030","Fixtures and fittings","asset_fixed","False"
"0031","0031","Fixtures and fittings Depreciation","asset_fixed","False"
"0040","0040","Land and buildings","asset_fixed","False"
"0041","0041","Land and buildings Depreciation","asset_fixed","False"
"0050","0050","Motor vehicles","asset_fixed","False"
"0051","0051","Motor vehicles Depreciation","asset_fixed","False"
"0060","0060","Office equipment (inc computer equipment)","asset_fixed","False"
"0061","0061","Office equipment (inc computer equipment) Depreciation","asset_fixed","False"
"0070","0070","Plant and machinery","asset_fixed","False"
"0071","0071","Plant and machinery Depreciation","asset_fixed","False"
"1001","1001","Stock","asset_current","True"
"1002","1002","Work in Progress","asset_current","False"
"1003","1003","Finished Goods","asset_current","False"
"1100","1100","Debtors Control Account","asset_receivable","True"
"1101","1101","Sundry Debtors","asset_receivable","True"
"1102","1102","Other Debtors","asset_current","False"
"1104","1104","Debtors Control Account (PoS)","asset_receivable","True"
"1240","1240","Company Credit Card","asset_current","True"
"1103","1103","Prepayments","asset_current","False"
"2100","2100","Creditors Control Account","liability_payable","True"
"2101","2101","Sundry Creditors","liability_current","False"
"2102","2102","Other Creditors","liability_current","False"
"2200","2200","Sales Tax Control Account","liability_current","False"
"2201","2201","Purchase Tax Control Account","asset_current","False"
"2202","2202","HMRC - VAT Account","liability_payable","True"
"2204","2204","Manual Adjustments ﾖ VAT","liability_current","False"
"2210","2210","P.A.Y.E. & NI","liability_payable","True"
"2220","2220","Net Wages","liability_payable","True"
"2230","2230","Pension Fund","liability_payable","True"
"2150","2150","Bad debt provision","liability_current","False"
"2109","2109","Accruals","liability_current","False"
"2320","2320","Corporation Tax","liability_payable","True"
"2300","2300","Loans","liability_current","False"
"2310","2310","Hire Purchase","liability_current","False"
"2330","2330","Mortgages","liability_current","False"
"3000","3000","Called up share capital","equity","False"
"3010","3010","Share premium account","equity","False"
"3020","3020","Revaluation reserve","equity","False"
"3030","3030","Other reserves","equity","False"
"4000","4000","Sales category 1","income","False"
"4001","4001","Sales category 2","income","False"
"4002","4002","Sales category 3","income","False"
"4003","4003","Sales category 4","income","False"
"5000","5000","Cost of sales 1","expense_direct_cost","False"
"5001","5001","Cost of sales 2","expense_direct_cost","False"
"5002","5002","Cost of sales 3","expense_direct_cost","False"
"5003","5003","Cost of sales 4","expense_direct_cost","False"
"6000","6000","Marketing, POS","expense","False"
"6001","6001","Exhibitions and events","expense","False"
"6002","6002","PR","expense","False"
"6010","6010","Distribution vehicles","expense","False"
"6020","6020","Distribution salaries and wages","expense","False"
"6030","6030","Shipping","expense","False"
"7000","7000","Directors pension","expense","False"
"7001","7001","Directors remuneration","expense","False"
"7010","7010","Admin gross salaries","expense","False"
"7011","7011","Management gross salaries","expense","False"
"7012","7012","Employers NIC","expense","False"
"7020","7020","Subcontractors payments","expense","False"
"7610","7610","Consultancy","expense","False"
"7620","7620","Legal and professional charges","expense","False"
"7601","7601","Accounting","expense","False"
"7602","7602","Auditing","expense","False"
"7110","7110","Light, heat and power","expense","False"
"7100","7100","Rent and rates","expense","False"
"7120","7120","Repairs, renewals and maintenance","expense","False"
"7300","7300","Car hire","expense","False"
"7301","7301","Car fuel","expense","False"
"7302","7302","Car maintenance","expense","False"
"7502","7502","Telephone","expense","False"
"7503","7503","Internet & hosting","expense","False"
"7504","7504","Mobiles","expense","False"
"7505","7505","Stationery","expense","False"
"7506","7506","Office consumables","expense","False"
"7507","7507","Postage and Carriage","expense","False"
"7508","7508","Books","expense","False"
"7509","7509","Network costs","expense","False"
"7510","7510","Software expenses","expense","False"
"7511","7511","Other computer costs","expense","False"
"7512","7512","Recruitment fees","expense","False"
"7513","7513","Other admin expenses","expense","False"
"7700","7700","Exchange gains/losses","expense","False"
"7710","7710","Other sundry expenses","expense","False"
"7850","7850","Bad debts","expense","False"
"7910","7910","Bank, credit card and other financial charges","expense","False"
"8000","8000","Intangible assets depn","expense","False"
"8001","8001","Tangible assets depn","expense","False"
"8200","8200","Donations","expense","False"
"8300","8300","Entertaining","expense","False"
"8400","8400","Insurance","expense","False"
"8500","8500","Travel and subsistence","expense","False"
"9000","9000","Profits/Losses on disposals of assets","income","False"
"4900","4900","Bank Interest received","income","False"
"4910","4910","Investment Interest received","income","False"
"7900","7900","Interest paid","expense","False"
"8800","8800","Corporation tax expense","expense","False"

```

## File: data\template\account.fiscal.position-uk.csv

```csv
"id","name","auto_apply","vat_required","sequence","country_id","country_group_id","active","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"account_fiscal_position_domestic_b2c","UK domestic B2C","True","False","1","base.uk","","True","",""
"account_fiscal_position_domestic_b2b","UK domestic B2B","True","True","2","base.uk","","True","",""
"account_fiscal_position_ni_to_eu_b2b","Northern Ireland to European Union member states B2B","True","True","3","","base.europe","False","ST0","ST4"
"","","","","","","","","ST2","ST4"
"","","","","","","","","ST5","ST4"
"","","","","","","","","ST11","ST4"
"","","","","","","","","ST_0_EX","ST4"
"","","","","","","","","PT2","PT7"
"","","","","","","","","PT_0_EX","PT7"
"","","","","","","","","PT_20_G","PT8"
"","","","","","","","","PT_20_PVA","PT8"
"","","","","","","","","PT_20_S","PT_20_RCI"
"account_fiscal_position_rest_of_world_b2b","Rest of the world","True","False","4","","","True","ST0","ST_0_EX"
"","","","","","","","","ST2","ST_0_EX"
"","","","","","","","","ST4","ST_0_EX"
"","","","","","","","","ST5","ST_0_EX"
"","","","","","","","","ST11","ST_0_EX"
"","","","","","","","","PT2","PT_0_EX"
"","","","","","","","","PT7","PT_0_EX"
"","","","","","","","","PT8","PT_0_EX"
"","","","","","","","","PT_20_G","PT_20_PVA"
"","","","","","","","","PT_20_S","PT_20_RCI"

```

## File: data\template\account.tax-uk.csv

```csv
"id","description","invoice_label","type_tax_use","tax_scope","name","amount_type","amount","tax_group_id","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent"
"ST11","","20%","sale","","20%","percent","20.0","tax_group_20","True","base","invoice","+6","",""
"","","","","","","","","","","tax","invoice","+1","2200",""
"","","","","","","","","","","base","refund","-6","",""
"","","","","","","","","","","tax","refund","-1","2200",""
"PT_20_G","","20%","purchase","consu","20% G","percent","20.0","tax_group_20","True","base","invoice","+7","",""
"","","","","","","","","","","tax","invoice","+4","2201",""
"","","","","","","","","","","base","refund","-7","",""
"","","","","","","","","","","tax","refund","-4","2201",""
"PT_20_S","","20%","purchase","service","20% S","percent","20.0","tax_group_20","True","base","invoice","+7","",""
"","","","","","","","","","","tax","invoice","+4","2201",""
"","","","","","","","","","","base","refund","-7","",""
"","","","","","","","","","","tax","refund","-4","2201",""
"ST0","Zero Rated Sales","0%","sale","","0%","percent","0.0","tax_group_0","True","base","invoice","+6","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-6","",""
"","","","","","","","","","","tax","refund","","",""
"ST2","","Exempt","sale","","Exempt","percent","0.0","tax_group_0","True","base","invoice","+6","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-6","",""
"","","","","","","","","","","tax","refund","","",""
"PT2","","Exempt","purchase","","Exempt","percent","0.0","tax_group_0","True","base","invoice","+7","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-7","",""
"","","","","","","","","","","tax","refund","","",""
"PT8","","20%","purchase","","20% EU (NIR)","percent","20.0","tax_group_0","False","base","invoice","+9||+7","",""
"","","","","","","","","","","tax","invoice","+2","2201",""
"","","","","","","","","","","tax","invoice","-4","2201","-100"
"","","","","","","","","","","base","refund","-9||-7","",""
"","","","","","","","","","","tax","refund","-2","2201",""
"","","","","","","","","","","tax","refund","+4","2201","-100"
"PT5","","5%","purchase","","5%","percent","5.0","tax_group_5","True","base","invoice","+7","",""
"","","","","","","","","","","tax","invoice","+4","2201",""
"","","","","","","","","","","base","refund","-7","",""
"","","","","","","","","","","tax","refund","-4","2201",""
"ST5","","5%","sale","","5%","percent","5.0","tax_group_5","True","base","invoice","+6","",""
"","","","","","","","","","","tax","invoice","+1","2200",""
"","","","","","","","","","","base","refund","-6","",""
"","","","","","","","","","","tax","refund","-1","2200",""
"ST4","","0%","sale","","0% EU (NIR)","percent","0.0","tax_group_0","False","base","invoice","+8||+6","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-8||-6","",""
"","","","","","","","","","","tax","refund","","",""
"PT7","","0%","purchase","","0% EU (NIR)","percent","0.0","tax_group_0","False","base","invoice","+9||+7","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-9||-7","",""
"","","","","","","","","","","tax","refund","","",""
"PT_20_RCI","Reverse Charge (International)","20% Reverse Charge","purchase","","20% RC EX","percent","20.0","tax_group_20","True","base","invoice","+6||+7","",""
"","","","","","","","","","","tax","invoice","+1","2201","100"
"","","","","","","","","","","tax","invoice","-4","2200","-100"
"","","","","","","","","","","base","refund","-6||-7","","100"
"","","","","","","","","","","tax","refund","-1","2201","100"
"","","","","","","","","","","tax","refund","+4","2200","-100"
"PT_20_RCD","Reverse Charge (Domestic)","20% Reverse Charge","purchase","","20% RC","percent","20.0","tax_group_20","True","base","invoice","+7","",""
"","","","","","","","","","","tax","invoice","+1","2201","100"
"","","","","","","","","","","tax","invoice","-4","2200","-100"
"","","","","","","","","","","base","refund","-7","","100"
"","","","","","","","","","","tax","refund","-1","2201","100"
"","","","","","","","","","","tax","refund","+4","2200","-100"
"ST_20_RCD","Reverse Charge (Domestic)","Reverse Charge: S55A VATA 94 applies","sale","","0% RC","percent","0","tax_group_0","True","base","invoice","+6","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-6","","100"
"","","","","","","","","","","tax","refund","","",""
"PT_20_PVA","20% Postponed Import VAT","20% PVA","purchase","","20% PVA","percent","20.0","tax_group_20","True","base","invoice","+7","",""
"","","","","","","","","","","tax","invoice","+1","2201",""
"","","","","","","","","","","tax","invoice","-4","2200","-100"
"","","","","","","","","","","base","refund","-7","",""
"","","","","","","","","","","tax","refund","-1","2201",""
"","","","","","","","","","","tax","refund","+4","2200","-100"
"ST_0_EX","","0%","sale","","0% EX","percent","0.0","tax_group_0","True","base","invoice","+6","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-6","",""
"","","","","","","","","","","tax","refund","","",""
"PT_0_EX","","0%","purchase","","0% EX","percent","0.0","tax_group_0","True","base","invoice","+7","",""
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-7","",""
"","","","","","","","","","","tax","refund","","",""

```

## File: data\template\account.tax.group-uk.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","TAX 0%","base.uk","2202","2202"
"tax_group_5","TAX 5%","base.uk","2202","2202"
"tax_group_175","TAX 17.5%","base.uk","2202","2202"
"tax_group_20","TAX 20%","base.uk","2202","2202"

```

## File: migrations\1.1\end-migrate.py

```python
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'uk')], order="parent_path"):
        env['account.chart.template'].try_loading('uk', company)

```

## File: models\template_uk.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('uk')
    def _get_uk_template_data(self):
        return {
            'property_account_receivable_id': '1100',
            'property_account_payable_id': '2100',
            'property_account_expense_categ_id': '5000',
            'property_account_income_categ_id': '4000',
            'code_digits': '6',
        }

    @template('uk', 'res.company')
    def _get_uk_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.uk',
                'bank_account_code_prefix': '1200',
                'cash_account_code_prefix': '1210',
                'transfer_account_code_prefix': '1220',
                'account_default_pos_receivable_account_id': '1104',
                'income_currency_exchange_account_id': '7700',
                'expense_currency_exchange_account_id': '7700',
                'account_sale_tax_id': 'ST11',
                'account_purchase_tax_id': 'PT_20_G',
                'deferred_expense_account_id': '1103',
                'deferred_revenue_account_id': '2109',
            },
        }

    def _post_load_data(self, template_code, company, template_data):
        """If the company is located in Northern Ireland, activate the relevant taxes and fiscal postions."""
        result = super()._post_load_data(template_code, company, template_data)

        is_ni = {
            'base.state_uk18', 'base.state_uk19', 'base.state_uk20', 'base.state_uk21',
            'base.state_uk22', 'base.state_uk23', 'base.state_uk24',
        }.intersection(
            company.state_id._get_external_ids().get(company.state_id.id, [])
        )

        if is_ni:
            for xmlid in ['PT8', 'ST4', 'PT7', 'account_fiscal_position_ni_to_eu_b2b']:
                self.ref(xmlid).active = True

        return result

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_uk

```

