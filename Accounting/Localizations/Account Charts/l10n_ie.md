# Odoo Module: l10n_ie

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
    "name": "Ireland - Accounting",
    "version": "2.0",
    'countries': ['ie'],
    "icon": '/account/static/description/l10n.png',
    "category": "Accounting/Localizations/Account Charts",
    "description": """
This is the base module to manage the accounting chart for Republic of Ireland in Odoo. 
    """,
    "author": "Odoo SA",
    "depends": [
        "account",
        "base_iban",
        "base_vat",
    ],
    "data": [
        "data/tax_report-ie.xml",
    ],
    "demo": [
        "demo/demo_company.xml",
    ],
    "license": "LGPL-3",
}

```

## File: data\tax_report-ie.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_ie_tr" model="account.report">
        <field name="name">Tax report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ie"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ie_tr_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ie_tr_T1" model="account.report.line">
                <field name="name">T1: VAT on Sales</field>
                <field name="code">l10n_ie_tr_T1</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_T1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">T1</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_T2" model="account.report.line">
                <field name="name">T2: VAT on Purchases</field>
                <field name="code">l10n_ie_tr_T2</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_T2_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">T2</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_T3" model="account.report.line">
                <field name="name">T3: VAT payable</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_T3_aggregation" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ie_tr_T1.balance - l10n_ie_tr_T2.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_T4" model="account.report.line">
                <field name="name">T4: VAT repayable</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_T4_aggregation" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ie_tr_T2.balance - l10n_ie_tr_T1.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_E1" model="account.report.line">
                <field name="name">E1: intra-EU supplies of goods</field>
                <field name="code">l10n_ie_tr_E1</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_E1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">E1</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_E2" model="account.report.line">
                <field name="name">E2: intra-EU acquisitions of goods</field>
                <field name="code">l10n_ie_tr_E2</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_E2_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">E2</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_ES1" model="account.report.line">
                <field name="name">ES1: intra-EU supply of service</field>
                <field name="code">l10n_ie_tr_ES1</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_ES1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">ES1</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_ES2" model="account.report.line">
                <field name="name">ES2: intra-EU acquisition of services</field>
                <field name="code">l10n_ie_tr_ES2</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_ES2_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">ES2</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ie_tr_PA1" model="account.report.line">
                <field name="name">PA1: postponed accounting</field>
                <field name="code">l10n_ie_tr_PA1</field>
                <field name="expression_ids">
                    <record id="l10n_ie_tr_PA1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">PA1</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-ie.csv

```csv
"id","code","name","account_type","reconcile","tag_ids"
"l10n_ie_account_100","100","Development costs","asset_fixed","False",""
"l10n_ie_account_101","101","Concessions, patents, licences, trade marks and similar rights and assets","asset_fixed","False",""
"l10n_ie_account_102","102","Goodwill","asset_fixed","False",""
"l10n_ie_account_103","103","Payments on account","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_110","110","Investment property","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_111","111","Land and buildings","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_112","112","Plant and machinery","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_113","113","Fixtures, fittings, tools and equipment","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_114","114","Payments on account and assets in course of construction","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_120","120","Shares in group undertakings","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_121","121","Loans to group undertakings","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_122","122","Participating interests","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_123","123","Loans to undertakings in which the company has a participating interest","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1240","1240","Listed shares","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1241","1241","Unlisted shares","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1242","1242","Other securities held as fixed assets (equity right)","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1243","1243","Debentures","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1244","1244","Other securities held as fixed assets (creditor's right)","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1245","1245","Shares of collective investment funds","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1246","1246","Other securities held as fixed assets","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1250","1250","Loans","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1251","1251","Deposits and guarantees paid","asset_fixed","False","account.account_tag_investing"
"l10n_ie_account_1252","1252","Long-term receivables","asset_fixed","False",""
"l10n_ie_account_2000","2000","Inventories of raw materials","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2001","2001","Inventories of consumable materials and supplies","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2002","2002","Inventories of packaging","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2010","2010","Inventories of work in progress","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2011","2011","Contracts in progress - goods","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2012","2012","Contracts in progress - services","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2013","2013","Buildings under construction","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2020","2020","Inventories of finished goods","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2021","2021","Inventories of semi-finished goods","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2022","2022","Inventories of residual goods (waste, rejected and recuperable material)","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2023","2023","Inventories of merchandise","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2024","2024","Inventories of land for resale","asset_current","False","account.account_tag_operating"
"l10n_ie_account_2025","2025","Inventories of buildings for resale","asset_current","False","account.account_tag_operating"
"l10n_ie_account_203","203","Payments on account","asset_current","False",""
"l10n_ie_account_2100","2100","Customers due within one year","asset_receivable","True",""
"l10n_ie_account_2101","2101","Customers (PoS) due within one year","asset_receivable","True",""
"l10n_ie_account_2102","2102","Customers - Receivable bills of exchange due within one year","asset_current","True",""
"l10n_ie_account_2103","2103","Doubtful or disputed customers due within one year","asset_current","True",""
"l10n_ie_account_2104","2104","Customers - Unbilled sales due within one year","asset_current","True",""
"l10n_ie_account_2105","2105","Customers with a credit balance due within one year","liability_current","True",""
"l10n_ie_account_2106","2106","Value adjustments of receivables due within one year","asset_current","False",""
"l10n_ie_account_211","211","Amounts owed by group undertakings","asset_current","False",""
"l10n_ie_account_212","212","Amounts owed by undertakings in which the company has a participating interest","asset_current","False",""
"l10n_ie_account_2130","2130","Other debtors","asset_current","False",""
"l10n_ie_account_2131","2131","VAT receivable","asset_current","False",""
"l10n_ie_account_2132","2132","VAT down payments made","asset_current","False",""
"l10n_ie_account_214","214","Subscribed capital called but unpaid","asset_fixed","False",""
"l10n_ie_account_215","215","Prepayments","asset_current","False",""
"l10n_ie_account_2160","2160","Accrued income","asset_current","False",""
"l10n_ie_account_2161","2161","Deferred expenses","asset_current","False",""
"l10n_ie_account_220","220","Shares in group undertakings","asset_current","False",""
"l10n_ie_account_221","221","Other investments","asset_current","False",""
"l10n_ie_account_30","30","Debenture loans: due and payable within one year","liability_current","False",""
"l10n_ie_account_31","31","Amounts owed to credit institutions: due and payable within one year","liability_current","False",""
"l10n_ie_account_32","32","Called-up share capital presented as a liability due within one year","liability_non_current","False",""
"l10n_ie_account_33","33","Payments received on account","liability_current","False",""
"l10n_ie_account_34","34","Suppliers due within one year","liability_payable","True",""
"l10n_ie_account_35","35","Bills of exchange payable within one year","liability_current","False",""
"l10n_ie_account_36","36","Amounts owed to group undertakings due within one year","liability_current","False",""
"l10n_ie_account_37","37","Amounts owed to undertakings with which the company is linked by virtue of participating interests due within one year","liability_current","False",""
"l10n_ie_account_3800","3800","Income tax","liability_current","False",""
"l10n_ie_account_3801","3801","Withholding tax on wages and salaries","liability_current","False",""
"l10n_ie_account_3802","3802","Withholding tax on financial investment income","liability_current","False",""
"l10n_ie_account_3803","3803","VAT received","liability_current","True",""
"l10n_ie_account_3804","3804","VAT payable","liability_current","False",""
"l10n_ie_account_3805","3805","Other indirect taxes","liability_current","False",""
"l10n_ie_account_3806","3806","Foreign VAT","liability_current","False",""
"l10n_ie_account_3807","3807","Other foreign taxes","liability_current","False",""
"l10n_ie_account_381","381","Pay Related Social insurance (PRSI)","liability_current","False",""
"l10n_ie_account_3820","3820","Received deposits and guarantees within one year","liability_current","True",""
"l10n_ie_account_3821","3821","Amounts payable to partners and shareholders (others than from affiliated undertakings) within one year","liability_current","True","account.account_tag_financing"
"l10n_ie_account_3822","3822","Amounts payable to directors, managers, statutory auditors and similar within one year","liability_current","True",""
"l10n_ie_account_3823","3823","Amounts payable to staff within one year","liability_current","True",""
"l10n_ie_account_3824","3824","Other miscellaneous debts within one year","liability_current","False",""
"l10n_ie_account_39","39","Deferred income to be recognized within one year","liability_current","False",""
"l10n_ie_account_40","40","Debenture loans: due and payable after more than one year","liability_non_current","False","account.account_tag_financing"
"l10n_ie_account_41","41","Amounts owed to credit institutions: due and payable after more than one year","liability_non_current","False","account.account_tag_financing"
"l10n_ie_account_42","42","Called-up share capital presented as a liability due after more than one year","liability_non_current","False","account.account_tag_financing"
"l10n_ie_account_43","43","Down payments received after more than one year","liability_current","False",""
"l10n_ie_account_44","44","Suppliers due after more than one year","liability_payable","True",""
"l10n_ie_account_45","45","Bills of exchange payable after more than one year","liability_current","False",""
"l10n_ie_account_46","46","Amounts owed to group undertakings due after more than one year","liability_current","False",""
"l10n_ie_account_47","47","Amounts owed to undertakings with which the company is linked by virtue of participating interests due after more than one year","liability_current","False",""
"l10n_ie_account_480","480","Received deposits and guarantees due after more than one year","liability_current","True",""
"l10n_ie_account_481","481","Amounts payable to partners and shareholders (others than from affiliated undertakings) due after more than one year","liability_current","True","account.account_tag_financing"
"l10n_ie_account_482","482","Amounts payable to directors, managers, statutory auditors and similar due after more than one year","liability_current","True",""
"l10n_ie_account_483","483","Amounts payable to staff due after more than one year","liability_current","True",""
"l10n_ie_account_484","484","Other miscellaneous debts due after more than one year","liability_current","False",""
"l10n_ie_account_49","49","Deferred income to be recognized after more than one year","liability_current","False",""
"l10n_ie_account_500","500","Provisions for pensions and similar obligations","liability_non_current","False",""
"l10n_ie_account_5010","5010","Provisions for taxation","liability_non_current","False",""
"l10n_ie_account_5011","5011","Deferred tax provisions","liability_non_current","False",""
"l10n_ie_account_5020","5020","Operating provisions","liability_non_current","False",""
"l10n_ie_account_5021","5021","Financial provisions","liability_non_current","False",""
"l10n_ie_account_5100","5100","Called-up capital","equity","False","account.account_tag_financing"
"l10n_ie_account_5101","5101","Capital of individual companies, corporate partnerships and similar","equity","False","account.account_tag_financing"
"l10n_ie_account_5110","5110","Share premium","equity","False","account.account_tag_financing"
"l10n_ie_account_5111","5111","Merger premium","equity","False","account.account_tag_financing"
"l10n_ie_account_5112","5112","Contribution premium","equity","False","account.account_tag_financing"
"l10n_ie_account_5113","5113","Premiums on conversion of bonds into shares","equity","False","account.account_tag_financing"
"l10n_ie_account_5114","5114","Capital contribution without issue of shares","equity","False","account.account_tag_financing"
"l10n_ie_account_5120","5120","Reserves in application of the equity method","equity","False",""
"l10n_ie_account_5121","5121","Temporarily not taxable currency translation adjustments","equity","False",""
"l10n_ie_account_5122","5122","Other revaluation reserves","equity","False",""
"l10n_ie_account_5130","5130","Legal reserve","equity","False",""
"l10n_ie_account_5131","5131","Reserves for own shares or own corporate units","equity","False",""
"l10n_ie_account_5132","5132","Reserves provided for by the articles of association","equity","False",""
"l10n_ie_account_51330","51330","Other reserves available for distribution","equity","False",""
"l10n_ie_account_51331","51331","Reserves in application of fair value","equity","False",""
"l10n_ie_account_51332","51332","Reserves not available for distribution not mentioned above","equity","False",""
"l10n_ie_account_5140","5140","Results brought forward in the process of assignment","equity","False",""
"l10n_ie_account_5141","5141","Results brought forward (assigned)","equity","False",""
"l10n_ie_account_515","515","Result for the financial year","equity_unaffected","False",""
"l10n_ie_account_60","60","Purchase of Raw materials and consumables","expense","False","account.account_tag_operating"
"l10n_ie_account_61","61","Other external expenses","expense","False","account.account_tag_operating"
"l10n_ie_account_620","620","Wages and salaries","expense","False","account.account_tag_operating"
"l10n_ie_account_621","621","Social insurance costs","expense","False","account.account_tag_operating"
"l10n_ie_account_622","622","Other retirement benefit costs","expense","False","account.account_tag_operating"
"l10n_ie_account_623","623","Other compensation costs","expense","False",""
"l10n_ie_account_630","630","Value adjustments in respect of fixed assets","expense","False",""
"l10n_ie_account_631","631","Extraordinary value adjustments in respect of current assets","expense","False",""
"l10n_ie_account_640","640","Cash Discount Loss","expense","False","account.account_tag_operating"
"l10n_ie_account_641","641","Cash Difference Loss","expense","False","account.account_tag_operating"
"l10n_ie_account_642","642","Other operating expenses","expense","False","account.account_tag_operating"
"l10n_ie_account_65","65","Value adjustments in respect of financial assets and investments held as current assets - Expense","expense","False",""
"l10n_ie_account_660","660","Interest payable and similar expenses","expense","False","account.account_tag_financing"
"l10n_ie_account_661","661","Foreign currency exchange losses","expense","False","account.account_tag_operating"
"l10n_ie_account_67","67","Tax on profit or loss","expense","False",""
"l10n_ie_account_68","68","Other taxes not shown under the above items","expense","False",""
"l10n_ie_account_70","70","Sales","income","False","account.account_tag_operating"
"l10n_ie_account_71","71","Variation in stocks of finished goods and in work in progress","income","False","account.account_tag_operating"
"l10n_ie_account_72","72","Own work capitalized","income","False","account.account_tag_investing"
"l10n_ie_account_730","730","Cash Discount Gain","income","False","account.account_tag_operating"
"l10n_ie_account_731","731","Cash Difference Gain","income","False","account.account_tag_operating"
"l10n_ie_account_732","732","Other operating income","income","False","account.account_tag_operating"
"l10n_ie_account_74","74","Income from shares in group undertakings","income","False","account.account_tag_investing"
"l10n_ie_account_75","75","Income from participating interests","income","False","account.account_tag_investing"
"l10n_ie_account_760","760","Income from other financial assets","income","False","account.account_tag_investing"
"l10n_ie_account_761","761","Foreign currency exchange gains","income","False","account.account_tag_operating"
"l10n_ie_account_762","762","Income from other financial assets","income","False","account.account_tag_investing"
"l10n_ie_account_77","77","Other interest receivable and similar income","income","False","account.account_tag_investing"
"l10n_ie_account_78","78","Value adjustments in respect of financial assets and investments held as current assets - Income","income","False",""

