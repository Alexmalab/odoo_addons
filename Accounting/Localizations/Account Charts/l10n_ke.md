# Odoo Module: l10n_ke

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Kenya - Accounting',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This provides a base chart of accounts and taxes template for use in Odoo.
    """,
    'author': 'Odoo S.A.',
    'depends': [
        'account',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/l10n_ke_chart_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template.xml',
        'data/account_chart_template_configure_data.xml',
        'data/menu_item_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml'
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id:id","chart_template_id:id","tag_ids/id","reconcile"
"ke0010","Software","0010","account.data_account_type_non_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke0020","Patents & Trademarks","0020","account.data_account_type_non_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke0030","Fixtures and fittings","0030","account.data_account_type_fixed_assets","l10n_ke.l10nke_chart_template","","False"
"ke0040","Land and buildings","0040","account.data_account_type_fixed_assets","l10n_ke.l10nke_chart_template","","False"
"ke0050","Motor vehicles","0050","account.data_account_type_fixed_assets","l10n_ke.l10nke_chart_template","","False"
"ke0060","Office equipment (inc computer equipment)","0060","account.data_account_type_fixed_assets","l10n_ke.l10nke_chart_template","","False"
"ke0070","Plant and machinery","0070","account.data_account_type_fixed_assets","l10n_ke.l10nke_chart_template","","False"
"ke0080","Financial assets","0080","account.data_account_type_non_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke0090","Biological assets","0090","account.data_account_type_non_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke1001","Stock","1001","account.data_account_type_current_assets","l10n_ke.l10nke_chart_template","","True"
"ke1002","Work in Progress","1002","account.data_account_type_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke1003","Finished Goods","1003","account.data_account_type_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke1100","Debtors Control Account","1100","account.data_account_type_receivable","l10n_ke.l10nke_chart_template","","True"
"ke1101","Sundry Debtors","1101","account.data_account_type_receivable","l10n_ke.l10nke_chart_template","","True"
"ke1102","Other Debtors","1102","account.data_account_type_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke1103","Prepayments","1103","account.data_account_type_prepayments","l10n_ke.l10nke_chart_template","","False"
"ke1110","Purchase Tax Control Account","1110","account.data_account_type_current_assets","l10n_ke.l10nke_chart_template","","False"
"ke2100","Creditors Control Account","2100","account.data_account_type_payable","l10n_ke.l10nke_chart_template","","True"
"ke2101","Sundry Creditors","2101","account.data_account_type_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2102","Other Creditors","2102","account.data_account_type_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2103","Accruals","2103","account.data_account_type_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2104","Company Credit Card","2104","account.data_account_type_credit_card","l10n_ke.l10nke_chart_template","","True"
"ke2105","Bad debt provision","2105","account.data_account_type_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2200","Sales Tax Control Account","2200","account.data_account_type_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2201","HMRC - VAT Account","2201","account.data_account_type_payable","l10n_ke.l10nke_chart_template","","True"
"ke2202","Manual Adjustments & VAT","2202","account.data_account_type_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2210","P.A.Y.E. & NI","2210","account.data_account_type_payable","l10n_ke.l10nke_chart_template","","True"
"ke2220","Net Wages","2220","account.data_account_type_payable","l10n_ke.l10nke_chart_template","","True"
"ke2230","Pension Fund","2230","account.data_account_type_payable","l10n_ke.l10nke_chart_template","","True"
"ke2240","Corporation Tax","2240","account.data_account_type_payable","l10n_ke.l10nke_chart_template","","True"
"ke2300","Loans","2300","account.data_account_type_non_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2310","Hire Purchase","2310","account.data_account_type_non_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke2320","Mortgages","2330","account.data_account_type_non_current_liabilities","l10n_ke.l10nke_chart_template","","False"
"ke3000","Called up share capital","3000","account.data_account_type_equity","l10n_ke.l10nke_chart_template","","False"
"ke3010","Share premium account","3010","account.data_account_type_equity","l10n_ke.l10nke_chart_template","","False"
"ke3020","Revaluation reserve","3020","account.data_account_type_equity","l10n_ke.l10nke_chart_template","","False"
"ke3030","Other reserves","3030","account.data_account_type_equity","l10n_ke.l10nke_chart_template","","False"
"ke3040","Capital","3040","account.data_account_type_equity","l10n_ke.l10nke_chart_template","","False"
"ke4001","Sales category 1","4001","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4002","Sales category 2","4002","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4003","Sales category 3","4003","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4004","Sales category 4","4004","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4005","Bank Interest received","4005","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4006","Investment Interest received","4006","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4007","Revenue Income","4007","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4008","Profits/Losses on disposals of assets","4008","account.data_account_type_revenue","l10n_ke.l10nke_chart_template","","False"
"ke4010","Other Income","4010","account.data_account_type_other_income","l10n_ke.l10nke_chart_template","","False"
"ke5001","Cost of sales 1","5001","account.data_account_type_direct_costs","l10n_ke.l10nke_chart_template","","False"
"ke5002","Cost of sales 2","5002","account.data_account_type_direct_costs","l10n_ke.l10nke_chart_template","","False"
"ke5003","Cost of sales 3","5003","account.data_account_type_direct_costs","l10n_ke.l10nke_chart_template","","False"
"ke5004","Cost of sales 4","5004","account.data_account_type_direct_costs","l10n_ke.l10nke_chart_template","","False"
"ke5101","Marketing","5101","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5102","Exhibitions and events","5102","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5103","PR","5103","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5104","Distribution vehicles","5104","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5105","Distribution salaries and wages","5105","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5106","Shipping","5106","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5107","Directors pension","5107","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5108","Directors remuneration","5108","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5109","Gross Salaries","5109","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5110","Employers SDL & UIF","5110","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5111","Subcontractors payments","5111","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5112","Rent and rates","5112","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5113","Light / heat and power","5113","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5114","Repairs and maintenance","5114","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5115","Car hire","5115","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5116","Car fuel","5116","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5117","Car maintenance","5117","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5118","Telephone","5118","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5119","Internet & hosting","5119","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5120","Mobiles","5120","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5121","Stationery","5121","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5122","Office consumables","5122","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5123","Postage and Carriage","5123","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5124","Books","5124","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5125","Network costs","5125","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5126","Software expenses","5126","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5127","Other computer costs","5127","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5128","Recruitment fees","5128","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5129","Other admin expenses","5129","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5130","Accounting","5130","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5131","Auditing","5131","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5132","Consultancy","5132","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5133","Legal and professional charges","5133","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5134","Exchange gains/losses","5134","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5135","Other sundry expenses","5135","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5136","Bad debts","5136","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5137","Interest paid","5137","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5138","Bank Charges","5138","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5139","Donations","5139","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5140","Entertaining","5140","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5141","Insurance","5141","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5142","Travel and subsistence","5142","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5143","Corporation tax expense","5143","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5144","Foreign Exchange Gains/Losses","5144","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5145","Price Differences Control Account","5145","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5146","Cash Register Gains/Losses","5146","account.data_account_type_expenses","l10n_ke.l10nke_chart_template","","False"
"ke5201","Software Depreciation","5201","account.data_account_type_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5202","Patents & Trademarks Depreciation","5202","account.data_account_type_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5203","Fixtures and fittings Depreciation","5203","account.data_account_type_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5204","Land and buildings Depreciation","5204","account.data_account_type_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5205","Motor vehicles Depreciation","5205","account.data_account_type_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5206","Office equipment (inc computer equipment) Depreciation","5206","account.data_account_type_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5207","Plant and machinery Depreciation","5207","account.data_account_type_depreciation","l10n_ke.l10nke_chart_template","","False"

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ke.l10nke_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart template -->
    <record id="l10nke_chart_template" model="account.chart.template">
        <field name="name">Kenyan COA</field>
        <field name="bank_account_code_prefix">12000</field>
        <field name="cash_account_code_prefix">12500</field>
        <field name="transfer_account_code_prefix">12100</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.KES"/>
        <field name="country_id" ref="base.ke"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Fiscal Position Templates -->
        <record id="fiscal_position_template_national" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">National</field>
            <field name="chart_template_id" ref="l10nke_chart_template" />
            <field name="auto_apply" eval="True"/>
            <field name="country_id" ref="base.ke"/>
        </record>

        <record id="fiscal_position_template_non_kenyan" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">International</field>
            <field name="chart_template_id" ref="l10nke_chart_template" />
            <field name="auto_apply" eval="True"/>
        </record>

        <record id="import_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_kenyan"/>
            <field name="tax_src_id" ref="ST16"/>
            <field name="tax_dest_id" ref="ST0"/>
        </record>

        <record id="import_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_kenyan"/>
            <field name="tax_src_id" ref="ST8"/>
            <field name="tax_dest_id" ref="ST0"/>
        </record>

        <record id="export_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_kenyan"/>
            <field name="tax_src_id" ref="PT16"/>
            <field name="tax_dest_id" ref="PT0"/>
        </record>

        <record id="export_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_kenyan"/>
            <field name="tax_src_id" ref="PT8"/>
            <field name="tax_dest_id" ref="PT0"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_16" model="account.tax.group">
            <field name="name">TVA 16%</field>
            <field name="country_id" ref="base.ke"/>
        </record>

        <record id="tax_group_8" model="account.tax.group">
            <field name="name">TVA 8%</field>
            <field name="country_id" ref="base.ke"/>
        </record>

        <record id="tax_group_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
            <field name="country_id" ref="base.ke"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_ke" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.ke"/>
    </record>
    <record id="tax_report_line_sales_base" model="account.tax.report.line">
        <field name="name">Details of Sales (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_line_general_rate_sales_base" model="account.tax.report.line">
        <field name="name">1. General Rate 16% (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_base"/>
        <field name="tag_name">1 General Rate Base</field>
        <field name="code">box_1_base</field>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_line_other_rate_sales_base" model="account.tax.report.line">
        <field name="name">2. Other Rate 8% (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_base"/>
        <field name="tag_name">2 Other Rate Base</field>
        <field name="code">box_2_base</field>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_line_zero_rated_sales_base" model="account.tax.report.line">
        <field name="name">3. Zero Rated 0% (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_base"/>
        <field name="tag_name">3 Zero Rated Base</field>
        <field name="code">box_3_base</field>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_line_exempt_sales_base" model="account.tax.report.line">
        <field name="name">4. Exempt Sales (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_base"/>
        <field name="tag_name">4 Exempt Base</field>
        <field name="code">box_4_base</field>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_line_total_sales_base" model="account.tax.report.line">
        <field name="name">5. Total Sales (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_base"/>
        <field name="sequence">5</field>
        <field name="formula">box_1_base + box_2_base + box_3_base + box_4_base</field>
    </record>
    <record id="tax_report_line_total_output_base" model="account.tax.report.line">
        <field name="name">6. Total Output (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_base"/>
        <field name="sequence">5</field>
        <field name="formula">box_1_base + box_2_base + box_3_base</field>
    </record>

    <record id="tax_report_line_sales_tax" model="account.tax.report.line">
        <field name="name">Details of Sales (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_line_general_rate_sales_tax" model="account.tax.report.line">
        <field name="name">1. General Rate 16% (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_tax"/>
        <field name="tag_name">1 General Rate Tax</field>
        <field name="code">box_1_tax</field>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_line_other_rate_sales_tax" model="account.tax.report.line">
        <field name="name">2. Other Rate 8% (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_tax"/>
        <field name="tag_name">2 Other Rate Tax</field>
        <field name="code">box_2_tax</field>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_line_zero_rated_sales_tax" model="account.tax.report.line">
        <field name="name">3. Zero Rated 0% (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_tax"/>
        <field name="tag_name">3 Zero Rate Tax</field>
        <field name="code">box_3_tax</field>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_line_exempt_sales_tax" model="account.tax.report.line">
        <field name="name">4. Exempt Sales (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_tax"/>
        <field name="code">box_4_tax</field>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_line_total_sales_tax" model="account.tax.report.line">
        <field name="name">5. Total Sales (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_tax"/>
        <field name="code">box_5_tax</field>
        <field name="sequence">5</field>
        <field name="formula">box_1_tax + box_2_tax + box_3_tax</field>
    </record>
    <record id="tax_report_line_total_output_tax" model="account.tax.report.line">
        <field name="name">6. Total Output (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_sales_tax"/>
        <field name="code">box_6_output_vat</field>
        <field name="sequence">5</field>
        <field name="formula">box_1_tax + box_2_tax + box_3_tax</field>
    </record>

    <record id="tax_report_line_purchases_base" model="account.tax.report.line">
        <field name="name">Details of Purchases (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_line_general_rate_purchases_base" model="account.tax.report.line">
        <field name="name">7. General Rate 16% (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_base"/>
        <field name="tag_name">7 General Rate Base</field>
        <field name="code">box_7_base</field>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_line_other_rate_purchases_base" model="account.tax.report.line">
        <field name="name">8. Other Rate 8% (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_base"/>
        <field name="tag_name">8 Other Rate Base</field>
        <field name="code">box_8_base</field>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_line_zero_rated_purchases_base" model="account.tax.report.line">
        <field name="name">9. Zero Rated 0% (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_base"/>
        <field name="tag_name">9 Zero Rate Base</field>
        <field name="code">box_9_base</field>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_line_exempt_purchases_base" model="account.tax.report.line">
        <field name="name">10. Exempt Purchases (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_base"/>
        <field name="tag_name">10 Exempt Base</field>
        <field name="code">box_10_base</field>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_line_total_purchases_base" model="account.tax.report.line">
        <field name="name">11. Total Purchases (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_base"/>
        <field name="sequence">5</field>
        <field name="formula">box_7_base + box_8_base + box_9_base + box_10_base</field>
    </record>
    <record id="tax_report_line_total_input_vat_base" model="account.tax.report.line">
        <field name="name">12. Total Input VAT (Base)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_base"/>
        <field name="sequence">5</field>
        <field name="formula">box_7_base + box_8_base + box_9_base</field>
    </record>
    <record id="tax_report_line_purchases_tax" model="account.tax.report.line">
        <field name="name">Details of Purchases (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_line_general_rate_purchases_tax" model="account.tax.report.line">
        <field name="name">7. General Rate 16% (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_tax"/>
        <field name="tag_name">7 General Rate Purchase</field>
        <field name="code">box_7_tax</field>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_line_other_rate_purchases_tax" model="account.tax.report.line">
        <field name="name">8. Other Rate 8% (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_tax"/>
        <field name="tag_name">8 Other Rate Tax</field>
        <field name="code">box_8_tax</field>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_line_zero_rated_purchases_tax" model="account.tax.report.line">
        <field name="name">9. Zero Rated 0% (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_tax"/>
        <field name="tag_name">9 Zero Rate Tax</field>
        <field name="code">box_9_tax</field>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_line_exempt_purchases_tax" model="account.tax.report.line">
        <field name="name">10. Exempt Purchases (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_tax"/>
        <field name="code">box_10_tax</field>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_line_total_purchases_tax" model="account.tax.report.line">
        <field name="name">11. Total Purchases (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_tax"/>
        <field name="sequence">5</field>
        <field name="formula">box_7_tax + box_8_tax + box_9_tax</field>
    </record>
    <record id="tax_report_line_total_input_vat_tax" model="account.tax.report.line">
        <field name="name">12. Total Input VAT (Tax)</field>
        <field name="report_id" ref="tax_report_ke"/>
        <field name="parent_id" ref="tax_report_line_purchases_tax"/>
        <field name="sequence">5</field>
        <field name="formula">box_7_tax + box_8_tax + box_9_tax</field>
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="ST16" model="account.tax.template">
        <field name="description">Sales VAT (16%)</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Sales VAT (16%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">16</field>
        <field name="tax_group_id" ref="tax_group_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_general_rate_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'plus_report_line_ids': [ref('tax_report_line_general_rate_sales_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_general_rate_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'minus_report_line_ids': [ref('tax_report_line_general_rate_sales_tax')],
                }),
            ]"/>
    </record>

    <record id="ST8" model="account.tax.template">
        <field name="description">Sales VAT (8%)</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Sales VAT (8%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">8</field>
        <field name="tax_group_id" ref="tax_group_8"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_other_rate_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'plus_report_line_ids': [ref('tax_report_line_other_rate_sales_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_other_rate_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'minus_report_line_ids': [ref('tax_report_line_other_rate_sales_tax')],
                }),
            ]"/>
    </record>

    <record id="ST0" model="account.tax.template">
        <field name="description">Sales VAT Zero Rated</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Sales VAT Zero Rated (0%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_zero_rated_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'plus_report_line_ids': [ref('tax_report_line_zero_rated_sales_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_zero_rated_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'minus_report_line_ids': [ref('tax_report_line_zero_rated_sales_tax')],
                }),
            ]"/>
    </record>

    <record id="STEX" model="account.tax.template">
        <field name="description">Sales VAT Exempt</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Sales VAT Exempt</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_exempt_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_exempt_sales_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>

    <record id="PT16" model="account.tax.template">
        <field name="description">Purchases VAT (16%)</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Purchases VAT (16%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">16</field>
        <field name="tax_group_id" ref="tax_group_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_general_rate_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'plus_report_line_ids': [ref('tax_report_line_general_rate_purchases_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_general_rate_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'minus_report_line_ids': [ref('tax_report_line_general_rate_purchases_tax')],
                }),
            ]"/>
    </record>

    <record id="PT8" model="account.tax.template">
        <field name="description">Purchases VAT (8%)</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Purchases VAT (8%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">8</field>
        <field name="tax_group_id" ref="tax_group_8"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_other_rate_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'plus_report_line_ids': [ref('tax_report_line_other_rate_purchases_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_other_rate_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'minus_report_line_ids': [ref('tax_report_line_other_rate_purchases_tax')],
                }),
            ]"/>
    </record>

    <record id="PT0" model="account.tax.template">
        <field name="description">Purchases VAT Zero rated</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Purchases VAT Zero rated (0%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_zero_rated_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'plus_report_line_ids': [ref('tax_report_line_zero_rated_purchases_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_zero_rated_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'minus_report_line_ids': [ref('tax_report_line_zero_rated_purchases_tax')],
                }),
            ]"/>
    </record>

    <record id="PTEX" model="account.tax.template">
        <field name="description">Purchase VAT Exempt</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Purchase VAT Exempt</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_exempt_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_exempt_purchases_base')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
</odoo>

```

## File: data\l10n_ke_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="l10nke_chart_template" model="account.chart.template">
            <field name="name">Kenyan COA</field>
            <field name="property_account_receivable_id" ref="ke1100"/>
            <field name="property_account_payable_id" ref="ke2100"/>
            <field name="property_account_expense_categ_id" ref="ke5001"/>
            <field name="property_account_income_categ_id" ref="ke4001"/>
            <field name="property_tax_receivable_account_id" ref="ke1110"/>
            <field name="property_tax_payable_account_id" ref="ke2200"/>
            <field name="income_currency_exchange_account_id" ref="ke5144"/>
            <field name="expense_currency_exchange_account_id" ref="ke5144"/>
            <field name="default_cash_difference_income_account_id" ref="ke5146"/>
            <field name="default_cash_difference_expense_account_id" ref="ke5146"/>
            <field name="use_anglo_saxon" eval="True"/>
        </record>
</odoo>

```

## File: data\menu_item_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_ke_statements_menu" name="Kenya" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

