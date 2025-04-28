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
    ],
    'demo': [
        'demo/demo_company.xml'
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id:id","tag_ids/id","reconcile"
"ke0010","Software","0010","asset_non_current","l10n_ke.l10nke_chart_template","","False"
"ke0020","Patents & Trademarks","0020","asset_non_current","l10n_ke.l10nke_chart_template","","False"
"ke0030","Fixtures and fittings","0030","asset_fixed","l10n_ke.l10nke_chart_template","","False"
"ke0040","Land and buildings","0040","asset_fixed","l10n_ke.l10nke_chart_template","","False"
"ke0050","Motor vehicles","0050","asset_fixed","l10n_ke.l10nke_chart_template","","False"
"ke0060","Office equipment (inc computer equipment)","0060","asset_fixed","l10n_ke.l10nke_chart_template","","False"
"ke0070","Plant and machinery","0070","asset_fixed","l10n_ke.l10nke_chart_template","","False"
"ke0080","Financial assets","0080","asset_non_current","l10n_ke.l10nke_chart_template","","False"
"ke0090","Biological assets","0090","asset_non_current","l10n_ke.l10nke_chart_template","","False"
"ke1001","Stock","1001","asset_current","l10n_ke.l10nke_chart_template","","True"
"ke100110","Stock Interim (Received)","100110","asset_current","l10n_ke.l10nke_chart_template","","True"
"ke100120","Stock Interim (Delivered)","100120","asset_current","l10n_ke.l10nke_chart_template","","True"
"ke1002","Work in Progress","1002","asset_current","l10n_ke.l10nke_chart_template","","False"
"ke1003","Finished Goods","1003","asset_current","l10n_ke.l10nke_chart_template","","False"
"ke1100","Debtors Control Account","1100","asset_receivable","l10n_ke.l10nke_chart_template","","True"
"ke110010","Debtors Control Account (POS)","110010","asset_receivable","l10n_ke.l10nke_chart_template","","True"
"ke1101","Sundry Debtors","1101","asset_receivable","l10n_ke.l10nke_chart_template","","True"
"ke1102","Other Debtors","1102","asset_current","l10n_ke.l10nke_chart_template","","False"
"ke1103","Prepayments","1103","asset_prepayments","l10n_ke.l10nke_chart_template","","False"
"ke1110","Purchase Tax Control Account","1110","asset_current","l10n_ke.l10nke_chart_template","","False"
"ke1120","Withholding Tax Advance on Sales","1120","asset_current","l10n_ke.l10nke_chart_template","","True"
"ke2100","Creditors Control Account","2100","liability_payable","l10n_ke.l10nke_chart_template","","True"
"ke2101","Sundry Creditors","2101","liability_current","l10n_ke.l10nke_chart_template","","False"
"ke2102","Other Creditors","2102","liability_current","l10n_ke.l10nke_chart_template","","False"
"ke2103","Accruals","2103","liability_current","l10n_ke.l10nke_chart_template","","False"
"ke2104","Company Credit Card","2104","liability_credit_card","l10n_ke.l10nke_chart_template","","False"
"ke2105","Bad debt provision","2105","liability_current","l10n_ke.l10nke_chart_template","","False"
"ke2200","Sales Tax Control Account","2200","liability_current","l10n_ke.l10nke_chart_template","","False"
"ke2201","HMRC - VAT Account","2201","liability_payable","l10n_ke.l10nke_chart_template","","True"
"ke2202","Manual Adjustments & VAT","2202","liability_current","l10n_ke.l10nke_chart_template","","False"
"ke2203","Withholding Tax Payable","2203","liability_current","l10n_ke.l10nke_chart_template","","True"
"ke2210","P.A.Y.E. & NI","2210","liability_payable","l10n_ke.l10nke_chart_template","","True"
"ke2220","Net Wages","2220","liability_payable","l10n_ke.l10nke_chart_template","","True"
"ke2230","Pension Fund","2230","liability_payable","l10n_ke.l10nke_chart_template","","True"
"ke2240","Corporation Tax","2240","liability_payable","l10n_ke.l10nke_chart_template","","True"
"ke2300","Loans","2300","liability_non_current","l10n_ke.l10nke_chart_template","","False"
"ke2310","Hire Purchase","2310","liability_non_current","l10n_ke.l10nke_chart_template","","False"
"ke2320","Mortgages","2320","liability_non_current","l10n_ke.l10nke_chart_template","","False"
"ke3000","Called up share capital","3000","equity","l10n_ke.l10nke_chart_template","","False"
"ke3010","Share premium account","3010","equity","l10n_ke.l10nke_chart_template","","False"
"ke3020","Revaluation reserve","3020","equity","l10n_ke.l10nke_chart_template","","False"
"ke3030","Other reserves","3030","equity","l10n_ke.l10nke_chart_template","","False"
"ke3040","Capital","3040","equity","l10n_ke.l10nke_chart_template","","False"
"ke4001","Sales category 1","4001","income","l10n_ke.l10nke_chart_template","","False"
"ke4002","Sales category 2","4002","income","l10n_ke.l10nke_chart_template","","False"
"ke4003","Sales category 3","4003","income","l10n_ke.l10nke_chart_template","","False"
"ke4004","Sales category 4","4004","income","l10n_ke.l10nke_chart_template","","False"
"ke4005","Bank Interest received","4005","income","l10n_ke.l10nke_chart_template","","False"
"ke4006","Investment Interest received","4006","income","l10n_ke.l10nke_chart_template","","False"
"ke4007","Revenue Income","4007","income","l10n_ke.l10nke_chart_template","","False"
"ke400710","Cash Discount Gain","400710","income_other","l10n_ke.l10nke_chart_template","","False"
"ke4008","Profits/Losses on disposals of assets","4008","income","l10n_ke.l10nke_chart_template","","False"
"ke4010","Other Income","4010","income_other","l10n_ke.l10nke_chart_template","","False"
"ke5001","Cost of sales 1","5001","expense_direct_cost","l10n_ke.l10nke_chart_template","","False"
"ke5002","Cost of sales 2","5002","expense_direct_cost","l10n_ke.l10nke_chart_template","","False"
"ke5003","Cost of sales 3","5003","expense_direct_cost","l10n_ke.l10nke_chart_template","","False"
"ke5004","Cost of sales 4","5004","expense_direct_cost","l10n_ke.l10nke_chart_template","","False"
"ke5101","Marketing","5101","expense","l10n_ke.l10nke_chart_template","","False"
"ke5102","Exhibitions and events","5102","expense","l10n_ke.l10nke_chart_template","","False"
"ke5103","PR","5103","expense","l10n_ke.l10nke_chart_template","","False"
"ke5104","Distribution vehicles","5104","expense","l10n_ke.l10nke_chart_template","","False"
"ke5105","Distribution salaries and wages","5105","expense","l10n_ke.l10nke_chart_template","","False"
"ke5106","Shipping","5106","expense","l10n_ke.l10nke_chart_template","","False"
"ke5107","Directors pension","5107","expense","l10n_ke.l10nke_chart_template","","False"
"ke5108","Directors remuneration","5108","expense","l10n_ke.l10nke_chart_template","","False"
"ke5109","Gross Salaries","5109","expense","l10n_ke.l10nke_chart_template","","False"
"ke5110","Employers SDL & UIF","5110","expense","l10n_ke.l10nke_chart_template","","False"
"ke5111","Subcontractors payments","5111","expense","l10n_ke.l10nke_chart_template","","False"
"ke5112","Rent and rates","5112","expense","l10n_ke.l10nke_chart_template","","False"
"ke5113","Light / heat and power","5113","expense","l10n_ke.l10nke_chart_template","","False"
"ke5114","Repairs and maintenance","5114","expense","l10n_ke.l10nke_chart_template","","False"
"ke5115","Car hire","5115","expense","l10n_ke.l10nke_chart_template","","False"
"ke5116","Car fuel","5116","expense","l10n_ke.l10nke_chart_template","","False"
"ke5117","Car maintenance","5117","expense","l10n_ke.l10nke_chart_template","","False"
"ke5118","Telephone","5118","expense","l10n_ke.l10nke_chart_template","","False"
"ke5119","Internet & hosting","5119","expense","l10n_ke.l10nke_chart_template","","False"
"ke5120","Mobiles","5120","expense","l10n_ke.l10nke_chart_template","","False"
"ke5121","Stationery","5121","expense","l10n_ke.l10nke_chart_template","","False"
"ke5122","Office consumables","5122","expense","l10n_ke.l10nke_chart_template","","False"
"ke5123","Postage and Carriage","5123","expense","l10n_ke.l10nke_chart_template","","False"
"ke5124","Books","5124","expense","l10n_ke.l10nke_chart_template","","False"
"ke5125","Network costs","5125","expense","l10n_ke.l10nke_chart_template","","False"
"ke5126","Software expenses","5126","expense","l10n_ke.l10nke_chart_template","","False"
"ke5127","Other computer costs","5127","expense","l10n_ke.l10nke_chart_template","","False"
"ke5128","Recruitment fees","5128","expense","l10n_ke.l10nke_chart_template","","False"
"ke5129","Other admin expenses","5129","expense","l10n_ke.l10nke_chart_template","","False"
"ke5130","Accounting","5130","expense","l10n_ke.l10nke_chart_template","","False"
"ke5131","Auditing","5131","expense","l10n_ke.l10nke_chart_template","","False"
"ke5132","Consultancy","5132","expense","l10n_ke.l10nke_chart_template","","False"
"ke5133","Legal and professional charges","5133","expense","l10n_ke.l10nke_chart_template","","False"
"ke5134","Exchange gains/losses","5134","expense","l10n_ke.l10nke_chart_template","","False"
"ke5135","Other sundry expenses","5135","expense","l10n_ke.l10nke_chart_template","","False"
"ke5136","Bad debts","5136","expense","l10n_ke.l10nke_chart_template","","False"
"ke5137","Interest paid","5137","expense","l10n_ke.l10nke_chart_template","","False"
"ke5138","Bank Charges","5138","expense","l10n_ke.l10nke_chart_template","","False"
"ke5139","Donations","5139","expense","l10n_ke.l10nke_chart_template","","False"
"ke5140","Entertaining","5140","expense","l10n_ke.l10nke_chart_template","","False"
"ke5141","Insurance","5141","expense","l10n_ke.l10nke_chart_template","","False"
"ke5142","Travel and subsistence","5142","expense","l10n_ke.l10nke_chart_template","","False"
"ke5143","Corporation tax expense","5143","expense","l10n_ke.l10nke_chart_template","","False"
"ke5144","Foreign Exchange Gains/Losses","5144","expense","l10n_ke.l10nke_chart_template","","False"
"ke5145","Price Differences Control Account","5145","expense","l10n_ke.l10nke_chart_template","","False"
"ke5146","Cash Register Gains/Losses","5146","expense","l10n_ke.l10nke_chart_template","","False"
"ke5147","Cash Discount Loss","5147","expense","l10n_ke.l10nke_chart_template","","False"
"ke5201","Software Depreciation","5201","expense_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5202","Patents & Trademarks Depreciation","5202","expense_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5203","Fixtures and fittings Depreciation","5203","expense_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5204","Land and buildings Depreciation","5204","expense_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5205","Motor vehicles Depreciation","5205","expense_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5206","Office equipment (inc computer equipment) Depreciation","5206","expense_depreciation","l10n_ke.l10nke_chart_template","","False"
"ke5207","Plant and machinery Depreciation","5207","expense_depreciation","l10n_ke.l10nke_chart_template","","False"

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

        <record id="tax_group_withholding" model="account.tax.group">
            <field name="name">Withholding Tax</field>
            <field name="country_id" ref="base.ke"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_ke" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ke"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_base_column" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="tax_report_tax_column" model="account.report.column">
                <field name="name">VAT</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_general_rate_sales" model="account.report.line">
                <field name="name">1. Taxable Sales (General Rate 16%)</field>
                <field name="sequence">1</field>
                <field name="code">box_1</field>
                <field name="expression_ids">
                        <record id="tax_report_general_rate_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Sales Base</field>
                    </record>
                    <record id="tax_report_general_rate_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Sales Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_other_rate_sales" model="account.report.line">
                <field name="name">2. Taxable Sales (Other Rate 8%)</field>
                <field name="sequence">2</field>
                <field name="code">box_2</field>
                <field name="expression_ids">
                        <record id="tax_report_other_rate_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% Sales Base</field>
                    </record>
                    <record id="tax_report_other_rate_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% Sales Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_zero_rated_sales" model="account.report.line">
                <field name="name">3. Sales (Zero Rated 0%)</field>
                <field name="sequence">3</field>
                <field name="code">box_3</field>
                <field name="expression_ids">
                        <record id="tax_report_zero_rated_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Zero Rated Sales Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_exempt_sales" model="account.report.line">
                <field name="name">4. Sales (Exempt)</field>
                <field name="sequence">4</field>
                <field name="code">box_4</field>
                <field name="expression_ids">
                        <record id="tax_report_exempt_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Exempt Sales Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_sales" model="account.report.line">
                <field name="name">5. Total Sales</field>
                <field name="sequence">5</field>
                <field name="code">box_5</field>
                <field name="expression_ids">
                        <record id="tax_report_total_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.base + box_2.base + box_3.base + box_4.base</field>
                    </record>
                    <record id="tax_report_total_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.tax + box_2.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_output_vat" model="account.report.line">
                <field name="name">6. Total Output VAT</field>
                <field name="sequence">6</field>
                <field name="code">box_6</field>
                <field name="expression_ids">
                        <record id="tax_report_output_vat_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.base + box_2.base + box_3.base</field>
                    </record>
                    <record id="tax_report_output_vat_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.tax + box_2.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_general_rate_purchases" model="account.report.line">
                <field name="name">7. Taxable Purchases (General Rate 16%)</field>
                <field name="sequence">7</field>
                <field name="code">box_7</field>
                <field name="expression_ids">
                        <record id="tax_report_general_rate_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Purchases Base</field>
                    </record>
                    <record id="tax_report_general_rate_purchases_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Purchases Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_other_rate_purchases" model="account.report.line">
                <field name="name">8. Taxable Purchases (Other Rate 8%)</field>
                <field name="sequence">8</field>
                <field name="code">box_8</field>
                <field name="expression_ids">
                        <record id="tax_report_other_rate_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% purchases Base</field>
                    </record>
                    <record id="tax_report_other_rate_purchases_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% Purchases Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_zero_rated_purchases" model="account.report.line">
                <field name="name">9. Purchases (Zero Rated 0%)</field>
                <field name="sequence">9</field>
                <field name="code">box_9</field>
                <field name="expression_ids">
                        <record id="tax_report_zero_rated_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Zero Rated Purchases Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_exempt_purchases" model="account.report.line">
                <field name="name">10. Purchases (Exempt)</field>
                <field name="sequence">10</field>
                <field name="code">box_10</field>
                <field name="expression_ids">
                        <record id="tax_report_exempt_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Exempt Purchases Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_purchases" model="account.report.line">
                <field name="name">11. Total Purchases</field>
                <field name="sequence">11</field>
                <field name="code">box_11</field>
                <field name="expression_ids">
                        <record id="tax_report_total_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.base + box_8.base + box_9.base + box_10.base</field>
                    </record>
                    <record id="tax_report_total_purchases_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.tax + box_8.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_input_vat" model="account.report.line">
                <field name="name">12. Total Input VAT</field>
                <field name="sequence">12</field>
                <field name="code">box_12</field>
                <field name="expression_ids">
                        <record id="tax_report_input_vat_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.base + box_8.base + box_9.base</field>
                    </record>
                    <record id="tax_report_input_vat_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.tax + box_8.tax</field>
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_general_rate_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'plus_report_expression_ids': [ref('tax_report_general_rate_sales_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_general_rate_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'minus_report_expression_ids': [ref('tax_report_general_rate_sales_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_other_rate_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'plus_report_expression_ids': [ref('tax_report_other_rate_sales_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_other_rate_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke2200'),
                    'minus_report_expression_ids': [ref('tax_report_other_rate_sales_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_zero_rated_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_zero_rated_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_exempt_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_exempt_sales_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record id="SWT3" model="account.tax.template">
        <field name="name">3% WHT</field>
        <field name="description">3%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-3</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
    </record>
    <record id="SWT5" model="account.tax.template">
        <field name="name">5% WHT</field>
        <field name="description">5%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
    </record>
    <record id="SWT10" model="account.tax.template">
        <field name="name">10% WHT</field>
        <field name="description">10%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
    </record>
    <record id="SWT12" model="account.tax.template">
        <field name="name">12% WHT</field>
        <field name="description">12%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-12</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
    </record>
    <record id="SWT15" model="account.tax.template">
        <field name="name">15% WHT</field>
        <field name="description">15%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-15</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
    </record>
    <record id="SWT20" model="account.tax.template">
        <field name="name">20% WHT</field>
        <field name="description">20%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
    </record>
    <record id="SWT25" model="account.tax.template">
        <field name="name">25% WHT</field>
        <field name="description">25%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-25</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
    </record>
    <record id="SWT30" model="account.tax.template">
        <field name="name">30% WHT</field>
        <field name="description">30%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">-30</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke1120'),
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_general_rate_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'plus_report_expression_ids': [ref('tax_report_general_rate_purchases_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_general_rate_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'minus_report_expression_ids': [ref('tax_report_general_rate_purchases_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_other_rate_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'plus_report_expression_ids': [ref('tax_report_other_rate_purchases_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_other_rate_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ke1110'),
                    'minus_report_expression_ids': [ref('tax_report_other_rate_purchases_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_zero_rated_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_zero_rated_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_exempt_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_exempt_purchases_base_tag')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record id="PWT3" model="account.tax.template">
        <field name="name">3% WHT</field>
        <field name="description">3%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-3</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
    </record>
    <record id="PWT5" model="account.tax.template">
        <field name="name">5% WHT</field>
        <field name="description">5%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
    </record>
    <record id="PWT10" model="account.tax.template">
        <field name="name">10% WHT</field>
        <field name="description">10%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
    </record>
    <record id="PWT12" model="account.tax.template">
        <field name="name">12% WHT</field>
        <field name="description">12%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-12</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
    </record>
    <record id="PWT15" model="account.tax.template">
        <field name="name">15% WHT</field>
        <field name="description">15%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-15</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
    </record>
    <record id="PWT20" model="account.tax.template">
        <field name="name">20% WHT</field>
        <field name="description">20%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
    </record>
    <record id="PWT25" model="account.tax.template">
        <field name="name">25% WHT</field>
        <field name="description">25%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-25</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
    </record>
    <record id="PWT30" model="account.tax.template">
        <field name="name">30% WHT</field>
        <field name="description">30%</field>
        <field name="chart_template_id" ref="l10nke_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">-30</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('ke2203'),
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
            <field name="account_journal_early_pay_discount_loss_account_id" ref="ke5147"/>
            <field name="account_journal_early_pay_discount_gain_account_id" ref="ke400710"/>
            <field name="default_pos_receivable_account_id" ref="ke110010"/>
            <field name="property_stock_valuation_account_id" ref="ke1001"/>
            <field name="property_stock_account_output_categ_id" ref="ke100120"/>
            <field name="property_stock_account_input_categ_id" ref="ke100110"/>
            <field name="use_anglo_saxon" eval="True"/>
        </record>
</odoo>

```