```

## File: data\template\account.fiscal.position-ie.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id"
"ie_fp_domestic","1","Domestic","1","1","base.ie","","","","",""
"ie_fp_eu_private","2","EU private","1","","","base.europe","","","",""
"ie_fp_eu","3","Intra-community (EU)","1","1","","base.europe","ie_tax_sale_goods_23","ie_tax_sale_goods_eu_0","",""
"","","","","","","","ie_tax_sale_services_23","ie_tax_sale_services_eu_0","",""
"","","","","","","","ie_tax_sale_goods_13_5","ie_tax_sale_goods_eu_0","",""
"","","","","","","","ie_tax_sale_services_13_5","ie_tax_sale_services_eu_0","",""
"","","","","","","","ie_tax_sale_goods_9","ie_tax_sale_goods_eu_0","",""
"","","","","","","","ie_tax_sale_services_9","ie_tax_sale_services_eu_0","",""
"","","","","","","","ie_tax_sale_ls_4_8","ie_tax_sale_goods_eu_0","",""
"","","","","","","","ie_tax_sale_goods_0","ie_tax_sale_goods_eu_0","",""
"","","","","","","","ie_tax_sale_services_0","ie_tax_sale_services_eu_0","",""
"","","","","","","","ie_tax_sale_goods_exempt","ie_tax_sale_goods_eu_0","",""
"","","","","","","","ie_tax_sale_services_exempt","ie_tax_sale_services_eu_0","",""
"","","","","","","","ie_tax_purchase_goods_23","ie_tax_purchase_goods_eu_23","",""
"","","","","","","","ie_tax_purchase_services_23","ie_tax_purchase_services_eu_23","",""
"","","","","","","","ie_tax_purchase_goods_13_5","ie_tax_purchase_goods_eu_13_5","",""
"","","","","","","","ie_tax_purchase_services_13_5","ie_tax_purchase_services_eu_13_5","",""
"","","","","","","","ie_tax_purchase_goods_9","ie_tax_purchase_goods_eu_9","",""
"","","","","","","","ie_tax_purchase_services_9","ie_tax_purchase_services_eu_9","",""
"","","","","","","","ie_tax_purchase_goods_0","ie_tax_purchase_eu_goods_0","",""
"","","","","","","","ie_tax_purchase_services_0","ie_tax_purchase_eu_services_0","",""
"","","","","","","","ie_tax_purchase_goods_exempt","ie_tax_purchase_eu_goods_0","",""
"","","","","","","","ie_tax_purchase_services_exempt","ie_tax_purchase_eu_services_0","",""
"ie_fp_ex","4","Import/Export (EX)","1","","","","ie_tax_sale_goods_23","ie_tax_sale_ex_goods_0","",""
"","","","","","","","ie_tax_sale_services_23","ie_tax_sale_ex_services_0","",""
"","","","","","","","ie_tax_sale_goods_13_5","ie_tax_sale_ex_goods_0","",""
"","","","","","","","ie_tax_sale_services_13_5","ie_tax_sale_ex_services_0","",""
"","","","","","","","ie_tax_sale_goods_9","ie_tax_sale_ex_goods_0","",""
"","","","","","","","ie_tax_sale_services_9","ie_tax_sale_ex_services_0","",""
"","","","","","","","ie_tax_sale_ls_4_8","ie_tax_sale_ex_goods_0","",""
"","","","","","","","ie_tax_sale_goods_0","ie_tax_sale_ex_goods_0","",""
"","","","","","","","ie_tax_sale_services_0","ie_tax_sale_ex_services_0","",""
"","","","","","","","ie_tax_sale_goods_exempt","ie_tax_sale_ex_goods_0","",""
"","","","","","","","ie_tax_sale_services_exempt","ie_tax_sale_ex_services_0","",""
"","","","","","","","ie_tax_purchase_goods_23","ie_tax_purchase_goods_ex_23","",""
"","","","","","","","ie_tax_purchase_services_23","ie_tax_purchase_services_ex_23","",""
"","","","","","","","ie_tax_purchase_goods_13_5","ie_tax_purchase_goods_ex_13_5","",""
"","","","","","","","ie_tax_purchase_services_13_5","ie_tax_purchase_services_ex_13_5","",""
"","","","","","","","ie_tax_purchase_goods_9","ie_tax_purchase_goods_ex_9","",""
"","","","","","","","ie_tax_purchase_services_9","ie_tax_purchase_services_ex_9","",""
"","","","","","","","ie_tax_purchase_goods_0","ie_tax_purchase_ex_goods_0","",""
"","","","","","","","ie_tax_purchase_services_0","ie_tax_purchase_ex_services_0","",""
"","","","","","","","ie_tax_purchase_goods_exempt","ie_tax_purchase_ex_goods_0","",""
"","","","","","","","ie_tax_purchase_services_exempt","ie_tax_purchase_ex_services_0","",""

```

