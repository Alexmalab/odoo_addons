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
id,name,country_id/id
tax_group_0,税対象外/免除,base.jp
tax_group_8,8% 対象,base.jp
tax_group_10,10% 対象,base.jp

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
        <field name="country_id" ref="base.jp"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="4.8" width="50.4" height="36.4" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.68" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
    </mask>
    <symbol id="c" data-name="account icon" viewBox="0 0 106 106">
      <g style="mask: url(#a)">
        <g>
          <path d="M0,0H106V106H0Z" style="fill: #5a5a64;fill-rule: evenodd"/>
          <path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill: #fff;fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <g>
            <path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill: #393939;fill-rule: evenodd;opacity: 0.324000000953674;isolation: isolate"/>
            <g style="opacity: 0.30000000000000004">
              <g>
                <path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/>
                <path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/>
                <path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/>
                <path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/>
                <path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/>
                <path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/>
                <path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/>
              </g>
              <path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/>
            </g>
            <path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill: #a8a9ab"/>
            <g>
              <path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill: #a8a9ab"/>
              <path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill: #a8a9ab"/>
              <path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill: #a8a9ab"/>
              <path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill: #a8a9ab"/>
              <path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill: #a8a9ab"/>
              <path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill: #a8a9ab"/>
              <path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill: #a8a9ab"/>
            </g>
          </g>
        </g>
      </g>
    </symbol>
  </defs>
  <g>
    <use width="106" height="106" transform="translate(-0.07 0)" xlink:href="#c"/>
    <rect x="6.2" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="1200" height="800" transform="translate(4.8 4.8) scale(0.04 0.05)" xlink:href="data:image/png;base64,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"/>
    </g>
  </g>
</svg>

```

