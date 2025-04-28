# Odoo Module: l10n_jp

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) Quartile Limited

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) Quartile Limited

{
    'name': 'Japan - Accounting',
    'version': '2.2',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """

Overview:
---------

* Chart of Accounts and Taxes template for companies in Japan.
* This probably does not cover all the necessary accounts for a company. \
You are expected to add/delete/modify accounts based on this template.

Note:
-----

* Fiscal positions '内税' and '外税' have been added to handle special \
requirements which might arise from POS implementation. [1]  Under normal \
circumstances, you might not need to use those at all.

[1] See https://github.com/odoo/odoo/pull/6470 for detail.

    """,
    'author': 'Quartile Limited',
    'website': 'https://www.quartile.co/',
    'depends': ['account'],
    'data': [
        'data/l10n_jp_chart_data.xml',
        'data/account.account.template.csv',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_data.xml',
        'data/account.fiscal.position.template.csv',
        'data/account.fiscal.position.tax.template.csv',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,user_type_id/id,tax_ids/id,reconcile,currency_id/id,chart_template_id/id
A11101,A11101,普通預金,account.data_account_type_liquidity,,FALSE,,l10n_jp1
A11103,A11103,定期預金,account.data_account_type_liquidity,,FALSE,,l10n_jp1
A11211,A11211,売掛金,account.data_account_type_receivable,,TRUE,,l10n_jp1
A11212,A11212,売掛金(USD),account.data_account_type_receivable,,TRUE,base.USD,l10n_jp1
A11213,A11213,売掛金(PoS),account.data_account_type_receivable,,TRUE,,l10n_jp1
A11215,A11215,受取手形,account.data_account_type_receivable,,TRUE,,l10n_jp1
A11216,A11216,割引手形,account.data_account_type_receivable,,TRUE,,l10n_jp1
A11219,A11219,出庫請求仮,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11301,A11301,未収収益,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11401,A11401,有価証券,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11501,A11501,商品,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11502,A11502,製品,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11504,A11504,仕掛品,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11505,A11505,原材料,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11506,A11506,資材,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11503,A11503,積送品,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11601,A11601,未収金,account.data_account_type_receivable,,TRUE,,l10n_jp1
A11802,A11802,前渡金,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11804,A11804,前払費用,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11805,A11805,立替金,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11806,A11806,仮払金,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11807,A11807,敷金・保証金,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11809,A11809,仮払消費税,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A11901,A11901,貸倒引当金,account.data_account_type_current_assets,,FALSE,,l10n_jp1
A12111,A12111,建物及び構築物,account.data_account_type_fixed_assets,,FALSE,,l10n_jp1
A12112,A12112,建物及び構築物減償累計額,account.data_account_type_fixed_assets,,FALSE,,l10n_jp1
A12121,A12121,機械及び装置,account.data_account_type_fixed_assets,,FALSE,,l10n_jp1
A12122,A12122,機械及び装置物減償累計額,account.data_account_type_fixed_assets,,FALSE,,l10n_jp1
A12131,A12131,工具、機器及び備品,account.data_account_type_fixed_assets,,FALSE,,l10n_jp1
A12132,A12132,工具、機器及び備品減償累計額,account.data_account_type_fixed_assets,,FALSE,,l10n_jp1
A12141,A12141,土地,account.data_account_type_fixed_assets,,FALSE,,l10n_jp1
A1221,A1221,ソフトウェア,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A1222,A1222,ソフトウェア仮勘定,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A1223,A1223,その他無形固定資産,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A12301,A12301,投資有価証券,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A12302,A12302,関係会社株式,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A12303,A12303,長期貸付金,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A12304,A12304,長期前払費用,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A13001,A13001,創立費,account.data_account_type_non_current_assets,,FALSE,,l10n_jp1
A21211,A21211,買掛金,account.data_account_type_payable,,TRUE,,l10n_jp1
A21212,A21212,買掛金(USD),account.data_account_type_payable,,TRUE,base.USD,l10n_jp1
A21215,A21215,支払手形,account.data_account_type_payable,,TRUE,,l10n_jp1
A21219,A21219,入庫請求仮,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A213,A213,短期借入金等,account.data_account_type_payable,,TRUE,,l10n_jp1
A21301,A21301,短期借入金,account.data_account_type_payable,,TRUE,,l10n_jp1
A21302,A21302,一年以内返済長期借入金,account.data_account_type_payable,,TRUE,,l10n_jp1
A21401,A21401,未払金,account.data_account_type_payable,,TRUE,,l10n_jp1
A21404,A21404,未払金(クレジット),account.data_account_type_payable,,TRUE,,l10n_jp1
A21407,A21407,未払配当金,account.data_account_type_payable,,TRUE,,l10n_jp1
A21501,A21501,未払費用,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A21601,A21601,未払法人税等,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A21802,A21802,前受金,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A21803,A21803,預り金,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A21804,A21804,仮受金,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A21806,A21806,賞与引当金,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A21809,A21809,仮受消費税,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A22001,A22001,社債,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A22002,A22002,長期借入金,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A22005,A22005,退職給付引当金,account.data_account_type_current_liabilities,,FALSE,,l10n_jp1
A31001,A31001,資本金,account.data_account_type_equity,,FALSE,,l10n_jp1
A32101,A32101,資本準備金,account.data_account_type_equity,,FALSE,,l10n_jp1
A32109,A32109,その他資本剰余金,account.data_account_type_equity,,FALSE,,l10n_jp1
A33101,A33101,利益準備金,account.data_account_type_equity,,FALSE,,l10n_jp1
A33202,A33202,特別償却準備金,account.data_account_type_equity,,FALSE,,l10n_jp1
A33204,A33204,任意積立金,account.data_account_type_equity,,FALSE,,l10n_jp1
A33209,A33209,繰越利益剰余金,account.data_account_type_equity,,FALSE,,l10n_jp1
B41001,B41001,売上高,account.data_account_type_revenue,,FALSE,,l10n_jp1
B42001,B42001,売上原価,account.data_account_type_other_income,,FALSE,,l10n_jp1
B42091,B42091,購買価格差異,account.data_account_type_other_income,,FALSE,,l10n_jp1
B42092,B42092,棚卸調整,account.data_account_type_other_income,,FALSE,,l10n_jp1
B50011,B50011,貸倒引当金繰入額,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50041,B50041,役員報酬,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50071,B50071,給料及び手当,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50081,B50081,賞与,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50091,B50091,退職金,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50111,B50111,賞与引当金繰入,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50131,B50131,退職給付引当金繰入,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50151,B50151,法定福利費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50161,B50161,福利厚生費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50171,B50171,保険料,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50191,B50191,外注費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50211,B50211,荷造運賃,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50221,B50221,棚卸減耗費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50231,B50231,商品評価損,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50241,B50241,地代家賃,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50251,B50251,リース料,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50271,B50271,広告宣伝費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50281,B50281,通信費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50311,B50311,消耗品費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50341,B50341,旅費交通費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50371,B50371,交際費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50401,B50401,支払手数料,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50431,B50431,諸会費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50461,B50461,新聞図書費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50491,B50491,租税公課,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50511,B50511,水道光熱費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50521,B50521,会議費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B50551,B50551,雑費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B61101,B61101,受取利息,account.data_account_type_other_income,,FALSE,,l10n_jp1
B61201,B61201,受取配当金,account.data_account_type_other_income,,FALSE,,l10n_jp1
B61401,B61401,未実現為替差益,account.data_account_type_other_income,,FALSE,,l10n_jp1
B61501,B61501,実現為替差益,account.data_account_type_other_income,,FALSE,,l10n_jp1
B61601,B61601,投資有価証券売却益(外益),account.data_account_type_other_income,,FALSE,,l10n_jp1
B61602,B61602,投資有価証券評価益(外益),account.data_account_type_other_income,,FALSE,,l10n_jp1
B61801,B61801,雑収入,account.data_account_type_other_income,,FALSE,,l10n_jp1
B62101,B62101,支払利息,account.data_account_type_expenses,,FALSE,,l10n_jp1
B62201,B62201,創立費償却,account.data_account_type_expenses,,FALSE,,l10n_jp1
B62202,B62202,減価償却費,account.data_account_type_expenses,,FALSE,,l10n_jp1
B62401,B62401,未実現為替差損,account.data_account_type_expenses,,FALSE,,l10n_jp1
B62501,B62501,実現為替差損,account.data_account_type_expenses,,FALSE,,l10n_jp1
B62601,B62601,投資有価証券売却損(外損),account.data_account_type_expenses,,FALSE,,l10n_jp1
B62602,B62602,投資有価証券評価損(外損),account.data_account_type_expenses,,FALSE,,l10n_jp1
B62801,B62801,雑費用,account.data_account_type_expenses,,FALSE,,l10n_jp1
B71101,B71101,固定資産売却益,account.data_account_type_other_income,,FALSE,,l10n_jp1
B71102,B71102,固定資産評価益,account.data_account_type_other_income,,FALSE,,l10n_jp1
B71201,B71201,投資有価証券売却益(特益),account.data_account_type_other_income,,FALSE,,l10n_jp1
B71202,B71202,投資有価証券評価益(特益),account.data_account_type_other_income,,FALSE,,l10n_jp1
B72101,B72101,固定資産売却損,account.data_account_type_expenses,,FALSE,,l10n_jp1
B72102,B72102,固定資産評価損,account.data_account_type_expenses,,FALSE,,l10n_jp1
B72201,B72201,投資有価証券売却損(特損),account.data_account_type_expenses,,FALSE,,l10n_jp1
B72202,B72202,投資有価証券評価損(特損),account.data_account_type_expenses,,FALSE,,l10n_jp1
B80101,B80101,法人税、住民税及び事業税,account.data_account_type_expenses,,FALSE,,l10n_jp1
B90001,B90001,期間利益,account.data_account_type_expenses,,FALSE,,l10n_jp1

```

## File: data\account.fiscal.position.tax.template.csv

```csv
id,position_id/id,tax_src_id/id,tax_dest_id/id
fp_inclusive_tax_template_in_10,fiscal_position_tax_inclusive_template,tax_in_e_10,tax_in_i_10
fp_inclusive_tax_template_out_10,fiscal_position_tax_inclusive_template,tax_out_e_10,tax_out_i_10
fp_inclusive_tax_template_in,fiscal_position_tax_inclusive_template,tax_in_e,tax_in_i
fp_inclusive_tax_template_out,fiscal_position_tax_inclusive_template,tax_out_e,tax_out_i
fp_exclusive_tax_template_in_10,fiscal_position_tax_exclusive_template,tax_in_i_10,tax_in_e_10
fp_exclusive_tax_template_out_10,fiscal_position_tax_exclusive_template,tax_out_i_10,tax_out_e_10
fp_exclusive_tax_template_in,fiscal_position_tax_exclusive_template,tax_in_i,tax_in_e
fp_exclusive_tax_template_out,fiscal_position_tax_exclusive_template,tax_out_i,tax_out_e
fp_exepmt_tax_template_in_10,fiscal_position_tax_exempt_template,tax_in_e_10,tax_in_x
fp_exepmt_tax_template_in_i_10,fiscal_position_tax_exempt_template,tax_in_i_10,tax_in_x
fp_exepmt_tax_template_out_10,fiscal_position_tax_exempt_template,tax_out_e_10,tax_out_im
fp_exepmt_tax_template_out_i_10,fiscal_position_tax_exempt_template,tax_out_i_10,tax_out_im
fp_exepmt_tax_template_in,fiscal_position_tax_exempt_template,tax_in_e,tax_in_x
fp_exepmt_tax_template_in_i,fiscal_position_tax_exempt_template,tax_in_i,tax_in_x
fp_exepmt_tax_template_out,fiscal_position_tax_exempt_template,tax_out_e,tax_out_im
fp_exepmt_tax_template_out_i,fiscal_position_tax_exempt_template,tax_out_i,tax_out_im
fp_reduction_tax_template_in_e,fiscal_position_tax_reduction_template,tax_in_e_10,tax_in_e
fp_reduction_tax_template_in_i,fiscal_position_tax_reduction_template,tax_in_i_10,tax_in_i
fp_reduction_tax_template_out_e,fiscal_position_tax_reduction_template,tax_out_e_10,tax_out_e
fp_reduction_tax_template_out_i,fiscal_position_tax_reduction_template,tax_out_i_10,tax_out_i

```

## File: data\account.fiscal.position.template.csv

```csv
id,chart_template_id/id,name
fiscal_position_tax_inclusive_template,l10n_jp1,内税
fiscal_position_tax_exclusive_template,l10n_jp1,外税
fiscal_position_tax_exempt_template,l10n_jp1,海外取引先
fiscal_position_tax_reduction_template,l10n_jp1,軽減税率

```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_0,税対象外/免除
tax_group_8,消費税 8%
tax_group_10,消費税 10%

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_jp.l10n_jp1')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart Template -->
    <record id="l10n_jp1" model="account.chart.template">
        <field name="property_account_receivable_id" ref="A11211"/>
        <field name="property_account_payable_id" ref="A21211"/>
        <field name="property_account_expense_id" ref="A21219"/>
        <field name="property_account_income_id" ref="B41001"/>
        <field name="property_account_expense_categ_id" ref="A21219"/>
        <field name="property_account_income_categ_id" ref="B41001"/>
        <field name="income_currency_exchange_account_id" ref="B61501"/>
        <field name="expense_currency_exchange_account_id" ref="B62501"/>
        <field name="default_pos_receivable_account_id" ref="A11213" />
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.jp"/>
    </record>

    <record id="tax_report_to_pay" model="account.tax.report.line">
        <field name="name">支払対象税額</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_to_pay_temp_tx" model="account.tax.report.line">
        <field name="name">仮受税額</field>
        <field name="parent_id" ref="tax_report_to_pay"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_to_pay_temp_tx_output_8" model="account.tax.report.line">
        <field name="name">仮受消費税(8%)</field>
        <field name="tag_name">仮受消費税(8%)</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_tx"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_to_pay_temp_tx_output_10" model="account.tax.report.line">
        <field name="name">仮受消費税(10%)</field>
        <field name="tag_name">仮受消費税(10%)</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_tx"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_to_pay_temp_tx_duty_free" model="account.tax.report.line">
        <field name="name">免税</field>
        <field name="tag_name">免税</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_tx"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_to_pay_temp_tx_tax_free" model="account.tax.report.line">
        <field name="name">不課税</field>
        <field name="tag_name">不課税 (仮受税額)</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_tx"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_to_pay_temp_pmt" model="account.tax.report.line">
        <field name="name">仮払税額</field>
        <field name="parent_id" ref="tax_report_to_pay"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_to_pay_temp_pmt_susp_cons_8" model="account.tax.report.line">
        <field name="name">仮払消費税(8%)</field>
        <field name="tag_name">仮払消費税(8%)</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_pmt"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_to_pay_temp_pmt_susp_cons_10" model="account.tax.report.line">
        <field name="name">仮払消費税(10%)</field>
        <field name="tag_name">仮払消費税(10%)</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_pmt"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_to_pay_temp_pmt_import_8" model="account.tax.report.line">
        <field name="name">輸入</field>
        <field name="tag_name">輸入</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_pmt"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_to_pay_temp_pmt_tax_free" model="account.tax.report.line">
        <field name="name">不課税</field>
        <field name="tag_name">不課税 (仮払税額)</field>
        <field name="parent_id" ref="tax_report_to_pay_temp_pmt"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_comp_basis" model="account.tax.report.line">
        <field name="name">税計算基準額</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_comp_basis_sales" model="account.tax.report.line">
        <field name="name">販売基準額</field>
        <field name="parent_id" ref="tax_report_comp_basis"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_comp_basis_sales_taxable_8" model="account.tax.report.line">
        <field name="name">課税対象売上(8%)</field>
        <field name="tag_name">課税対象売上(8%)</field>
        <field name="parent_id" ref="tax_report_comp_basis_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_comp_basis_sales_taxable_10" model="account.tax.report.line">
        <field name="name">課税対象売上(10%)</field>
        <field name="tag_name">課税対象売上(10%)</field>
        <field name="parent_id" ref="tax_report_comp_basis_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_comp_basis_sales_duty_free" model="account.tax.report.line">
        <field name="name">免税売上</field>
        <field name="tag_name">免税売上</field>
        <field name="parent_id" ref="tax_report_comp_basis_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_comp_basis_sales_tax_free" model="account.tax.report.line">
        <field name="name">不課税売上</field>
        <field name="tag_name">不課税売上</field>
        <field name="parent_id" ref="tax_report_comp_basis_sales"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_comp_basis_purchases" model="account.tax.report.line">
        <field name="name">購買基準額</field>
        <field name="parent_id" ref="tax_report_comp_basis"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_comp_basis_purchases_taxable_8" model="account.tax.report.line">
        <field name="name">課税対象仕入(8%)</field>
        <field name="tag_name">課税対象仕入(8%)</field>
        <field name="parent_id" ref="tax_report_comp_basis_purchases"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_comp_basis_purchases_taxable_10" model="account.tax.report.line">
        <field name="name">課税対象仕入(10%)</field>
        <field name="tag_name">課税対象仕入(10%)</field>
        <field name="parent_id" ref="tax_report_comp_basis_purchases"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_comp_basis_purchases_import" model="account.tax.report.line">
        <field name="name">輸入仕入</field>
        <field name="tag_name">輸入仕入</field>
        <field name="parent_id" ref="tax_report_comp_basis_purchases"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_comp_basis_purchases_tax_free" model="account.tax.report.line">
        <field name="name">不課税仕入</field>
        <field name="tag_name">不課税仕入</field>
        <field name="parent_id" ref="tax_report_comp_basis_purchases"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

</odoo>
```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_in_e" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮受消費税(外) 8%</field>
        <field name="description">仮受消費税(外) 8%</field>
        <field name="amount_type">percent</field>
        <field name="amount">8</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_8')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_8')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_8"/>
    </record>
    <record id="tax_in_e_10" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮受消費税(外) 10%</field>
        <field name="description">仮受消費税(外) 10%</field>
        <field name="amount_type">percent</field>
        <field name="amount">10</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_10')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_10')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_10"/>
    </record>
    <record id="tax_in_i" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮受消費税(内) 8%</field>
        <field name="description">仮受消費税(内) 8%</field>
        <field name="amount_type">percent</field>
        <field name="amount">8</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">TRUE</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_8')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_8')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_8"/>
    </record>
    <record id="tax_in_i_10" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮受消費税(内) 10%</field>
        <field name="description">仮受消費税(内) 10%</field>
        <field name="amount_type">percent</field>
        <field name="amount">10</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">TRUE</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_10')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A21809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_tx_output_10')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_10"/>
    </record>
    <record id="tax_in_x" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">輸出免税</field>
        <field name="description">輸出免税</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_duty_free')],
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
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_duty_free')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_0"/>
    </record>
    <record id="tax_in_o" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">非課税販売</field>
        <field name="description">非課税販売</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_tax_free')],
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
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_sales_tax_free')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_0"/>
    </record>
    <record id="tax_out_e" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮払消費税(外) 8%</field>
        <field name="description">仮払消費税(外) 8%</field>
        <field name="amount_type">percent</field>
        <field name="amount">8</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_8')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_8')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_8"/>
    </record>
    <record id="tax_out_e_10" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮払消費税(外) 10%</field>
        <field name="description">仮払消費税(外) 10%</field>
        <field name="amount_type">percent</field>
        <field name="amount">10</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_10')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_10')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_10"/>
    </record>
    <record id="tax_out_i" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮払消費税(内) 8%</field>
        <field name="description">仮払消費税(内) 8%</field>
        <field name="amount_type">percent</field>
        <field name="amount">8</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">True</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_8')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_8')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_8"/>
    </record>
    <record id="tax_out_i_10" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">仮払消費税(内) 10%</field>
        <field name="description">仮払消費税(内) 10%</field>
        <field name="amount_type">percent</field>
        <field name="amount">10</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">True</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'plus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_10')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_taxable_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('A11809'),
                'minus_report_line_ids': [ref('l10n_jp.tax_report_to_pay_temp_pmt_susp_cons_10')],
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_10"/>
    </record>
    <record id="tax_out_im" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">海外仕入</field>
        <field name="description">海外仕入</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_import')],
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
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_import')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_0"/>
    </record>
    <record id="tax_out_o" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="chart_template_id" ref="l10n_jp1"/>
        <field name="name">非課税購買</field>
        <field name="description">非課税購買</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="active">True</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_tax_free')],
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
                'minus_report_line_ids': [ref('l10n_jp.tax_report_comp_basis_purchases_tax_free')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_0"/>
    </record>

</odoo>
```

## File: data\l10n_jp_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart Template -->
    <record id="l10n_jp1" model="account.chart.template">
        <field name="name">日本勘定設定テンプレート</field>
        <field name="code_digits">7</field>
        <field name="bank_account_code_prefix">A11102</field>
        <field name="cash_account_code_prefix">A11105</field>
        <field name="transfer_account_code_prefix">A11109</field>
        <field name="currency_id" ref="base.JPY"/>
    </record>
</odoo>

```