## File: data\template\account.tax-ie.csv

```csv
"id","name","description","invoice_label","type_tax_use","amount_type","amount","tax_group_id","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/factor_percent","repartition_line_ids/account_id","repartition_line_ids/tag_ids"
"ie_tax_sale_goods_23","23% G","23% Standard rate - Goods","23%","sale","percent","23","ie_tax_group_vat_23","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","l10n_ie_account_3804","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","l10n_ie_account_3804","-T1"
"ie_tax_sale_services_23","23% S","23% Standard rate - Services","23%","sale","percent","23","ie_tax_group_vat_23","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_3804","+T1"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_3804","-T1"
"ie_tax_sale_goods_13_5","13.5% G","13.5% Reduced rate - Goods","13.5%","sale","percent","13.5","ie_tax_group_vat_13_5","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","l10n_ie_account_3804","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","l10n_ie_account_3804","-T1"
"ie_tax_sale_services_13_5","13.5% S","13.5% Reduced rate - Services","13.5%","sale","percent","13.5","ie_tax_group_vat_13_5","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_3804","+T1"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_3804","-T1"
"ie_tax_sale_goods_9","9% G","9% Second reduced rate - Goods","9%","sale","percent","9","ie_tax_group_vat_9","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","l10n_ie_account_3804","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","l10n_ie_account_3804","-T1"
"ie_tax_sale_services_9","9% S","9% Second reduced rate - Services","9%","sale","percent","9","ie_tax_group_vat_9","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_3804","+T1"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_3804","-T1"
"ie_tax_sale_ls_4_8","4.8% LS","4.8% Livestock rate","4.8%","sale","percent","4.8","ie_tax_group_vat_4_8","False","base","invoice","","",""
"","","","","","","","","","tax","invoice","","l10n_ie_account_3804","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","l10n_ie_account_3804","-T1"
"ie_tax_sale_goods_0","0% G","0% Goods","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_sale_services_0","0% S","0% Services","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_sale_goods_exempt","0% G EXEMPT","0% Goods Exempt","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_sale_services_exempt","0% S EXEMPT","0% Services Exempt","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_sale_goods_eu_0","0% EU G","0% EU Goods","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","","+E1"
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","","-E1"
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_sale_services_eu_0","0% EU S","0% EU Services","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","","+ES1"
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","","-ES1"
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_sale_ex_goods_0","0% EX G","0% Export Goods","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_sale_ex_services_0","0% EX S","0% Export Services","0%","sale","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
"","","","","","","","","","tax","invoice","","","+T1"
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","","","-T1"
"ie_tax_purchase_goods_23","23% G","23% Standard rate - Goods","23%","purchase","percent","23","ie_tax_group_vat_23","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_23","23% S","23% Standard rate - Services","23%","purchase","percent","23","ie_tax_group_vat_23","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_eu_23","23% EU G","23% Standard rate - EU - Goods","23%","purchase","percent","23","ie_tax_group_vat_23","True","base","invoice","","","+E2"
,"","","","","","","","","tax","invoice","-100","l10n_ie_account_3804","-T1"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","","-E2"
,"","","","","","","","","tax","refund","-100","l10n_ie_account_3804","+T1"
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_eu_23","23% EU S","23% Standard rate - EU - Services","23%","purchase","percent","23","ie_tax_group_vat_23","True","base","invoice","","","+ES2"
,"","","","","","","","","tax","invoice","-100","l10n_ie_account_3804","-T1"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","","-ES2"
,"","","","","","","","","tax","refund","-100","l10n_ie_account_3804","+T1"
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_ex_23","23% EX G","23% Standard rate - Import - Goods","23%","purchase","percent","23","ie_tax_group_vat_23","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_ex_23","23% EX S","23% Standard rate - Import - Services","23%","purchase","percent","23","ie_tax_group_vat_23","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_13_5","13.5% G","13.5% Reduced rate - Goods","13.5%","purchase","percent","13.5","ie_tax_group_vat_13_5","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_13_5","13.5% S","13.5% Reduced rate - Services","13.5%","purchase","percent","13.5","ie_tax_group_vat_13_5","True","base","invoice"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_eu_13_5","13.5% EU G","13.5% Reduced rate - EU - Goods","13.5%","purchase","percent","13.5","ie_tax_group_vat_13_5","True","base","invoice","","","+E2"
,"","","","","","","","","tax","invoice","-100","l10n_ie_account_3804","-T1"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","","-E2"
,"","","","","","","","","tax","refund","-100","l10n_ie_account_3804","+T1"
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_eu_13_5","13.5% EU S","13.5% Reduced rate - EU - Services","13.5%","purchase","percent","13.5","ie_tax_group_vat_13_5","True","base","invoice","","","+ES2"
,"","","","","","","","","tax","invoice","-100","l10n_ie_account_3804","-T1"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","","-ES2"
,"","","","","","","","","tax","refund","-100","l10n_ie_account_3804","+T1"
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_ex_13_5","13.5% EX G","13.5% Reduced rate - Import - Goods","13.5%","purchase","percent","13.5","ie_tax_group_vat_13_5","False","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_ex_13_5","13.5% EX S","13.5% Reduced rate - Import - Services","13.5%","purchase","percent","13.5","ie_tax_group_vat_13_5","False","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_9","9% G","9% Second reduced rate - Goods","9%","purchase","percent","9","ie_tax_group_vat_9","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_9","9% S","9% Second reduced rate - Services","9%","purchase","percent","9","ie_tax_group_vat_9","True","base","invoice"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_eu_9","9% EU G","9% Second reduced rate - EU - Goods","9%","purchase","percent","9","ie_tax_group_vat_9","True","base","invoice","","","+E2"
,"","","","","","","","","tax","invoice","-100","l10n_ie_account_3804","-T1"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","","-E2"
,"","","","","","","","","tax","refund","-100","l10n_ie_account_3804","+T1"
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_eu_9","9% EU S","9% Second reduced rate - EU - Services","9%","purchase","percent","9","ie_tax_group_vat_9","True","base","invoice","","","+E2"
,"","","","","","","","","tax","invoice","-100","l10n_ie_account_3804","-T1"
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","","-E2"
,"","","","","","","","","tax","refund","-100","l10n_ie_account_3804","+T1"
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_ex_9","9% EX G","9% Second reduced rate - Import - Goods","9%","purchase","percent","9","ie_tax_group_vat_9","False","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_services_ex_9","9% EX S","9% Second reduced rate - Import - Services","9%","purchase","percent","9","ie_tax_group_vat_9","False","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_ls_4_8","4.8% LS","4.8% Livestock rate","4.8%","purchase","percent","4.8","ie_tax_group_vat_4_8","False","base","invoice","","",""
,"","","","","","","","","tax","invoice","","l10n_ie_account_2131","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","l10n_ie_account_2131","-T2"
"ie_tax_purchase_goods_0","0% G","0% Goods","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_services_0","0% S","0% Services","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_goods_exempt","0% G EXEMPT","0% Goods Exempt","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_services_exempt","0% S EXEMPT","0% Services Exempt","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_eu_goods_0","0% EU G","0% EU Goods","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","","+E2"
,"","","","","","","","","tax","invoice","","","+T1"
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","","-E2"
,"","","","","","","","","tax","refund","","","-T1"
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_eu_services_0","0% EU S","0% EU Services","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","","+ES2"
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","","-ES2"
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_ex_goods_0","0% EX G","0% Import Goods","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_ex_services_0","0% EX S","0% Import Services","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","",""
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","",""
,"","","","","","","","","tax","refund","","","-T2"
"ie_tax_purchase_ex_goods_pa_0","0% EX G PA","0% Import Goods - Postponed Accounting","0%","purchase","percent","0","ie_tax_group_vat_0","True","base","invoice","","","+PA1"
,"","","","","","","","","tax","invoice","-100","","-T1"
,"","","","","","","","","tax","invoice","","","+T2"
,"","","","","","","","","base","refund","","","-PA1"
,"","","","","","","","","tax","refund","-100","","+T1"
,"","","","","","","","","tax","refund","","","-T2"

```

## File: data\template\account.tax.group-ie.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id"
"ie_tax_group_vat_23","VAT 23%","base.ie","l10n_ie_account_3804","l10n_ie_account_2132"
"ie_tax_group_vat_13_5","VAT 13.5%","base.ie","l10n_ie_account_3804","l10n_ie_account_2132"
"ie_tax_group_vat_9","VAT 9%","base.ie","l10n_ie_account_3804","l10n_ie_account_2132"
"ie_tax_group_vat_4_8","VAT 4.8%","base.ie","l10n_ie_account_3804","l10n_ie_account_2132"
"ie_tax_group_vat_0","VAT 0%","base.ie","l10n_ie_account_3804","l10n_ie_account_2132"

```

## File: models\template_ie.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ie')
    def _get_ie_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_ie_account_2100',
            'property_account_payable_id': 'l10n_ie_account_34',
            'property_account_expense_categ_id': 'l10n_ie_account_60',
            'property_account_income_categ_id': 'l10n_ie_account_70',
            'property_stock_valuation_account_id': 'l10n_ie_account_630',
            'property_advance_tax_payment_account_id': 'l10n_ie_account_2132',
            'code_digits': '6',
        }

    @template('ie', 'res.company')
    def _get_ie_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ie',
                'bank_account_code_prefix': '230',
                'cash_account_code_prefix': '231',
                'transfer_account_code_prefix': '232',
                'account_default_pos_receivable_account_id': 'l10n_ie_account_2101',
                'income_currency_exchange_account_id': 'l10n_ie_account_761',
                'expense_currency_exchange_account_id': 'l10n_ie_account_661',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_ie_account_640',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_ie_account_730',
                'default_cash_difference_expense_account_id': 'l10n_ie_account_641',
                'default_cash_difference_income_account_id': 'l10n_ie_account_731',
                'account_sale_tax_id': 'ie_tax_sale_goods_23',
                'account_purchase_tax_id': 'ie_tax_purchase_goods_23',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ie

```

